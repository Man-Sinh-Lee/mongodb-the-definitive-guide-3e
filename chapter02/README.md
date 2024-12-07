### Chapter 2: Getting Started

#### Documents
At the core of MongoDB is the **document**, which is a flexible and hierarchical structure that stores data as key-value pairs. These documents are similar to rows in relational databases but are far more flexible. MongoDB documents can contain different data types and even embed other documents and arrays. For example:

```js
{
   "name": "Alice",
   "age": 30,
   "address": {
       "street": "123 Main St",
       "city": "New York"
   },
   "hobbies": ["reading", "traveling"]
}
```

This document shows MongoDB’s flexibility, where fields can store not only simple data types like strings or integers, but also complex types like embedded documents (`address`) and arrays (`hobbies`).

#### Collections

##### Dynamic Schemas
MongoDB collections are schema-less, meaning that documents within a collection are not required to follow a fixed schema. This allows documents with varying structures to exist in the same collection. For example:

```js
{
   "name": "Alice",
   "age": 30
}
```

can coexist with:

```js
{
   "name": "Bob",
   "city": "Chicago",
   "phone": "123-456-7890"
}
```

This flexibility supports rapid iteration during development but requires careful planning to ensure consistency.

##### Naming
Collections are identified by their name, which can be any UTF-8 string, with some restrictions. Names cannot contain the null character (`\0`), and the use of `system.` as a prefix is reserved for internal collections. Subcollections can be defined using a dot (`.`) notation, such as `blog.posts`, allowing for organized namespacing. However, there’s no special relationship between these subcollections at the database level.

#### Databases
MongoDB groups collections into databases, and a single MongoDB instance can host multiple databases. Each database is identified by a name, which follows specific rules (e.g., no spaces or special characters). Databases in MongoDB include reserved ones like:

- `admin` for administrative tasks,
- `local` for instance-specific data, and
- `config` for sharded cluster metadata.

MongoDB’s flexible design allows for each application to use its own database while coexisting on the same MongoDB instance.

#### Getting and Starting MongoDB
MongoDB can be started by running the `mongod` process. By default, it uses the `/data/db/` directory for storage and listens on port 27017. For example, starting MongoDB from the command line looks like this:

```js
$ mongod
```

If the directory does not exist or permissions are incorrect, MongoDB will fail to start. Once started, MongoDB listens for connections, and multiple instances can be managed via different ports or storage directories. On Windows, the process is initiated with `mongod.exe`.

#### Introduction to the MongoDB Shell

##### Running the Shell
MongoDB includes an interactive JavaScript shell (`mongo`) for working with the database. The shell connects to a MongoDB instance and allows the execution of JavaScript commands. It can be started with:

```js
$ mongosh
```

The shell connects to the default database (`test`), but you can switch to another database using the `use` command:

```js
use myDatabase
```

##### A MongoDB Client
The `mongosh` shell acts as a MongoDB client, providing the `db` object to interact with the connected database. For instance, to see the current database:

```js
db
myDatabase
```

You can interact with collections in the current database using JavaScript-like syntax:

```js
db.users.find()
```

##### Basic Operations with the Shell
CRUD (Create, Read, Update, Delete) operations are easy in MongoDB. For example:

- **Create (Insert)**:
  ```js
  db.users.insertOne({name: "Alice", age: 30})
  ```
- **Read (Query)**:
  ```js
  db.users.find({name: "Alice"}).pretty()
  ```
- **Update**:
  ```js
  db.users.updateOne({name: "Alice"}, {$set: {age: 31}})
  ```
- **Delete**:
  ```js
  db.users.deleteOne({name: "Alice"})
  ```

These operations allow users to interact with documents, perform complex queries, and manipulate the data in real-time.

#### Data Types

##### Basic Data Types
MongoDB supports a wide range of data types, including:

- **Null**: `{ "key": null }`
- **Boolean**: `{ "key": true }`
- **Number**: `{ "key": 123 }`
- **String**: `{ "key": "hello" }`
- **Date**: `{ "key": new Date() }`
- **Array**: `{ "key": [1, 2, 3] }`
- **Embedded Document**: `{ "key": { "subkey": "value" } }`
- **ObjectId**: MongoDB uses ObjectIds as unique identifiers for documents.

##### Dates
Dates are stored as 64-bit integers representing milliseconds since the Unix epoch. For example:

```js
db.events.insertOne({event: "meeting", date: new Date()})
```

The `new Date()` function generates the current date and time in JavaScript format.

##### Arrays
MongoDB allows arrays to be stored in documents, like this:

```js
db.users.insertOne({name: "Alice", hobbies: ["reading", "hiking"]})
```

Arrays can store multiple types of data, such as strings, numbers, or even other documents.

##### Embedded Documents
Embedded documents allow for hierarchical relationships. For example, embedding an address inside a user document:

```js
db.users.insertOne({name: "Alice", address: {city: "New York", zip: "10001"}})
```

##### _id and ObjectIds
Every MongoDB document has a unique `_id` field. If not provided by the user, MongoDB generates an ObjectId by default. ObjectIds are 12-byte unique identifiers created based on the timestamp, machine identifier, process ID, and a counter. For example:

```js
ObjectId("5f16d6789e7e3e243457dcac")
```

This structure ensures global uniqueness across all MongoDB instances.

#### Using the MongoDB Shell

##### Tips for Using the Shell
MongoDB’s shell offers many built-in helpers like `db.help()` or `db.collection.help()` for command guidance. You can also use JavaScript functions and syntax, such as loops, conditionals, and variable declarations, inside the shell. It’s helpful for debugging and rapid testing.

##### Running Scripts with the Shell
You can run JavaScript files in the MongoDB shell by passing them as arguments:

```js
$ mongo myScript.js
```

Inside the script, the `db` object allows interacting with the database. This can be useful for automating tasks or running batch commands.

##### Creating a `.mongorc.js`
The `.mongorc.js` file is a shell startup script. By placing commonly used commands or customizations in this file, they will run automatically whenever the shell is started.

##### Customizing Your Prompt
MongoDB allows users to customize the shell prompt. For example, to show the current database name in the prompt:

```js
prompt = function() { return db + ""; }
```

This gives a dynamic prompt that changes based on the active database.

##### Editing Complex Variables
To edit complex variables or collections, MongoDB shell provides commands like `edit`:

```js
var myVar = {name: "Alice", age: 30}
edit myVar
```

This command opens the variable in a text editor for easier modification of large or complex structures.

##### Inconvenient Collection Names
If a collection name contains special characters (e.g., `$` or `.`), direct access might not work using the dot notation. Instead, you can use `db.getCollection`:

```js
db.getCollection("weird$name").find()
```

This allows interaction with collections that have unconventional names.