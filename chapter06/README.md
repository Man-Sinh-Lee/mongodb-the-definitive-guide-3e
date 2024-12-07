### Chapter 6: Special Index and Collection Types

#### Geospatial Indexes

MongoDB supports geospatial data, enabling efficient storage and querying of location-based data. Geospatial indexes allow you to query documents that represent points on a map, such as locations with latitude and longitude coordinates.

##### Types of Geospatial Queries
Geospatial queries can be used to find documents within a certain area or proximity to a given point. The main types of geospatial queries include:

- **$near**: Finds documents near a specific point.
- **$geoWithin**: Finds documents within a specific area or shape, such as a polygon.
- **$geoIntersects**: Finds documents where the geometry of a shape intersects with a specified geometry.

For example, to find locations near a specific coordinate:

```js
db.places.find({
  location: {
    $near: {
      $geometry: { type: "Point", coordinates: [ -73.9667, 40.78 ] },
      $maxDistance: 1000  // within 1000 meters
    }
  }
})
```

##### Using Geospatial Indexes
Before performing geospatial queries, you need to create a **2dsphere** index to support geospatial data. This index supports data stored as GeoJSON objects and allows queries that handle spherical geometry (i.e., the curvature of the Earth).

```js
db.places.createIndex({ location: "2dsphere" })
```

This index allows efficient querying of documents where the `location` field contains geospatial data.

##### Compound Geospatial Indexes
You can combine geospatial indexes with other types of indexes to create compound indexes. For instance, you might want to query based on both location and another attribute like `category`. To do this, create a compound index:

```js
db.places.createIndex({ location: "2dsphere", category: 1 })
```

This index supports queries that filter by both geospatial location and other fields like `category`.

##### 2d Indexes
The **2d** index supports legacy geospatial data represented as two-dimensional coordinates. It is limited to flat, Cartesian plane geometries, and is mostly used in older MongoDB applications. An example of creating a **2d** index:

```js
db.places.createIndex({ location: "2d" })
```

This index supports basic geospatial queries such as finding points near a specific coordinate but doesn’t handle spherical geometry.

#### Indexes for Full-Text Search

MongoDB provides support for **full-text search**, which allows you to search for text within documents. Full-text search is useful for applications where users need to search for words or phrases in text fields.

##### Creating a Text Index
To enable full-text search, you need to create a **text index** on the fields that contain text. For example, if you want to perform text searches on the `description` field:

```js
db.articles.createIndex({ description: "text" })
```

A text index can be created on multiple fields as well:

```js
db.articles.createIndex({ title: "text", description: "text" })
```

##### Text Search
Once a text index is created, you can use the `$text` operator to search for words or phrases in the indexed fields. For example, to search for the word "mongodb":

```js
db.articles.find({ $text: { $search: "mongodb" } })
```

You can also search for phrases and use operators like `-` to exclude words or `"` for exact matches:

```js
db.articles.find({ $text: { $search: "\"full-text search\"" } })
```

##### Optimizing Full-Text Search
To optimize text search, MongoDB allows you to assign weights to fields in the text index. Fields with higher weights are prioritized in the search results. For example, if the `title` field is more important than the `description`, you can assign different weights:

```js
db.articles.createIndex(
  { title: "text", description: "text" },
  { weights: { title: 5, description: 1 } }
)
```

##### Searching in Other Languages
By default, MongoDB uses English for text search, but you can specify other languages when creating the index. For example, to create a text index that supports French:

```js
db.articles.createIndex({ description: "text" }, { default_language: "french" })
```

You can also override the language at the document level by specifying a `language_override` field in the document.

#### Capped Collections

Capped collections are fixed-size collections that automatically overwrite the oldest data when the size limit is reached. They are ideal for use cases like logging, where you want to retain only the most recent data.

##### Creating Capped Collections
Capped collections are created with a specified size limit (in bytes) and an optional maximum number of documents. For example, to create a capped collection with a size of 1MB:

```js
db.createCollection("logs", { capped: true, size: 1048576 })
```

You can also specify a document limit:

```js
db.createCollection("logs", { capped: true, size: 1048576, max: 1000 })
```

##### Tailable Cursors
Tailable cursors are a special type of cursor used with capped collections. They allow you to continuously retrieve new documents as they are added to the collection, similar to reading from a log file in real-time. This is useful for applications like monitoring or messaging queues.

```js
db.logs.find().tailable()
```

Tailable cursors remain open and keep returning results as new data is written to the collection.

#### Time-To-Live Indexes (TTL)

TTL indexes are used to automatically remove documents from a collection after a specified amount of time. This is useful for data that expires, such as session data, temporary files, or logs.

To create a TTL index, you use the `expireAfterSeconds` option. For example, to create a TTL index that removes documents 30 days after their `createdAt` field:

```js
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 2592000 })  // 30 days
```

TTL indexes work by scheduling a background process that checks for and removes expired documents. However, the removal process is not immediate—it runs periodically.

#### Storing Files with GridFS

When storing files larger than 16MB, MongoDB’s document size limit, you can use **GridFS** to divide the files into smaller chunks and store them in multiple documents. GridFS is useful for storing and retrieving large binary files like images, audio files, or videos.

##### Getting Started with GridFS: mongofiles
MongoDB provides a command-line utility called `mongofiles` to interact with GridFS. For example, to upload a file using `mongofiles`:

```js
mongofiles --db mydb put example.txt
```

This command stores `example.txt` in GridFS by splitting it into chunks and storing each chunk as a separate document.

You can also retrieve the file:

```js
mongofiles --db mydb get example.txt
```

##### Working with GridFS from the MongoDB Drivers
GridFS is fully integrated into MongoDB drivers, allowing you to upload, download, and manage files from your application. For example, in a Node.js application:

```js
const GridFSBucket = require('mongodb').GridFSBucket;
const bucket = new GridFSBucket(db);

bucket.uploadFromStream('example.txt', fs.createReadStream('/path/to/file'));
```

To download a file:

```js
bucket.openDownloadStreamByName('example.txt').pipe(fs.createWriteStream('/path/to/output'));
```

GridFS allows storing file metadata as well, making it easy to track additional information about the files.

##### Under the Hood
Underneath, GridFS stores files in two collections: `fs.files` and `fs.chunks`. The `fs.files` collection contains metadata about each file, including the filename, upload date, and the number of chunks. The `fs.chunks` collection stores the actual data chunks of the file. Each chunk has a size of 255KB by default, but this can be adjusted if needed.

GridFS automatically handles splitting and reassembling the file, allowing you to retrieve large files efficiently. It’s a useful tool for working with files that exceed MongoDB’s document size limit.

---

MongoDB’s special index and collection types provide powerful capabilities for handling geospatial data, full-text search, capped collections, TTL-based expiration, and large file storage. By leveraging these features, you can optimize your database for a wide range of application requirements, from real-time location-based services to efficient file storage.