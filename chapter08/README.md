### Chapter 8: Transactions

MongoDB’s **transactions** feature allows for multi-document operations to be executed with **ACID** properties, ensuring data consistency and integrity across multiple operations. This is particularly useful in scenarios where a sequence of operations must succeed or fail together, such as in financial transactions or any other critical processes that involve modifying multiple documents or collections.

#### Introduction to Transactions

##### A Definition of ACID
**ACID** is a set of properties that ensure reliable processing of database transactions:

- **Atomicity**: Ensures that either all operations within a transaction are completed successfully, or none are. If a transaction fails, no partial operations are committed.
- **Consistency**: Transactions must transition the database from one valid state to another, maintaining the integrity of the data.
- **Isolation**: Ensures that the operations within a transaction are isolated from other concurrent operations, preventing "dirty reads" or interference.
- **Durability**: Once a transaction is committed, its changes are permanently saved, even in the case of a system failure.

MongoDB supports **multi-document transactions** starting from version 4.0, which means that operations involving multiple documents or collections can be grouped together and either committed or rolled back as a unit, ensuring ACID properties.

#### How to Use Transactions

Transactions are initiated by starting a session and then beginning a transaction. Within the transaction, multiple operations (like inserts, updates, or deletes) can be performed across multiple collections or documents. Once all operations are successful, the transaction is committed; if any operation fails, the transaction can be rolled back.

Here is an example of how to use transactions in MongoDB:

1. **Start a Session**: First, a session is started, which acts as the context for the transaction.
   
   ```js
   const session = db.getMongo().startSession();
   ```

2. **Start the Transaction**: Begin the transaction within the session.

   ```js
   session.startTransaction();
   ```

3. **Perform Operations**: Execute the necessary operations inside the transaction.

   ```js
   try {
     db.customers.updateOne({ _id: 1 }, { $set: { balance: 100 } }, { session });
     db.orders.insertOne({ customerId: 1, amount: 100 }, { session });
   } catch (error) {
     // Abort transaction if any operation fails
     session.abortTransaction();
     throw error;  // Rethrow the error to handle it appropriately
   }
   ```

4. **Commit the Transaction**: If all operations succeed, commit the transaction to save the changes.

   ```js
   session.commitTransaction();
   session.endSession();
   ```

5. **Abort the Transaction**: If there is a failure, rollback (abort) the transaction to undo any changes.

   ```js
   session.abortTransaction();
   session.endSession();
   ```

In this example, the `customers` collection is updated to set a balance of 100 for the customer with `_id: 1`, and an order document is inserted into the `orders` collection. Both operations are part of a transaction, ensuring that either both succeed or both fail.

#### Tuning Transaction Limits for Your Application

MongoDB imposes some limits on transactions that you should be aware of, particularly for large-scale applications. Understanding these limits can help you optimize the performance and reliability of transactions in your application.

##### Timing and Oplog Size Limits

- **Transaction Timeout**: By default, transactions are limited to a maximum duration of **60 seconds**. If a transaction takes longer than this to commit, it will be aborted. This ensures that long-running transactions do not block the system or consume excessive resources. You can tune this value to match your application’s needs by adjusting the `maxTransactionLockRequestTimeoutMillis` setting.

  For example, you might adjust the timeout based on the complexity and length of the transaction process:

  ```js
  mongod --setParameter maxTransactionLockRequestTimeoutMillis=120000  # 2 minutes
  ```

- **Oplog Size Limits**: Transactions are also limited by the size of the **oplog** (the log used to replicate operations in a replica set). If the operations performed in a transaction exceed the available oplog space, the transaction will fail. This is particularly important in replica sets where the oplog has a finite size, and it is shared with all other operations.

  You can mitigate oplog size issues by increasing the size of the oplog or by keeping transactions small and efficient:

  ```js
  mongod --oplogSize 10240  # Set oplog size to 10GB
  ```

- **Document Size Limits**: The total size of the documents modified in a transaction must fit within MongoDB’s document size limit (16MB). This includes updates to embedded documents or fields. If you are updating many large documents within a single transaction, it’s important to ensure that you do not exceed this limit.

In general, you should aim to keep transactions short and only include the operations that are absolutely necessary. Large or long-running transactions can consume significant resources and cause contention with other operations.

#### Best Practices for Transactions
- **Use Transactions When Necessary**: Only use transactions for critical operations that require multi-document atomicity. Single-document operations in MongoDB are already atomic and do not require transactions.
  
- **Keep Transactions Short**: Long-running transactions can cause performance bottlenecks and increase the likelihood of lock contention. Break down large processes into smaller, faster transactions where possible.
  
- **Monitor Transaction Performance**: Regularly monitor transaction durations, oplog usage, and resource consumption to ensure that transactions are running efficiently without causing system degradation.

---

By understanding MongoDB’s transaction capabilities and the associated limits, developers can ensure that their applications remain reliable and efficient. Transactions in MongoDB bring full ACID compliance to complex operations involving multiple documents or collections, making them a powerful tool for maintaining data integrity in critical workflows.