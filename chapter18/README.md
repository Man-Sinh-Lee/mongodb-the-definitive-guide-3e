### Chapter 18: Seeing What Your Application Is Doing

#### Seeing the Current Operations

1. **Finding Problematic Operations**:
   MongoDB provides the `db.currentOp()` function to view active operations, which is useful for identifying slow-running operations. For example:
   ```js
   db.currentOp({ "active": true, "secs_running": { "$gt": 3 } });
   ```
   This shows all operations that have been running for more than 3 seconds, helping pinpoint slow operations such as missing indexes or inefficient queries .

2. **Killing Operations**:
   To stop problematic operations, MongoDB allows the use of `db.killOp(opid)`, where `opid` is the operation’s unique identifier. Not all operations can be killed, especially those waiting for locks .

3. **False Positives**:
   Long-running internal operations, such as replication or writeback listeners, may show up as slow operations but can be ignored. Killing these processes can briefly halt replication, so they should be handled carefully .

4. **Preventing Phantom Operations**:
   Phantom operations occur when unacknowledged writes back up in the operating system's buffer. They continue even after a client stops sending them. This can be avoided by using acknowledged writes, where each write waits for the previous one to be completed  .

#### Using the System Profiler

The **system profiler** records detailed information about all operations that take longer than a specified threshold. To turn on the profiler:
```js
db.setProfilingLevel(2);  // Profiling all operations
```
You can limit profiling to slower operations with a threshold:
```js
db.setProfilingLevel(1, 500);  // Operations taking longer than 500ms
```
The profiler slows down overall performance, so it’s best used selectively  .

#### Calculating Sizes

1. **Documents**:
   To calculate the size of a document, MongoDB provides the `Object.bsonsize()` function. For example:
   ```js
   Object.bsonsize(db.users.findOne());
   ```
   This function returns the size of the document in bytes, which is useful for capacity planning.

2. **Collections**:
   Collection statistics, such as the number of documents, average object size, and total storage size, can be retrieved with `db.collection.stats()`. Example:
   ```js
   db.movies.stats();
   ```
   This helps track the space utilization of a collection.

3. **Databases**:
   To view database-level statistics, use `db.stats()`, which provides details like total data size, storage size, and the number of documents across all collections .

#### Using mongotop and mongostat

1. **mongotop**:
   This command provides insight into how much time is spent reading and writing data in each collection. It gives real-time statistics about the busiest collections in the system.

2. **mongostat**:
   The `mongostat` command provides overall server statistics, including counts of inserts, updates, and queries per second. Example fields include:
   - `insert/query/update/delete`: Counts of these operations since the last output.
   - `conn`: Number of active connections.
   - `vsize`: Amount of virtual memory MongoDB is using.