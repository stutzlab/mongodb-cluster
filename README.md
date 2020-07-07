# mongodb-cluster

This is a set of Mongo containers for creating clusters using Docker

## Usage

* Create docker-compose.yml

```yml
version: '3.5'
services:
  router1:
    image: stutzlab/mongo-cluster-router
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1
      - SHARD_NAME_PREFIX=shard
      - SHARD_1_NODES=shard1-a
      - SHARD_2_NODES=shard2-a
    ports:
      - 27111:27017

  configsrv1:
    image: stutzlab/mongo-cluster-configsrv
    environment:
      - CONFIG_SERVER_NAME=config-server
      - CONFIG_SERVER_NODES=configsrv1
    ports:
      - 27211:27017

  shard1-a:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard-1
      - SHARD_NODES=shard1-a
    ports:
      - 27311:27017

  shard2-a:
    image: stutzlab/mongo-cluster-shard
    environment:
      - SHARD_NAME=shard-2
      - SHARD_NODES=shard2-a
    ports:
      - 27312:27017
```

* Execute ```docker-compose up -d```

* Show cluster status

```sh
docker-compose exec router1 mongo --port 27017
sh.status()
```
