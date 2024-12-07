### Chapter 5: Indexes

#### Introduction to Indexes
Indexes in MongoDB are data structures that improve the efficiency of query operations. Indexes store a small portion of the collection’s data in a specific format, which makes it faster to look up values. Without indexes, MongoDB must scan the entire collection to find documents, which is inefficient, especially for large datasets.

##### Creating an Index
Indexes are created using the `createIndex` method. For example, to create an index on the `name` field in a `users` collection:

```js
db.users.createIndex({ name: 1 })
```

Here, `1` represents ascending order. MongoDB uses this index to speed up queries that involve the `name` field, like `db.users.find({ name: "Alice" })`.

Indexes can also be created on multiple fields, allowing MongoDB to optimize more complex queries.

##### Introduction to Compound Indexes
Compound indexes are indexes that include more than one field. They allow MongoDB to handle queries that involve multiple fields efficiently. For example, to create an index on both `name` and `age` fields:

```js
db.users.createIndex({ name: 1, age: -1 })
```

This index can support queries on `name`, `age`, or both. However, MongoDB will prioritize the first field (`name`) when selecting how to use the index. Compound indexes can be particularly useful for sorting and filtering across multiple fields simultaneously.

##### How MongoDB Selects an Index
When a query is executed, MongoDB analyzes available indexes and selects the one that best fits the query. It looks at factors like:

- **Prefix matching**: MongoDB matches queries to the fields in a compound index from left to right. For example, an index on `{name: 1, age: -1}` will efficiently support queries like `db.users.find({ name: "Alice" })` or `db.users.find({ name: "Alice", age: { $gt: 25 } })`, but not `db.users.find({ age: 30 })`.
  
- **Query specificity**: MongoDB favors indexes that closely match the query criteria. If multiple indexes can be used, MongoDB picks the most selective one (i.e., the index that filters out the most data).

##### Using Compound Indexes
Compound indexes are useful when queries often involve multiple fields. For example, if your application frequently queries users by both `name` and `age`, a compound index will significantly improve query performance:

```js
db.users.createIndex({ name: 1, age: 1 })
```

MongoDB can also use parts of compound indexes for queries on individual fields. For example, the above index supports both `db.users.find({ name: "Alice" })` and `db.users.find({ name: "Alice", age: { $gt: 25 } })`, but it won’t optimize queries that only involve the `age` field.

##### How `$` Operators Use Indexes
MongoDB’s query operators, such as `$gt`, `$lt`, `$eq`, etc., use indexes when possible. For example, a query using `$gt` will utilize an index efficiently if it matches the indexed field:

```js
db.users.find({ age: { $gt: 25 } })
```

If an index exists on the `age` field, MongoDB will quickly locate documents where `age` is greater than 25 without scanning the entire collection.

##### Indexing Objects and Arrays
MongoDB supports indexing on fields that contain objects and arrays. For example, an index on an array field allows efficient querying for documents where an array contains a specific value:

```js
db.users.createIndex({ hobbies: 1 })
db.users.find({ hobbies: "reading" })
```

This index makes it faster to find users with "reading" as a hobby. Similarly, you can index on fields within embedded documents using dot notation:

```js
db.users.createIndex({ "address.city": 1 })
db.users.find({ "address.city": "New York" })
```

##### Index Cardinality
Index cardinality refers to the uniqueness of the values in an indexed field. Fields with high cardinality (many unique values, like user IDs) tend to perform better when indexed than fields with low cardinality (few unique values, like a `gender` field). Indexes with low cardinality may not significantly improve performance because many documents share the same value.

#### explain Output
The `explain` method provides insight into how MongoDB is using indexes during query execution. It shows whether an index is being used and how efficiently the query is running. For example, running `explain` on a query:

```js
db.users.find({ name: "Alice" }).explain("executionStats")
```

The output will include details like:

- Whether an index was used (`IXSCAN`)
- How many documents were scanned (`totalDocsExamined`)
- How many documents were returned (`nReturned`)

By analyzing this output, developers can optimize their queries and indexes for better performance.

#### When Not to Index
While indexes improve query performance, they come with trade-offs:

- **Increased write overhead**: Every time a document is inserted, updated, or deleted, MongoDB must also update the associated indexes. Too many indexes can slow down write operations.
  
- **Memory usage**: Indexes consume memory. Over-indexing a collection can lead to high memory usage, affecting overall system performance.

Indexes should not be used on fields with low cardinality or on fields that are rarely used in queries.

#### Types of Indexes

##### Unique Indexes
Unique indexes enforce uniqueness for values in the indexed field, meaning no two documents can have the same value for that field. This is useful for fields like email addresses or usernames:

```js
db.users.createIndex({ email: 1 }, { unique: true })
```

If an attempt is made to insert a document with a duplicate email, MongoDB will reject the operation.

##### Partial Indexes
Partial indexes index only documents that match a specified filter. This can be useful when indexing fields in large collections where only a subset of documents is queried often. For example, you can create a partial index on active users:

```js
db.users.createIndex(
   { name: 1 },
   { partialFilterExpression: { status: "active" } }
)
```

This index only includes documents where the `status` field is "active", making it more efficient for queries that focus on active users.

#### Index Administration

##### Identifying Indexes
You can view all indexes on a collection using the `getIndexes` method:

```js
db.users.getIndexes()
```

This command returns a list of all indexes, including the default `_id` index, and any additional indexes you’ve created.

##### Changing Indexes
If you need to modify an index, you must first drop the existing index and then recreate it with the new parameters. To drop an index, use the `dropIndex` method:

```js
db.users.dropIndex("name_1")
```

You can then create a new index with the updated configuration:

```js
db.users.createIndex({ name: 1, age: 1 })
```

Care should be taken when dropping and recreating indexes in production, as it can affect query performance during the operation.

---

By utilizing indexes efficiently, MongoDB users can significantly enhance query performance, particularly for large datasets. Understanding the trade-offs and knowing when and how to index properly is key to maintaining a balanced database that performs well under both read and write-heavy workloads.