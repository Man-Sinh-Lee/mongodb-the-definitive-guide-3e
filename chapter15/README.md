### Chapter 15: Configuring Sharding

#### When to Shard
Sharding is a strategy used when a single server can no longer handle the size or throughput requirements of an application. Common reasons to shard include:
- **Increasing RAM or disk space**: A sharded cluster distributes data across multiple machines, making more RAM and disk available collectively.
- **Reducing load on a server**: Distributing data helps reduce the load on individual servers.
- **Handling higher throughput**: Sharding improves read and write capacity by parallelizing operations across multiple servers .

It's crucial to carefully monitor the system's performance metrics to decide when sharding is needed. Sharding prematurely adds complexity, while sharding too late may result in downtime for a heavily loaded system .

#### Starting the Servers
1. **Config Servers**:
   Config servers are critical because they store metadata about the cluster, such as which shards hold what data. A config server replica set typically consists of three members. These must be set up before the `mongos` processes. In production, it's important to use nonephemeral drives for config servers, and they should run on separate physical machines to ensure reliability.

2. **The mongos Processes**:
   The `mongos` process acts as a router, forwarding client requests to the appropriate shards. It holds no data itself, instead pulling cluster configuration information from the config servers. A minimal setup requires at least two `mongos` processes to ensure high availability. These should be located close to the shards to minimize latency  .

3. **Adding a Shard from a Replica Set**:
   When adding a replica set as a shard, the `mongos` must be instructed on how to locate the replica set. For example, using `mongo shell`, one would connect to the primary member of the replica set and use the `rs.status()` command to check the configuration before adding it to the cluster .

4. **Adding Capacity**:
   To scale a sharded cluster, new shards are added as needed. MongoDB then redistributes chunks of data from overloaded shards to the new shards through a background process .

5. **Sharding Data**:
   Once the setup is complete, you explicitly tell MongoDB which collections to shard. For example:
   ```js
   db.enableSharding("music");
   sh.shardCollection("music.artists", {"name": 1});
   ```
   Data will be distributed across shards based on the shard key .

#### How MongoDB Tracks Cluster Data
MongoDB uses chunks to group documents based on the shard key. Each chunk is assigned to a shard, and the `mongos` router uses this information to forward queries to the appropriate shard. Chunk sizes and their distribution can be modified to optimize performance .

- **Chunk Ranges**: Chunks are created based on ranges of shard key values, and each chunk is associated with a specific shard.
- **Splitting Chunks**: As chunks grow in size, they are split into smaller chunks to ensure even distribution across the cluster. The splitting process is managed automatically by MongoDB, but can also be influenced manually  .

#### The Balancer
The balancer is responsible for redistributing chunks among shards to maintain balance in the cluster. It works in the background, triggered when one shard holds significantly more chunks than others. MongoDB supports multiple concurrent migrations to optimize balancing across a large number of shards  .

#### Collations
MongoDB allows the use of collations for string comparison rules, such as case sensitivity or accent handling, in sharded clusters. Collation is specified at the collection level, and there are specific rules for sharding collections with collations .

#### Change Streams
Change streams allow applications to listen to real-time changes in the data within a sharded cluster. Change streams can be opened against a `mongos` process, and MongoDB guarantees that notifications will be sent in the correct order through a global logical clock. However, certain situations, such as removing a shard, may cause change stream cursors to close, and they may not be resumable .