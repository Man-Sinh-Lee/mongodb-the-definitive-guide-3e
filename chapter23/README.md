### Chapter 23: Making Backups

Backing up MongoDB is critical to protect data from accidental loss, corruption, or hardware failure. MongoDB provides several methods to back up and restore data. This chapter covers backup strategies, methods for single servers, replica sets, and sharded clusters.

#### Backup Methods

MongoDB offers various methods for creating backups, each suited to different use cases:

1. **Filesystem Snapshots**: Fast and efficient backups that use the operating system’s snapshot features to capture the state of the MongoDB data directory.
2. **Copying Data Files**: Manually copying MongoDB’s data files from the data directory to another location. This method can be risky if MongoDB is not properly shut down.
3. **`mongodump` and `mongorestore`**: Utility tools provided by MongoDB that create logical backups by exporting data into BSON files, which can later be restored.

#### Backing Up a Server

Backing up a single MongoDB server is straightforward but requires careful consideration of consistency and data integrity.

##### Filesystem Snapshot

**Filesystem snapshots** are the fastest way to back up large databases. They are typically created using tools provided by the operating system, such as **LVM** on Linux. Snapshots capture the state of the entire file system at a given moment.

Steps to take a snapshot:

1. **Freeze MongoDB Writes**: You can ensure consistency by freezing MongoDB writes before taking the snapshot. Use the `fsync` command with the `lock` option:

   ```js
   db.fsyncLock()
   ```

   This prevents MongoDB from writing to the disk while the snapshot is being taken.

2. **Take the Snapshot**: Use the OS tool to take the snapshot, such as `lvcreate` for LVM on Linux:

   ```js
   lvcreate --size 10G --snapshot --name mdb-backup /dev/vg0/mongodb
   ```

3. **Unlock MongoDB**: After the snapshot is completed, unlock the database to resume normal operations:

   ```js
   db.fsyncUnlock()
   ```

Filesystem snapshots provide a fast and consistent backup method without significantly impacting the database’s availability.

##### Copying Data Files

Copying MongoDB’s data files manually can be an effective backup method for smaller datasets, but requires caution to ensure consistency:

1. **Shutdown MongoDB**: To avoid any file corruption, MongoDB should be properly shut down before copying the data files:

   ```js
   sudo systemctl stop mongod
   ```

2. **Copy Data Files**: Once MongoDB is shut down, copy the contents of the `dbpath` (e.g., `/var/lib/mongodb/`):

   ```js
   cp -r /var/lib/mongodb /backup/mongodb-backup
   ```

3. **Restart MongoDB**: After copying, restart the MongoDB server:

   ```js
   sudo systemctl start mongod
   ```

This method ensures that data files are copied in a consistent state, but it may cause downtime as MongoDB needs to be stopped during the backup process.

##### Using `mongodump`

**`mongodump`** creates a logical backup by exporting the data to BSON format. It is ideal for backups that need to be portable or restored to other MongoDB instances.

Example of using `mongodump`:

```js
mongodump --db myDatabase --out /backup/mongodb-backup
```

This command exports the `myDatabase` database to the `/backup/mongodb-backup` directory. You can also back up all databases without specifying `--db`.

To restore the backup, use **`mongorestore`**:

```js
mongorestore /backup/mongodb-backup
```

`mongodump` is useful for small to medium-sized datasets and offers flexibility in selective restoration, but it can be slower for large datasets compared to filesystem snapshots.

#### Specific Considerations for Replica Sets

When backing up a **replica set**, MongoDB ensures that the backup is consistent across multiple nodes. The preferred backup method is to take backups from a **secondary** member, as this reduces the load on the primary node.

Steps for backing up a secondary:

1. **Freeze Writes**: Lock the secondary node using `fsyncLock` to ensure data consistency:

   ```js
   db.fsyncLock()
   ```

2. **Take the Backup**: Use any backup method (filesystem snapshot, copying data files, or `mongodump`) to back up the data.

3. **Unlock Writes**: After the backup is complete, unlock the secondary node:

   ```js
   db.fsyncUnlock()
   ```

Using a secondary node for backups ensures that the primary remains unaffected, reducing the impact on live traffic. However, it is important to monitor the replication lag to avoid backing up an out-of-date secondary.

#### Specific Considerations for Sharded Clusters

Backing up **sharded clusters** introduces additional complexity, as each shard stores a subset of the data, and the **config servers** store metadata about the cluster.

##### Backing Up and Restoring an Entire Cluster

For sharded clusters, MongoDB recommends using **`mongodump`** and **`mongorestore`** or taking **consistent filesystem snapshots** from each shard and config server. It is important to coordinate the backup across all components of the cluster to ensure consistency.

1. **Back Up the Config Servers**: Config servers contain the metadata of the cluster, so they must be backed up first.
   
   ```js
   mongodump --db config --out /backup/config-backup
   ```

2. **Back Up Each Shard**: Use `mongodump` or filesystem snapshots to back up each shard in parallel. Ensure consistency across the shards by locking or freezing writes.

3. **Restore the Cluster**: When restoring, you must first restore the config servers and then restore each shard.

Example for restoring the config servers:

```js
mongorestore --db config /backup/config-backup
```

Next, restore each shard using the corresponding backups.

##### Backing Up and Restoring a Single Shard

If you only need to back up a single shard, the process is simpler but requires coordination with the cluster. You can back up and restore a single shard without affecting the rest of the cluster.

1. **Back Up the Shard**: Use any backup method to back up the data from the shard, ensuring it is consistent.

2. **Restore the Shard**: When restoring, ensure that the shard is re-synchronized with the rest of the cluster, and any chunk metadata is updated if needed.

For example, to restore a single shard using `mongorestore`:

```js
mongorestore --db myDatabase /backup/shard-backup
```

This restores the data in the shard while keeping the overall cluster intact.

---

In summary, MongoDB offers multiple backup methods, including filesystem snapshots, copying data files, and using `mongodump`. Each method has its use cases, and the choice depends on the size of the dataset and the need for consistency. For replica sets, it is recommended to back up from a secondary node to reduce load on the primary, while sharded clusters require coordinated backups across all shards and config servers. Ensuring regular, consistent backups is essential for preventing data loss and ensuring that MongoDB can be restored in case of failure.