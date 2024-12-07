### Chapter 24: Deploying MongoDB

Deploying MongoDB in a production environment requires careful consideration of hardware, operating system configurations, and network settings to ensure high performance, reliability, and scalability. This chapter discusses the key factors and best practices involved in designing and configuring a system for MongoDB.

#### Designing the System

##### Choosing a Storage Medium

The choice of storage medium directly impacts MongoDB’s performance. **SSD (Solid-State Drives)** are strongly recommended for production environments due to their low latency and high throughput compared to traditional HDDs (Hard Disk Drives). SSDs help handle the heavy I/O workload of MongoDB, especially for write-heavy applications.

##### Recommended RAID Configurations

RAID (Redundant Array of Independent Disks) configurations provide data redundancy and performance benefits. Common RAID configurations for MongoDB include:

- **RAID 10**: Combines mirroring and striping, offering both data redundancy and improved performance. This is the preferred RAID setup for MongoDB.
- **RAID 0**: Provides high performance by striping data across multiple drives but offers no redundancy. Not recommended for production use due to the risk of data loss.
- **RAID 5/6**: Offers redundancy through parity, but performance is lower than RAID 10 due to the overhead of calculating parity.

##### CPU

MongoDB is designed to utilize multi-core processors efficiently. For most workloads, the number of CPU cores should scale with the anticipated read and write load. A multi-core processor (at least 8 cores) is recommended for handling concurrent requests, queries, and background tasks like index building.

##### Operating System

Linux is the recommended operating system for MongoDB in production, specifically distributions such as **Ubuntu**, **Red Hat**, or **CentOS**. Linux provides better performance and more control over system configurations. Windows can be used but is less commonly chosen for production.

##### Swap Space

While MongoDB benefits from having sufficient physical RAM, it is still important to configure swap space as a fallback in case memory is exhausted. However, MongoDB’s performance degrades significantly if it starts using swap, so it's better to have enough RAM to avoid swapping altogether.

##### Filesystem

For MongoDB’s data storage, **XFS** is the recommended filesystem. XFS is optimized for high-performance workloads, especially those involving large files and heavy I/O. **EXT4** can also be used, but XFS tends to provide better performance for MongoDB’s specific requirements.

#### Virtualization

Virtualization is commonly used in modern deployments, but MongoDB has some specific considerations when running on virtualized infrastructure.

##### Memory Overcommitting

Memory overcommitting in virtualized environments (where more memory is allocated to virtual machines than is physically available) can lead to performance issues. It’s critical to ensure that enough memory is reserved for MongoDB’s working set and that the system does not rely on swap.

##### Mystery Memory

In virtualized environments, “mystery memory” refers to memory that seems to disappear due to over-committing or misconfigured hypervisors. To mitigate this, monitor memory usage carefully and ensure that the virtual machine has sufficient dedicated RAM.

##### Handling Network Disk I/O Issues

Using network-attached storage (NAS) or storage area networks (SANs) can introduce latency and I/O bottlenecks. For optimal performance, it is recommended to use **local disks** for MongoDB’s data storage to avoid network-related delays.

##### Using Non-Networked Disks

If using virtualization, prefer **non-networked disks** (e.g., local SSDs) over networked disks like NFS or iSCSI. Non-networked disks provide lower latency and higher throughput, which is crucial for MongoDB’s performance.

#### Configuring System Settings

To optimize MongoDB’s performance, several system settings need to be adjusted.

##### Turning Off NUMA

**NUMA (Non-Uniform Memory Access)** can cause unpredictable memory allocation behavior in MongoDB. It is recommended to disable NUMA in the BIOS or use the `numactl` command to ensure that MongoDB uses memory efficiently.

```js
numactl --interleave=all mongod
```

This command ensures that MongoDB uses memory evenly across NUMA nodes.

##### Setting Readahead

The **readahead** value determines how much data the operating system reads in advance when accessing the disk. A smaller readahead value (16 to 32 blocks) is typically better for MongoDB, as it prevents the OS from reading unnecessary data during random I/O operations.

To adjust the readahead value:

```js
sudo blockdev --setra 32 /dev/sda
```

##### Disabling Transparent Huge Pages (THP)

**Transparent Huge Pages (THP)** can introduce performance issues in MongoDB by causing latency spikes. It is recommended to disable THP:

```js
echo never > /sys/kernel/mm/transparent_hugepage/enabled
```

##### Choosing a Disk Scheduling Algorithm

For MongoDB workloads, the **noop** or **deadline** I/O scheduler is recommended, as they are designed for SSDs and low-latency environments. To set the disk scheduler:

```js
echo noop > /sys/block/sda/queue/scheduler
```

##### Disabling Access Time Tracking

**Access time tracking** (`atime`) updates the file access time with each read, causing unnecessary write operations. Disable `atime` by mounting the filesystem with the `noatime` option:

```js
mount -o noatime /dev/sda1 /data/db
```

##### Modifying Limits

Increase the number of open file descriptors and processes available to MongoDB by editing the `/etc/security/limits.conf` file. For example:

```js
mongod soft nofile 64000
mongod hard nofile 64000
```

This ensures MongoDB can handle a large number of connections and files.

#### Configuring Your Network

Network performance is crucial in distributed MongoDB deployments, such as replica sets and sharded clusters.

- **Reduce Latency**: Place MongoDB instances on low-latency networks, preferably within the same data center.
- **Use Jumbo Frames**: Configure the network to use **jumbo frames** (9000 bytes MTU) if supported by the network infrastructure, to reduce overhead in large data transfers.
- **TCP Settings**: Optimize TCP settings, such as enabling **TCP keepalive** to detect and close dead connections more quickly.

```js
echo 1 > /proc/sys/net/ipv4/tcp_keepalive_time
```

#### System Housekeeping

Regular system housekeeping tasks can prevent unexpected performance drops or outages.

##### Synchronizing Clocks

Ensure that all MongoDB nodes in a replica set or sharded cluster have synchronized clocks using **NTP (Network Time Protocol)**. This is essential for accurate logging and replica set elections.

```js
sudo apt-get install ntp
```

##### The OOM Killer

The **Out-Of-Memory (OOM) Killer** is a Linux mechanism that terminates processes when the system runs out of memory. MongoDB processes can be inadvertently killed by the OOM Killer. To prevent this, adjust the **OOM score** of the MongoDB process:

```js
echo -1000 > /proc/$(pidof mongod)/oom_score_adj
```

This lowers the priority of MongoDB for OOM termination.

##### Turn Off Periodic Tasks

Certain background tasks, such as cron jobs or package updates, can consume resources and interfere with MongoDB’s performance. It’s a good practice to minimize unnecessary background tasks on production servers to ensure MongoDB has dedicated access to system resources.

---

In summary, deploying MongoDB in production involves choosing the right hardware (e.g., SSDs, RAID 10, multi-core CPUs), configuring the operating system for optimal performance (disabling NUMA, setting readahead, adjusting file descriptor limits), and carefully tuning network settings to minimize latency. For virtualized environments, it is important to manage memory carefully and avoid network-based storage where possible. By following these best practices, you can ensure that MongoDB performs reliably and efficiently in production settings.