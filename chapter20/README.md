### Chapter 20: Durability

Durability is a critical property in database systems that ensures data is permanently stored and remains consistent, even in the case of a crash or system failure. MongoDB provides several mechanisms to achieve durability at different levels, from individual members to the entire cluster.

#### Durability at the Member Level Through Journaling

At the **member level**, MongoDB ensures durability through **journaling**. Journaling records changes in a **write-ahead log** (journal file) before applying them to the database files. In the event of a crash, MongoDB can recover the data by replaying the journal files.

For example, when journaling is enabled, if MongoDB crashes after performing a write operation but before the data is fully written to disk, the journal will contain a record of the operation, and MongoDB can recover it upon restarting.

Journaling is enabled by default, but it can be explicitly turned on using:

```js
mongod --journal
```

The journal records are flushed to disk every 100ms by default, providing a balance between performance and durability.

#### Durability at the Cluster Level Using Write Concern

**Write concern** defines the level of acknowledgment required from MongoDB when performing write operations. It ensures that data is not only written to the primary but, depending on the configuration, may also be replicated to secondary members, increasing durability at the **cluster level**.

##### The `w` and `wtimeout` Options for `writeConcern`

- **`w`**: Specifies the number of members that must acknowledge a write before it is considered successful. For example, setting `w: 1` ensures only the primary acknowledges the write, while `w: "majority"` ensures a majority of replica set members have written the data.

  ```js
  db.orders.insertOne({ orderId: 123, amount: 50 }, { writeConcern: { w: "majority" } })
  ```

  This write is acknowledged only when the majority of members in the replica set confirm the write.

- **`wtimeout`**: Specifies the maximum time (in milliseconds) to wait for the `w` write concern to be satisfied. If the timeout is exceeded, the operation will fail.

  ```js
  db.orders.insertOne({ orderId: 124, amount: 75 }, { writeConcern: { w: 2, wtimeout: 5000 } })
  ```

  This ensures that at least two replica set members confirm the write within 5 seconds. If the condition is not met within the `wtimeout`, the operation will return an error.

##### The `j` (Journaling) Option for `writeConcern`

The **`j`** option ensures that writes are acknowledged only after they are written to the journal, adding an extra layer of durability. When `j: true`, MongoDB waits for the write operation to be written to the journal on disk before acknowledging the operation.

```js
db.orders.insertOne({ orderId: 125, amount: 100 }, { writeConcern: { j: true } })
```

This guarantees that the data is safely stored in the journal, making it durable even if the system crashes before the data is written to the database files.

#### Durability at a Cluster Level Using Read Concern

**Read concern** determines the level of isolation and consistency for read operations. It ensures that data read from the cluster is durable and reflects a certain level of replication or acknowledgment across the cluster.

- **`local`** (default): Reads the most recent data from the primary, which may not have been replicated to secondaries yet.
  
- **`majority`**: Ensures that the data read reflects the majority of replica set members having acknowledged the write. This guarantees that the data is durable and will not be lost even if the primary fails.

  ```js
  db.orders.find({ orderId: 125 }).readConcern("majority")
  ```

  This read concern ensures that the returned data reflects writes that have been acknowledged by the majority of members.

#### Durability of Transactions Using a Write Concern

In **transactions**, durability is ensured by using an appropriate **write concern**. Since transactions in MongoDB can involve multiple operations across different collections, it is crucial to ensure that the entire transaction is durable upon commit.

For example, to ensure that a transaction’s writes are acknowledged by a majority of the replica set members, you would set the write concern within the transaction:

```js
const session = db.getMongo().startSession();
session.startTransaction({ writeConcern: { w: "majority" } });

try {
  db.orders.insertOne({ orderId: 126, amount: 150 }, { session });
  db.payments.insertOne({ orderId: 126, status: "paid" }, { session });
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
}
```

Here, the transaction ensures that both the `orders` and `payments` collections have the write acknowledged by the majority of the replica set members.

#### What MongoDB Does Not Guarantee

While MongoDB provides robust mechanisms for ensuring durability, there are certain scenarios where it cannot guarantee full data persistence:

- **Rollbacks**: In the event of a **rollback** (when a secondary that became primary due to an election discovers it was out of sync), some acknowledged writes might be lost. For this reason, it’s important to use `writeConcern: "majority"` to minimize this risk.
  
- **Write Acknowledgment Without Sync**: If `writeConcern` is not used or set to `w: 1`, MongoDB only acknowledges the write on the primary and does not guarantee that the data is replicated to the secondaries or written to disk.

#### Checking for Corruption

MongoDB includes built-in tools to check for database **corruption**, ensuring that the data remains consistent and free from errors.

- **Repair**: The `repairDatabase()` command attempts to fix any corruption by rebuilding indexes and collections.

  ```js
  db.repairDatabase()
  ```

  This command should be used with caution, especially on large datasets, as it can be time-consuming.

- **Storage Engine Tools**: MongoDB’s **WiredTiger** storage engine includes mechanisms to detect and correct corruption issues during normal operation. For example, the `validate()` command checks for data corruption in a collection:

  ```js
  db.orders.validate()
  ```

  This command returns a report on whether any issues were found with the data in the `orders` collection, and whether the collection is structurally sound.

---

In summary, MongoDB provides a range of tools and configuration options to ensure **durability** at both the member and cluster levels. Journaling ensures that data is preserved at the member level, while write and read concerns provide cluster-level guarantees. Durability can be fine-tuned for transactions and regular operations to balance performance and safety. However, MongoDB has some limitations, especially during rollbacks, so understanding the nuances of write concerns is essential for maintaining a durable system. Finally, MongoDB offers tools for checking and addressing data corruption, ensuring data integrity over time.