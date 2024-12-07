### Chapter 17: Sharding Administration

#### Seeing the Current State
- **Getting a Summary with `sh.status()`**:
  The `sh.status()` command provides an overview of the cluster, showing shard details, databases, and the distribution of chunks across shards. For small clusters, it shows a breakdown of the chunks on each shard. For larger clusters, it summarizes the chunk counts per shard to avoid information overload. An example of the output includes shard metadata, balancing status, and collection details .

- **Seeing Configuration Information**:
  All cluster configuration data is stored in collections in the `config` database, including information about shards, databases, and collections. By querying the config database through a `mongos` instance (not directly from config servers), administrators can inspect metadata. For instance, the `config.shards` collection tracks shard details, and `config.collections` provides details about sharded collections  .

#### Tracking Network Connections
- **Getting Connection Statistics**:
  The `connPoolStats` command gives detailed statistics on outgoing connections from `mongod` or `mongos` instances to other members of the cluster. This command provides information such as the number of connections in use, available connections, and total connections created. This is essential for monitoring network activity between cluster components and managing resource use .

- **Limiting the Number of Connections**:
  Each `mongos` process establishes connections to shards when forwarding client requests. Large numbers of `mongos` processes and clients can lead to excessive connections to shards. Administrators may need to limit or monitor these connections to prevent performance degradation caused by connection overloads  .

#### Server Administration
- **Adding Servers**:
  Adding a new server to a sharded cluster is done through the `sh.addShard()` command. This process ensures that the cluster's data capacity can grow, and the balancer automatically redistributes chunks to the new shard .

- **Changing Servers in a Shard**:
  Sometimes, specific servers within a shard need to be swapped out. This is accomplished through reconfiguration of the shard’s replica set, ensuring minimal disruption to the data stored within the shard .

- **Removing a Shard**:
  Removing a shard is a multi-step process involving migrating all data from the shard to other shards. The `removeShard` command initiates the draining process, and once completed, the shard can be safely removed. The primary shard for any databases must also be moved to another shard before the removal is finalized .

#### Balancing Data
- **The Balancer**:
  MongoDB's balancer is responsible for automatically redistributing chunks across shards to maintain an even data load. The balancer can be turned off using `sh.setBalancerState(false)` when performing administrative tasks. The balancer status is tracked in the `config.locks` collection, where administrators can monitor ongoing balancing operations  .

- **Changing Chunk Size**:
  The default chunk size is 64MB, but this can be altered in the `config.settings` collection. Adjusting chunk size impacts how data is split and migrated across the shards, which can optimize cluster performance under varying workloads  .

- **Moving Chunks**:
  The `sh.moveChunk()` command allows manual migration of chunks between shards. This can be useful for redistributing data manually in scenarios where automatic balancing might not suffice or when preparing for shard removal .

- **Jumbo Chunks**:
  Jumbo chunks are chunks that exceed the maximum chunk size but cannot be split. These chunks cannot be automatically migrated by the balancer, so they may cause uneven data distribution. Administrators may need to manually split or move jumbo chunks to avoid performance issues  .

- **Refreshing Configurations**:
  Configuration data for the cluster may need to be refreshed after changes (e.g., adding/removing shards or updating shard keys). This can be done using the `sh.refreshConfig()` command to ensure all `mongos` processes are aware of the latest cluster state .