### Chapter 22: Monitoring MongoDB

Monitoring MongoDB is essential to ensure that the database performs optimally, uses system resources efficiently, and remains highly available. This chapter covers monitoring key aspects such as memory usage, replication status, and tracking performance.

#### Monitoring Memory Usage

MongoDB’s performance is heavily influenced by how it uses system memory, especially since it relies on the operating system’s file system cache for efficient data retrieval. Monitoring memory usage helps ensure that MongoDB has enough resources to handle read and write workloads effectively.

##### Introduction to Computer Memory

Computer memory is divided into **physical memory (RAM)** and **virtual memory (swap)**. MongoDB benefits significantly from having enough physical memory because frequently accessed data (the **working set**) is kept in memory, reducing disk I/O. Virtual memory, on the other hand, can slow down MongoDB performance if the system is forced to swap data to disk frequently.

##### Tracking Memory Usage

To monitor MongoDB’s memory usage, you can use the `db.serverStatus()` command, which provides detailed statistics about the server, including memory consumption:

```js
db.serverStatus().mem
```

This command returns information about:

- **resident**: The amount of memory MongoDB is using that resides in physical memory (RAM).
- **virtual**: The total amount of memory MongoDB is using, including both physical and swap memory.
- **mapped**: The amount of memory used for memory-mapped files, such as the WiredTiger cache.

Monitoring these metrics helps track how much data is kept in memory versus being read from disk.

##### Tracking Page Faults

**Page faults** occur when MongoDB attempts to access data that is not currently in memory, causing the operating system to retrieve the data from disk. High page fault rates indicate that MongoDB is frequently reading data from disk, which can slow down performance.

You can track page faults using:

```js
db.serverStatus().extra_info.page_faults
```

A high number of page faults may indicate that the working set size is larger than the available memory, leading to performance issues. In such cases, consider adding more RAM or optimizing the data model to reduce memory consumption.

##### I/O wait

**I/O wait** measures how long the CPU is waiting for disk I/O operations to complete. High I/O wait times can indicate that MongoDB is constrained by disk performance, often due to insufficient memory or slow disk access.

You can monitor I/O wait through system tools like `iostat` on Linux systems:

```js
iostat -x
```

Look for the `await` and `%iowait` metrics, which indicate how long I/O operations take and how much time the CPU spends waiting for I/O. High values suggest a bottleneck in disk performance.

#### Calculating the Working Set

The **working set** refers to the subset of data that is frequently accessed by MongoDB. Ideally, the working set fits entirely within the available physical memory to minimize disk access.

To estimate the working set size, you can monitor the **cache utilization** using WiredTiger’s cache statistics:

```js
db.serverStatus().wiredTiger.cache
```

Look for the following metrics:

- **bytes currently in cache**: The amount of data currently stored in the WiredTiger cache.
- **maximum bytes configured**: The maximum size of the WiredTiger cache, which is typically 50% of the system’s RAM by default.

If the **bytes currently in cache** is close to the **maximum bytes configured**, and you notice a high number of page faults, it indicates that the working set is larger than the available memory. In such cases, consider optimizing queries, adding indexes, or increasing memory.

#### Tracking Performance

Monitoring MongoDB’s performance is crucial for identifying and addressing bottlenecks in queries, writes, and other operations.

Key metrics for tracking performance include:

- **Query execution time**: Slow queries can significantly impact performance. The MongoDB profiler helps identify slow queries:

  ```js
  db.setProfilingLevel(1, { slowms: 100 })
  db.system.profile.find().sort({ ts: -1 }).limit(5)  // Retrieve recent slow queries
  ```

  This sets the profiling level to log queries that take longer than 100ms. You can then inspect the `system.profile` collection to identify and optimize slow queries.

- **Operation throughput**: Track the number of queries, inserts, updates, and deletes per second using `db.serverStatus()`:

  ```js
  db.serverStatus().opcounters
  ```

  This shows the current throughput of database operations, helping to monitor how the system is handling workloads.

- **Locking**: Track the amount of time MongoDB spends waiting on locks using:

  ```js
  db.serverStatus().locks
  ```

  High lock contention can slow down operations, especially in write-heavy workloads.

#### Tracking Free Space

Disk space is a critical resource for MongoDB, as insufficient disk space can cause performance degradation and even outages. Monitoring disk usage helps ensure that the database has enough space to grow as new data is added.

Use the `df` command on Linux systems to monitor available disk space:

```js
df -h /data/db
```

To check MongoDB’s internal storage statistics, use the `db.stats()` command:

```js
db.stats()
```

This returns information about:

- **storageSize**: The total size of the data in the database.
- **freeStorageSize**: The amount of free space remaining.
- **indexSize**: The total size of the indexes in the database.

Regularly monitor disk space to avoid running out of storage, which can lead to crashes or performance issues.

#### Monitoring Replication

In replica sets, it is important to monitor replication to ensure data consistency and availability across members. Key metrics to track include **replication lag**, **oplog size**, and **replica set health**.

- **Replication lag**: The time it takes for secondary members to replicate data from the primary. High replication lag can result in outdated data on secondaries and a delayed failover in case the primary fails. You can track replication lag using:

  ```js
  rs.printSlaveReplicationInfo()
  ```

  This command shows the replication delay for each secondary, including how far behind they are in terms of operations.

- **Oplog size**: The oplog (operations log) stores the operations to be replicated to secondaries. Monitor the oplog size using:

  ```js
  db.getReplicationInfo()
  ```

  This provides the size and usage of the oplog. If the oplog is too small, it can roll over before secondaries have a chance to catch up, causing replication issues.

- **Replica set health**: Check the overall health and status of your replica set using:

  ```js
  rs.status()
  ```

  This command provides a detailed view of each member’s state, whether they are the primary or secondary, and if any members are unreachable or unhealthy.

Monitoring replication ensures that data is synchronized across members, and secondary members are ready to take over if the primary fails.

---

In summary, monitoring MongoDB is critical for maintaining optimal performance and availability in production environments. By tracking memory usage, page faults, I/O wait times, and working set size, administrators can ensure efficient resource utilization. Monitoring replication ensures data consistency and high availability, while regular checks on performance metrics like query execution time, locking, and throughput help identify and address bottlenecks. Finally, keeping an eye on disk space prevents storage-related issues that could affect database uptime.