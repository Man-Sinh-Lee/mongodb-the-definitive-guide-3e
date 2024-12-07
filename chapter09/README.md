### Chapter 9: Application Design

#### Schema Design Considerations: Schema Design Patterns

MongoDB’s flexible schema allows developers to design the database schema based on the specific needs of the application, instead of adhering to strict, predefined schemas. However, this flexibility requires careful consideration of schema design patterns to ensure performance, scalability, and maintainability.

Common **schema design patterns** in MongoDB include:

- **Embed**: Embedding related data into a single document to reduce the need for complex joins and improve read performance.
- **Reference**: Referencing related data in different documents to reduce duplication and avoid large, unwieldy documents.
  
For example, in an e-commerce application, you might choose to embed order items within an `orders` document:

```js
{
  "orderId": 123,
  "customerId": 456,
  "items": [
    { "productId": 1, "quantity": 2 },
    { "productId": 2, "quantity": 1 }
  ],
  "totalPrice": 50.00
}
```

This embedding allows the order and its items to be retrieved in a single query. However, in scenarios with high item repetition or updates, referencing could be more suitable.

#### Normalization Versus Denormalization

##### Examples of Data Representations
- **Normalization**: In normalized schema design, data is divided into multiple related collections to avoid redundancy and ensure that updates only need to happen in one place. For example, you may store users in one collection and their addresses in a separate collection.

  ```js
  // Users Collection
  {
    "_id": 1,
    "name": "Alice"
  }
  
  // Addresses Collection
  {
    "_id": 1,
    "userId": 1,
    "address": "123 Main St"
  }
  ```

  Queries would involve looking up data across multiple collections using `userId` as a reference.

- **Denormalization**: Denormalization involves embedding related data within a single document, which improves read performance at the expense of potential data redundancy. For example, embedding user addresses in the same document:

  ```js
  {
    "_id": 1,
    "name": "Alice",
    "address": "123 Main St"
  }
  ```

  This makes reading simpler and faster but could result in redundant data if the same address appears in multiple documents.

##### Cardinality
**Cardinality** refers to the nature of relationships between entities in your data model. There are three primary types:

- **One-to-one**: Each entity is related to exactly one other entity (e.g., a user profile and its settings).
- **One-to-many**: An entity is related to multiple other entities (e.g., a blog post and its comments).
- **Many-to-many**: Multiple entities are related to multiple other entities (e.g., users and groups).

In MongoDB, cardinality considerations influence whether you choose to embed documents or use references. For example, in a one-to-many relationship, you might embed related data, but in a many-to-many relationship, referencing is usually more efficient to avoid excessive document growth.

##### Friends, Followers, and Other Inconveniences
Social networks and similar applications with many-to-many relationships, such as **friends** or **followers**, present unique challenges. Embedding a list of friends in each user document may become impractical due to the unbounded nature of these lists.

Instead, you might use a **reference-based approach**, where each user document references other users:

```js
{
  "_id": 1,
  "name": "Alice",
  "friends": [2, 3, 4]  // List of user IDs
}
```

Alternatively, for a **followers** model, a separate collection can be used to represent these relationships:

```js
{
  "userId": 1,
  "followerId": 2
}
```

This allows for more scalable querying and better handling of large user bases.

#### Optimizations for Data Manipulation: Removing Old Data

In some applications, data may need to be removed after a certain time period or once it becomes irrelevant. MongoDB provides several ways to handle this:

- **Time-To-Live (TTL) Indexes**: TTL indexes automatically remove documents after a specified time. For example, a TTL index can be created to remove session data after 24 hours:

  ```js
  db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 86400 })
  ```

- **Batch Deletion**: For more control, you can run periodic batch deletion operations using MongoDB queries. For example, to remove records older than a certain date:

  ```js
  db.logs.deleteMany({ createdAt: { $lt: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000) } })  // Remove logs older than 7 days
  ```

These methods help manage storage and improve performance by reducing unnecessary data.

#### Planning Out Databases and Collections

When designing your application, careful consideration should be given to how databases and collections are organized. Factors to consider include:

- **Data size and growth**: Large datasets may benefit from being split across multiple collections or databases to improve performance.
- **Access patterns**: Frequently accessed data should be stored in a way that minimizes the number of queries needed to retrieve it.
- **Sharding**: For large-scale applications, consider whether data should be sharded across multiple servers to handle large datasets and high throughput.

For example, an e-commerce platform might organize its data into multiple collections (`users`, `products`, `orders`, etc.) based on access patterns.

#### Managing Consistency

In distributed systems, ensuring **data consistency** is critical, especially for applications with concurrent reads and writes. MongoDB provides several consistency mechanisms:

- **Write Concerns**: Define the level of acknowledgment you want from MongoDB when writing data. For example, `w: "majority"` ensures that a majority of replica set members confirm the write:

  ```js
  db.orders.insertOne({ orderId: 1, total: 100 }, { writeConcern: { w: "majority" } })
  ```

- **Read Concerns**: Specify the level of isolation for read operations. For example, `readConcern: "majority"` ensures that the read operation returns only data that has been acknowledged by a majority of nodes:

  ```js
  db.orders.find({ orderId: 1 }).readConcern("majority")
  ```

These controls ensure that your application maintains consistency even in complex distributed environments.

#### Migrating Schemas

As applications evolve, schema changes may be required, such as adding new fields, changing data structures, or moving from an embedded to a reference-based model. MongoDB’s flexible schema makes it easier to evolve your schema over time without downtime.

For example, to migrate a field from a string to an array:

```js
db.users.updateMany(
  { interests: { $exists: true, $type: "string" } },
  { $set: { interests: [ "$interests" ] } }
)
```

In this case, a string field `interests` is converted into an array.

#### Managing Schemas

Schema management in MongoDB can be done using schema validation rules, which enforce structure and data types at the collection level. For example, you can define a schema for a `users` collection to ensure that documents conform to expected formats:

```js
db.createCollection("users", {
  validator: {
    $jsSchema: {
      bsonType: "object",
      required: ["name", "email"],
      properties: {
        name: { bsonType: "string" },
        email: { bsonType: "string" }
      }
    }
  }
})
```

This schema validation ensures that documents must have a `name` and `email` field, both of which must be strings.

#### When Not to Use MongoDB

While MongoDB is a flexible, general-purpose database, there are cases where it might not be the best fit:

- **Complex Transactions**: MongoDB supports multi-document transactions, but if your application relies heavily on complex, multi-row transactions with strict ACID requirements, a relational database like PostgreSQL might be a better choice.
- **Relational Joins**: Applications that require complex joins across multiple tables may find relational databases more suitable, as MongoDB doesn’t support traditional joins natively (though it has the `$lookup` operator for basic joining).
- **Small, Highly Structured Data**: If your data model is small, static, and highly structured (e.g., traditional accounting systems), a relational database with strict schemas may offer more efficient storage and better tools for data integrity.

---

In summary, MongoDB offers flexibility in schema design, supporting both normalization and denormalization depending on the use case. By understanding cardinality, managing consistency, planning database structures, and optimizing schema designs, developers can build scalable, efficient applications with MongoDB. However, it’s important to recognize when MongoDB might not be the best fit, particularly in scenarios involving complex transactions or highly structured data.