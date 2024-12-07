### Chapter 7: Introduction to the Aggregation Framework

MongoDB's **aggregation framework** allows for complex data processing and transformation operations directly within the database. It is similar to SQL's `GROUP BY`, but much more powerful and flexible. Aggregations are performed through a **pipeline** of stages, where each stage transforms the data before passing it to the next stage.

#### Pipelines, Stages, and Tunables

An **aggregation pipeline** consists of multiple stages that process documents in a sequence. Each stage in the pipeline performs a specific operation on the data (such as filtering, transforming, or grouping) and passes the results to the next stage. MongoDB’s aggregation framework is highly efficient, as operations are handled server-side, reducing the need for extensive data processing in application code.

For example, a simple pipeline might look like this:

```js
db.orders.aggregate([
  { $match: { status: "shipped" } },  // Stage 1: Filter documents
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },  // Stage 2: Group by customerId and calculate total
  { $sort: { total: -1 } }  // Stage 3: Sort by total amount in descending order
])
```

In this example, the pipeline filters orders that are `shipped`, groups them by customer ID, calculates the total order amount for each customer, and sorts the results.

- **Tunables**: MongoDB allows you to fine-tune the performance of aggregation pipelines using memory limits and cursor settings. You can also enable disk-based storage for large aggregation operations using the `allowDiskUse` option.

#### Getting Started with Stages: Familiar Operations

The stages in an aggregation pipeline correspond to specific operations, such as filtering (`$match`), sorting (`$sort`), and projecting fields (`$project`). Here are some common stages:

- **$match**: Filters documents that match specified criteria, similar to the `find()` query. For example, to filter orders by status:

  ```js
  { $match: { status: "shipped" } }
  ```

- **$group**: Groups documents by a specified field and performs operations like summing, averaging, or counting. For example, to group by customer ID:

  ```js
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
  ```

- **$sort**: Sorts the documents based on a specific field. For example, to sort by the total amount in descending order:

  ```js
  { $sort: { total: -1 } }
  ```

#### Expressions

Expressions in the aggregation framework are used to compute values, manipulate fields, or perform calculations. They are often used in stages like `$project` and `$group`. MongoDB provides a wide range of expressions, such as:

- **Arithmetic expressions**: For example, `$add`, `$subtract`, `$multiply`, and `$divide` are used for performing arithmetic operations.

  ```js
  { $project: { totalCost: { $multiply: ["$quantity", "$price"] } } }
  ```

- **String expressions**: Functions like `$concat`, `$toUpper`, and `$substr` are used to manipulate string fields.

  ```js
  { $project: { upperCaseName: { $toUpper: "$name" } } }
  ```

#### $project

The **$project** stage is used to specify which fields to include, exclude, or modify in the output documents. It reshapes documents by adding, removing, or transforming fields.

For example, to include only the `name` and `total` fields and add a new field called `discount`:

```js
db.sales.aggregate([
  { $project: { name: 1, total: 1, discount: { $multiply: ["$total", 0.1] } } }
])
```

In this example, a new field `discount` is calculated as 10% of the `total`.

#### $unwind

The **$unwind** stage deconstructs an array field from each document into separate documents, one for each element in the array. This is useful for processing arrays where each element needs to be treated individually.

For example, given the following document:

```js
{ "_id": 1, "name": "John", "hobbies": ["reading", "hiking", "swimming"] }
```

Using `$unwind` on the `hobbies` array:

```js
db.users.aggregate([
  { $unwind: "$hobbies" }
])
```

This will produce three documents, each with a different hobby:

```js
{ "_id": 1, "name": "John", "hobbies": "reading" }
{ "_id": 1, "name": "John", "hobbies": "hiking" }
{ "_id": 1, "name": "John", "hobbies": "swimming" }
```

#### Array Expressions

Array expressions allow you to manipulate arrays in aggregation pipelines. MongoDB provides several operators to work with arrays:

- **$size**: Returns the size of an array.
- **$slice**: Returns a subset of an array.
- **$arrayElemAt**: Returns the element at a specified index.

For example, to get the size of the `hobbies` array:

```js
db.users.aggregate([
  { $project: { name: 1, numberOfHobbies: { $size: "$hobbies" } } }
])
```

#### Accumulators: Using Accumulators in Project Stages

**Accumulators** are used in aggregation to compute values over groups of documents, like calculating totals or averages. Some common accumulators are:

- **$sum**: Adds values together.
- **$avg**: Calculates the average value.
- **$min** / **$max**: Returns the minimum or maximum value.
- **$push**: Collects values into an array.

For example, to calculate the total amount for each customer:

```js
db.orders.aggregate([
  { $group: { _id: "$customerId", totalAmount: { $sum: "$amount" } } }
])
```

You can also use accumulators in the `$project` stage. For example, to calculate an accumulated field for each document:

```js
db.sales.aggregate([
  { $project: { totalCost: { $sum: ["$price", "$tax", "$shipping"] } } }
])
```

#### Introduction to Grouping

The **$group** stage groups documents by a specific field (or fields) and allows the application of accumulator expressions to aggregate data. This is similar to SQL's `GROUP BY` clause.

##### The _id Field in Group Stages
In the `$group` stage, the `_id` field specifies how the documents should be grouped. For example, to group by `customerId`:

```js
db.orders.aggregate([
  { $group: { _id: "$customerId", totalAmount: { $sum: "$amount" } } }
])
```

The `_id` field defines the unique identifier for each group. If you want to group all documents into a single group, set `_id` to `null`.

##### Group Versus Project
The `$group` stage aggregates data and reduces the number of documents by grouping them, while the `$project` stage reshapes individual documents without reducing their number. The `$group` stage is used for operations like summing or averaging, whereas `$project` is used for field-level transformations.

For example, to group documents by `status` and calculate the total number of orders in each group:

```js
db.orders.aggregate([
  { $group: { _id: "$status", count: { $sum: 1 } } }
])
```

In contrast, `$project` would modify the structure of each document but wouldn’t change the number of documents.

#### Writing Aggregation Pipeline Results to a Collection

You can store the results of an aggregation pipeline into a new collection using the `$out` stage. This is useful when you want to persist the results of an aggregation for later use.

For example, to store the results of an aggregation into a collection called `summary`:

```js
db.orders.aggregate([
  { $group: { _id: "$customerId", totalAmount: { $sum: "$amount" } } },
  { $out: "summary" }
])
```

This will create or replace the `summary` collection with the results of the aggregation.

---

MongoDB’s aggregation framework is a powerful tool for performing complex data transformations and computations. By utilizing stages like `$match`, `$group`, `$project`, and `$unwind`, and leveraging expressions and accumulators, users can efficiently process and transform large datasets directly within the database.