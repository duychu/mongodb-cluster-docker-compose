- **Step 1: Start all of the containers**

```bash
docker-compose up -d
```

- **Step 2: Initialize the replica sets (config servers and shards) and routers**

```bash
docker-compose exec configsvr01 sh -c "mongo < /scripts/init-configserver.js"
docker-compose exec shard01-a sh -c "mongo < /scripts/init-shard01.js"
docker-compose exec shard02-a sh -c "mongo < /scripts/init-shard02.js"
docker-compose exec shard03-a sh -c "mongo < /scripts/init-shard03.js"
```

- **Step 3: Initializing the router**

```bash
docker-compose exec router01 sh -c "mongo < /scripts/init-router.js"
```

- **Step 4: Enable sharding and setup sharding-key**

```bash
docker-compose exec router01 mongo --port 27017

// Enable sharding for database `genieacs`
sh.enableSharding("genieacs")
// Setup shardingKey for collection `devices`**
db.adminCommand( { shardCollection: "genieacs.devices", key: { _id: "hashed" } } )
```

- **Verify the status of the sharded cluster**

```bash
docker-compose exec router01 mongo --port 27017

sh.status()
```

- **Verify status of replica set for each shard**

```bash
docker exec -it rydell-shard-01-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
docker exec -it rydell-shard-02-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
docker exec -it rydell-shard-03-node-a bash -c "echo 'rs.status()' | mongo --port 27017" 
```

- **Check the distribution of collection**

```bash
docker-compose exec router01 mongo --port 27017

db.adminCommand( { flushRouterConfig: "genieacs.devices" } );
db.getSiblingDB("genieacs").devices.getShardDistribution();

```