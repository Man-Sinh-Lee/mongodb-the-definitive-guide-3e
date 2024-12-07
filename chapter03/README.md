### Chapter 3: Creating, Updating, and Deleting Documents

#### Inserting Documents

##### `insertMany`
The `insertMany` function allows you to insert multiple documents at once into a collection. This function improves efficiency by reducing the number of separate insert operations needed. It accepts an array of documents as input. For example:

```js
db.users.insertMany([
   { name: "Alicia", age: 30 },
   { name: "Brooke", age: 25 },
   { name: "Celina", age: 35 }
])
```

This command inserts three documents into the `users` collection in a single operation. If any document in the array fails insertion due to validation or other reasons, none of the documents will be inserted unless you set the `ordered` option to `false`.

```js
db.users.insertMany(
   [
     { name: "Dave", age: 40 },
     { name: "Eve", age: "invalidAge" }
   ],
   { ordered: false }
)
```

In this case, MongoDB will insert valid documents, ignoring those that cause errors.

##### Insert Validation
MongoDB allows you to set schema validation rules, which ensure that inserted documents conform to specified constraints. For example, you may want to ensure that an `age` field is always a number:

```js
db.createCollection("users", {
   validator: {
      $jsSchema: {
         bsonType: "object",
         required: ["name", "age"],
         properties: {
            name: {
               bsonType: "string"
            },
            age: {
               bsonType: "int",
               minimum: 0
            }
         }
      }
   }
})
```

When inserting a document, MongoDB will validate it based on the schema, and invalid documents will be rejected:

```js
db.users.insertOne({ name: "Frank", age: "thirty" })  // This will fail due to schema validation.
```

##### `insert`
The `insert` method is an older version of MongoDB’s insert functionality and can insert either a single document or an array of documents. It has mostly been superseded by the more robust `insertOne` and `insertMany` methods but still works for bulk operations. For example:

```js
db.users.insert([
   { name: "Grace", age: 28 },
   { name: "Henry", age: 22 }
])
```

#### Removing Documents

##### `drop`
The `drop` method removes an entire collection from the database, including all its documents and associated indexes. For example, to completely remove the `users` collection:

```js
db.users.drop()
```

This is a destructive operation, and once a collection is dropped, it cannot be recovered. It's often used when cleaning up test environments or removing unused data.

If you want to remove specific documents from a collection without dropping it entirely, use the `deleteOne` or `deleteMany` methods.

#### Updating Documents

##### Document Replacement
MongoDB allows entire documents to be replaced. When using `replaceOne`, the document identified by the filter is completely replaced with a new document, except for the `_id` field. For example, if we want to replace a document for a user:

```js
db.users.replaceOne(
   { name: "Alice" },  // Filter criteria
   { name: "Alicia", age: 31 }  // New document
)
```

In this case, the existing document where the `name` is "Alice" will be replaced by `{ name: "Alicia", age: 31 }`. Note that if the original document contained any additional fields (e.g., `address`), they would be lost in the replacement unless explicitly added in the new document.

##### Using Update Operators
Instead of replacing entire documents, MongoDB allows updating specific fields using update operators. Some of the common update operators include:

- **$set**: Updates specific fields, adding them if they don’t exist.
- **$inc**: Increments the value of a field by a specified amount.
- **$push**: Adds an element to an array.
- **$unset**: Removes a field from the document.

Examples of using these operators:

- **$set**: Update or add the `age` field in a user document:

   ```js
   db.users.updateOne(
      { name: "Bob" },
      { $set: { age: 26 } }
   )
   ```

- **$inc**: Increment the `age` field:

   ```js
   db.users.updateOne(
      { name: "Charlie" },
      { $inc: { age: 1 } }
   )
   ```

- **$push**: Add a hobby to a user's hobbies array:

   ```js
   db.users.updateOne(
      { name: "Alicia" },
      { $push: { hobbies: "cooking" } }
   )
   ```

- **$unset**: Remove the `age` field from a user document:

   ```js
   db.users.updateOne(
      { name: "Henry" },
      { $unset: { age: "" } }
   )
   ```

##### Upserts
An **upsert** is a combination of update and insert. If no document matches the query, MongoDB inserts a new document instead of updating an existing one. This is useful when you want to ensure that a document exists, regardless of whether it was already in the collection.

To perform an upsert, set the `upsert` option to `true`:

```js
db.users.updateOne(
   { name: "Isabelle" },  // Search for this document
   { $set: { age: 30 } },  // Update if found
   { upsert: true }  // Insert if not found
)
```

If there is no document with the name "Isabelle", MongoDB will insert one with the name "Isabelle" and age `30`.

##### Updating Multiple Documents
The `updateMany` method is used to update multiple documents that match a filter. For example, suppose you want to increment the age of all users who are older than 30:

```js
db.users.updateMany(
   { age: { $gt: 30 } },
   { $inc: { age: 1 } }
)
```

This operation will increment the `age` of all users whose age is greater than 30. 

##### Returning Updated Documents
When updating documents, you may want to see the updated document immediately. In MongoDB, you can return the updated document using the `findOneAndUpdate` method, which returns the updated document in the result.

For example, to update and return the document:

```js
db.users.findOneAndUpdate(
   { name: "Alicia" },  // Filter
   { $set: { age: 32 } },  // Update operation
   { returnDocument: "after" }  // Return the updated document
)
```

This will return the document after it has been updated. You can also set `returnDocument: "before"` to see the document before it was updated. This is particularly useful when you want to verify the exact changes made to the document.

---

These operations provide a powerful toolkit for creating, updating, and managing documents in MongoDB. The combination of flexible schema handling, diverse update operators, and the ability to handle bulk operations makes MongoDB a highly versatile database for managing large datasets.