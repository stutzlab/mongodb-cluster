# mongodb-cluster

This is a set of Mongo containers for creating clusters using Docker

Check specific images used in this example at

* http://github.com/stutzlab/mongo-cluster-router
* http://github.com/stutzlab/mongo-cluster-configsrv
* http://github.com/stutzlab/mongo-cluster-shard

## Usage

* In this example we will create a cluster with:
  * 2 routers
  * 3 config servers
  * 2 shards, each with 2 replicas

* We had some issues running the shards in Docker for Mac. Some shards would be freezed and Docker had to be restarted. Use a VirtualBox VM if needed.

* Create docker-compose.yml

```yml
version: '3.5'

services:

  mongo-express:
    image: mongo-express:0.54.0
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_SERVER=router1
    #   - ME_CONFIG_BASICAUTH_USERNAME=
    #   - ME_CONFIG_BASICAUTH_PASSWORD=
    restart: always

  router1:
    image: stutzlab/mongo-cluster-router
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1,configsrv2,configsrv3
      - SHARD_NAME_PREFIX=shard
      - SHARD_1_NODES=shard1-a,shard1-b
      - SHARD_2_NODES=shard2-a,shard2-b
    ports:
      - 27111:27017

  router2:
    image: stutzlab/mongo-cluster-router
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1,configsrv2,configsrv3
      - SHARD_NAME_PREFIX=shard
      - SHARD_1_NODES=shard1-a,shard1-b
      - SHARD_2_NODES=shard2-a,shard2-b
    ports:
      - 27112:27017

  configsrv1:
    image: stutzlab/mongo-cluster-configsrv
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1,configsrv2,configsrv3
    ports:
      - 27311:27017
    volumes:
      - configsrv1-data:/data

  configsrv2:
    image: stutzlab/mongo-cluster-configsrv
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1,configsrv2,configsrv3
    ports:
      - 27312:27017
    volumes:
      - configsrv2-data:/data

  configsrv3:
    image: stutzlab/mongo-cluster-configsrv
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1,configsrv2,configsrv3
    ports:
      - 27413:27017
    volumes:
      - configsrv3-data:/data

  shard1-a:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard1
      - SHARD_NODES=shard1-a,shard1-b
    ports:
      - 27511:27017
    volumes:
      - shard1-a-data:/data

  shard1-b:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard1
      - SHARD_NODES=shard1-a,shard1-b
    ports:
      - 27512:27017
    volumes:
      - shard1-b-data:/data

  shard2-a:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard2
      - SHARD_NODES=shard2-a,shard2-b
    ports:
      - 27611:27017
    volumes:
      - shard2-a-data:/data

  shard2-b:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard2
      - SHARD_NODES=shard2-a,shard2-b
    ports:
      - 27612:27017
    volumes:
      - shard2-b-data:/data

volumes:
  configsrv1-data:
  configsrv2-data:
  configsrv3-data:
  shard1-a-data:
  shard1-b-data:
  shard2-a-data:
  shard2-b-data:
```

* Run in this order to see things happening

```sh
#if all services are run concurrently on the first creation, race conditions may occur and two replicasets are created at the same time. run one lime at each time

#create shard nodes with replication sets
docker-compose up shard1-a shard2-a
docker-compose up shard1-b shard2-b

#create config nodes with replication sets
docker-compose up configsrv1
docker-compose up configsrv2
docker-compose up configsrv3

#create routers according to config and shard nodes
docker-compose up router1 router2

docker-compose up mongo-express
```

* This may take a while. Check when logs stop going crazy!

* Connect to mongo-express and see some internal collections
  * open browser at http://localhost:8081

* Show cluster status

```sh
docker-compose exec router1 mongo --port 27017
sh.status()
```

* Enable sharding of a collection in a database

```sh
docker-compose exec router1 mongo --port 27017
>
```

```js
//create database 'sampledb'
use sampledb

//enable sharding for database
sh.enableSharding("sampledb")

//enable sharding for collection 'sample-collection'
db.adminCommand( { shardCollection: "sampledb.collection1", key: { mykey: "hashed" } } )
db.adminCommand( { shardCollection: "sampledb.collection2", key: { _id: "hashed" } } )

//inspect cluster status
sh.status()

//add some data
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})
db.collection2.insert({"name": _rand()})

//show details about qtty of records per shard
db.collection2.find().explain(true)
```

## More resources

* https://github.com/minhhungit/mongodb-cluster-docker-compose

* https://medium.com/@gustavo.leitao/criando-um-cluster-mongodb-com-replicaset-e-sharding-com-docker-9cb19d456b56

