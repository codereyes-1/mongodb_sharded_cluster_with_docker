## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://localhost:40001
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "localhost:40001" },
      { _id : 1, host : "localhost:40002" },
      { _id : 2, host : "localhost:40003" }
    ]
  }
)

rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://localhost:50001
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "localhost:50001" },
      { _id : 1, host : "localhost:50002" },
      { _id : 2, host : "localhost:50003" }
    ]
  }
)

rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://localhost:60000
```
Add shard
```
mongos> sh.addShard("shard1rs/localhost:50001,localhost:50002,localhost:50003")
mongos> sh.status()
```
## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://localhost:50004
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "localhost:50004" },
      { _id : 1, host : "localhost:50005" },
      { _id : 2, host : "localhost:50006" }
    ]
  }
)

rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://localhost:60000

```
Add shard
Verify Shard status
```
mongos> sh.addShard("shard2rs/localhost:50004,localhost:50005,localhost:50006")
mongos> sh.status()
```
