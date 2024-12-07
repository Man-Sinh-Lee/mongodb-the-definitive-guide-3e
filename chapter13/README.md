### Chapter 13: Administration

#### Starting Members in Standalone Mode
To perform certain maintenance tasks (like modifying data), it is often necessary to start a replica set member in standalone mode. This involves restarting the server without the `replSet` option and running it on a different port so that it won’t be recognized by the replica set as a member. The other members will assume it is down. After completing the necessary changes, you can shut it down and restart it with the original options, allowing the member to sync up with the replica set  .

#### Replica Set Configuration

1. **Creating a Replica Set**: 
   When creating a replica set, you need to start all the `mongod` processes for the members and initiate the set by passing a configuration document to one member using `rs.initiate()`. This member will propagate the configuration to the rest of the set .

2. **Changing Set Members**:
   You can add new members using `rs.add()` and remove them using `rs.remove()`. For example:
   ```js
   rs.add("newHost:27017")
   ```
   Members can also be reconfigured using `rs.reconfig()` to change attributes like priority or host name  .

3. **Creating Larger Sets**: 
   MongoDB supports up to 50 members in a replica set but limits voting members to seven. Non-voting members are useful for distributing reads or providing backups .

4. **Forcing Reconfiguration**: 
   If a replica set loses its primary and a majority of members, it can be reconfigured by force from a secondary using the `force` option in the reconfig command. This ensures that a valid configuration is sent and accepted by the remaining members  .

#### Manipulating Member State

1. **Turning Primaries into Secondaries**:
   A primary can be demoted to a secondary using the `rs.stepDown()` command. For example:
   ```js
   rs.stepDown(60)  // 60 seconds as a secondary
   ```
   This can be used when maintenance or reconfiguration is required .

2. **Preventing Elections**:
   To prevent elections temporarily, you can use the `rs.freeze()` command. This is useful when performing maintenance on a primary, preventing secondaries from trying to become the new primary  .

#### Monitoring Replication

1. **Getting the Status**:
   The `rs.status()` command provides comprehensive information about the state of each member in the replica set. This includes their roles (e.g., primary, secondary), replication lag, and more.

2. **Visualizing the Replication Graph**:
   Running `rs.status()` on secondaries can show the member that a node is syncing from (via the `syncingTo` field). This helps to identify the replication hierarchy within the set, useful for understanding the flow of data .

3. **Replication Loops and Disabling Chaining**:
   MongoDB's default replication behavior allows secondaries to replicate from other secondaries if necessary, creating a chain. While this minimizes load on the primary, it can increase replication lag. The `replSetSyncFrom()` command can be used to specify a different member to sync from if needed  .

4. **Calculating Lag**:
   Replication lag is the delay between when an operation is written on the primary and when it is applied on a secondary. This can be monitored using the timestamps provided by `rs.status()`.

5. **Resizing the Oplog**:
   The oplog size can affect replication. A larger oplog provides a greater window for secondaries to catch up if they fall behind. This can be configured to ensure better replication efficiency during heavy load periods .

6. **Building Indexes**:
   Indexes can be built on secondaries to offload the primary. This involves temporarily converting the secondary to standalone mode, building the index, and then re-joining the set .

7. **Replication on a Budget**:
   In cost-constrained environments, you can configure secondaries with lower hardware specifications to serve only as backup members. By setting their priority to 0 and hiding them from client traffic, these secondaries can act as disaster recovery nodes without taking on primary duties  .