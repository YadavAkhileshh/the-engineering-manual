# 08 MongoDB and Mongoose

This guide covers MongoDB internals, CRUD operations, indexing, aggregation pipeline, transactions, schema patterns, and Mongoose operations.

## MongoDB Architecture

### Definition
MongoDB is a document-oriented, NoSQL database that stores data in flexible, JSON-like BSON (Binary JSON) documents. It uses databases, collections (instead of tables), and relies on the WiredTiger storage engine to handle memory caching, concurrency, and compression.

### Real-world Analogy
Think of organizing customer files in folders. A relational database is like a spreadsheet where everyone gets exactly one row and you must create separate sheets for details. MongoDB is like a cabinet of folders containing paper sheets (documents): one sheet can list 5 addresses inline, another can have only a name, all stored in the same folder (collection).

### Code Example
```javascript
// A sample BSON document structure
const userDoc = {
  _id: "60c72b2f9b1d8b2bad18a202", // Object ID
  name: "Alice",
  contact: {
    email: "alice@test.com",
    phones: ["123-456", "789-012"] // Array support
  }
};
```

### Common Interview Questions
- What is the difference between JSON and BSON?
- Explain the role of the WiredTiger storage engine.
- How does MongoDB handle concurrency control (document-level locking)?

### Reference Links
- [MongoDB: Document Database](https://www.mongodb.com/docs/manual/core/document/)
- [MongoDB: WiredTiger Storage Engine](https://www.mongodb.com/docs/manual/core/wiredtiger/)

## MongoDB vs Relational Databases

### Definition
MongoDB is a non-relational, schema-less database designed for high horizontal scalability and flexible schema evolution. Relational databases (PostgreSQL) enforce strict schemas, relationships, and foreign key constraints, excelling at complex multi-table joins.

### Real-world Analogy
Relational is like building a block skyscraper: you must follow strict architectural rules, and every floor must align exactly. NoSQL MongoDB is like building a modular house: you can add a sunroom or a balcony to one room without needing to rebuild the foundations of the rest of the rooms.

### Code Example
```javascript
// MongoDB embed address (No joins needed)
// { name: "Bob", addresses: [{ city: "NY" }, { city: "SF" }] }

// Relational SQL counterpart requires tables:
// SELECT * FROM users LEFT JOIN addresses ON users.id = addresses.user_id;
```

### Common Interview Questions
- In what scenarios is a relational database preferred over MongoDB?
- How do the scaling strategies (vertical scale-up vs horizontal sharding) compare?
- What are the trade-offs of using schema-less collections?

### Reference Links
- [MongoDB: SQL to MongoDB Mapping](https://www.mongodb.com/docs/manual/reference/sql-comparison/)

## CRUD Operations

### Definition
CRUD operations are the foundation of data interaction. MongoDB provides query commands (insertOne, insertMany, find, updateOne, updateMany, findOneAndUpdate, deleteOne, deleteMany, findOneAndDelete) to create, read, update, and delete documents.

### Real-world Analogy
Think of writing entries in a ledger book. insertOne is writing a new line. find is reading the ledger using a magnifying glass to filter names. updateOne is crossing out an address and writing a new one next to it. deleteOne is erasing a line from the sheet.

### Code Example
```javascript
// Mongoose / MongoDB CRUD examples
async function performCrud(db) {
  // Create
  await db.collection("users").insertOne({ name: "Bob", active: true });

  // Read with Query Operators
  const activeUsers = await db.collection("users").find({ active: true }).toArray();

  // Update
  await db.collection("users").updateOne(
    { name: "Bob" },
    { $set: { active: false } }
  );

  // Delete
  await db.collection("users").deleteOne({ name: "Bob" });
}
```

### Common Interview Questions
- Compare updateOne and findOneAndUpdate in terms of return values.
- How do you perform upserts during document updates?
- What is the difference between $set and replacing a whole document?

### Reference Links
- [MongoDB: CRUD Operations](https://www.mongodb.com/docs/manual/crud/)

## Query Operators

### Definition
Query operators filter database queries. They include Comparison ($gt, $lt, $in), Logical ($or, $and, $not), Element ($exists, $type), Evaluation ($regex, $text), and Array operators ($all, $elemMatch).

### Real-world Analogy
Imagine filtering resumes. $gt is finding candidates with more than 5 years of experience. $in is checking if their skills include "React" or "Vue". $elemMatch is verifying if they worked at a company where their role was "Lead" AND their duration was over 2 years.

### Code Example
```javascript
// Query utilizing complex operators
// Find users aged between 25 and 35 who speak both English and Spanish
db.collection("users").find({
  age: { $gte: 25, $lte: 35 },
  languages: { $all: ["English", "Spanish"] }
});
```

### Common Interview Questions
- Why do you need $elemMatch when querying arrays of subdocuments?
- What is the difference between the $or operator and using an $in array check?
- How does the $exists operator identify missing properties?

### Reference Links
- [MongoDB: Query Operators](https://www.mongodb.com/docs/manual/reference/operator/query/)

## Projection

### Definition
Projection allows you to specify which fields to return or exclude in a query result, reducing network payload size and protecting sensitive database values.

### Real-world Analogy
Imagine printing a student report card. Instead of printing their entire file including medical logs, home address, and disciplinary notes, you project only the student name and their math grade.

### Code Example
```javascript
// Return only name and email, exclude _id
db.collection("users").find(
  { active: true },
  { projection: { name: 1, email: 1, _id: 0 } }
);
```

### Common Interview Questions
- Can you mix inclusion (1) and exclusion (0) rules in a single projection configuration? What is the exception?
- How does projection differ from database-level field permissions?
- What are the performance benefits of projection?

### Reference Links
- [MongoDB: Project Fields](https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/)

## Cursor Methods

### Definition
Cursor methods alter how queries are evaluated and returned. Methods include sort (reorders results), limit (caps output count), skip (skips index offset), count (returns total length), and lean (returns plain JS objects instead of heavy Mongoose documents).

### Real-world Analogy
Think of sorting library index cards. You sort them alphabetically (sort), skip the first 50 cards (skip), count the next 10 cards (limit), and count how many cards match your search term in total (count).

### Code Example
```javascript
// Pagination cursor chain
db.collection("users")
  .find({ active: true })
  .sort({ createdAt: -1 })
  .skip(10)
  .limit(5);
```

### Common Interview Questions
- Why should you always pair skip() with sort() in pagination queries?
- What does Mongoose's .lean() method do and how does it optimize server CPU/memory usage?
- Explain why countDocuments() is preferred over estimatedDocumentCount() for filtered queries.

### Reference Links
- [MongoDB: Iterate a Cursor](https://www.mongodb.com/docs/manual/tutorial/iterate-a-cursor/)
- [Mongoose: Lean Queries](https://mongoosejs.com/docs/tutorials/lean.html)

## Indexing

### Definition
Indexing creates structured B-tree data files to speed up document lookups. Types include single field, compound, text, partial (indexing filtered rows), and TTL (Time-To-Live, auto-deleting records after a set time).

### Real-world Analogy
Think of the index at the back of a cookbook. Instead of flipping page-by-page through 500 recipes to find "Lasagna" (full collection scan), you turn to the index under "L" and find "Lasagna - Page 142" (index seek).

### Code Example
```javascript
// Compound index with sort order strategy
// Indexing lastName ascending and age descending
db.collection("users").createIndex({ lastName: 1, age: -1 });

// TTL Index to expire documents after 1 hour (3600 seconds)
db.collection("sessions").createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

### Common Interview Questions
- How does compound index order matter when querying and sorting?
- What is a covered query and why is it extremely fast?
- How do you use explain("executionStats") to check if a query uses an index?

### Reference Links
- [MongoDB: Indexing](https://www.mongodb.com/docs/manual/indexes/)
- [MongoDB: Explain Results](https://www.mongodb.com/docs/manual/reference/explain-results/)

## Aggregation Pipeline

### Definition
The Aggregation Pipeline is a framework for data aggregation modeled on the concept of data processing pipelines. Documents enter a multi-stage pipeline that transforms them into aggregated results (stages: $match, $group, $project, $unwind, $lookup).

### Real-world Analogy
Imagine a toy factory assembly line. Raw wood blocks enter. First worker throws away damaged blocks ($match). Second worker groups blocks by color ($group). Third worker paints numbers on the blocks ($project). Fourth worker splits stacked blocks into single pieces ($unwind).

### Code Example
```javascript
// Aggregate order totals grouped by customer
db.collection("orders").aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", totalSpent: { $sum: "$amount" } } },
  { $sort: { totalSpent: -1 } }
]);
```

### Common Interview Questions
- What does the $unwind stage do to array fields?
- Explain how the $lookup stage performs left outer joins.
- How do you optimize aggregation pipelines (e.g. stage order rules)?

### Reference Links
- [MongoDB: Aggregation](https://www.mongodb.com/docs/manual/aggregation/)
- [MongoDB: Aggregation Pipeline Limits](https://www.mongodb.com/docs/manual/core/aggregation-pipeline-limits/)

## Transactions

### Definition
Transactions allow you to execute multiple operations across collections in a secure, ACID-compliant sequence. Transactions require a MongoDB replica set deployment and run inside a Session context.

### Real-world Analogy
Think of transferring money at a bank. The teller must subtract $100 from your account AND add $100 to your friend's account. If the power cuts out midway, the transaction aborts and both accounts are reset to their starting balances.

### Code Example
```javascript
async function transferFunds(client, fromId, toId, amount) {
  const session = client.startSession();
  try {
    session.startTransaction();
    
    await db.collection("accounts").updateOne({ _id: fromId }, { $inc: { balance: -amount } }, { session });
    await db.collection("accounts").updateOne({ _id: toId }, { $inc: { balance: amount } }, { session });
    
    await session.commitTransaction();
  } catch (error) {
    await session.abortTransaction();
    throw error;
  } finally {
    session.endSession();
  }
}
```

### Common Interview Questions
- What are the ACID properties in database transactions?
- Why do MongoDB transactions require a replica set setup?
- How does MongoDB handle write conflicts during concurrent transactions?

### Reference Links
- [MongoDB: Transactions](https://www.mongodb.com/docs/manual/core/transactions/)

## Schema Design Patterns

### Definition
Schema design patterns determine how data relationships are structured. Patterns include embedding (storing subdocuments inline), referencing (storing ObjectIDs), and specific models like the Subset, Attribute, Bucket, or Tree patterns.

### Real-world Analogy
Embedding is keeping your receipts inside the wallet: they are small and you need them on hand. Referencing is keeping a deed in the safety vault and holding only the registration number: the document is too big or modified independently.

### Code Example
```javascript
// Embedded Pattern (High performance, low size)
const postEmbed = {
  title: "My Post",
  comments: [
    { author: "Alice", text: "Nice!" },
    { author: "Bob", text: "Great post." }
  ]
};

// Referenced Pattern (Scales infinitely)
const postRef = {
  title: "My Post",
  comments: ["commentObjectId1", "commentObjectId2"]
};
```

### Common Interview Questions
- Explain when you should embed data versus referencing it.
- Describe the Outlier/Subset pattern and how it handles viral documents (like a post with millions of comments).
- What is the Bucket pattern and when is it used for time-series data?

### Reference Links
- [MongoDB: Data Modeling Introduction](https://www.mongodb.com/docs/manual/core/data-modeling-introduction/)
- [MongoDB: Schema Design Patterns](https://www.mongodb.com/blog/post/building-with-patterns-a-summary)

## Mongoose Deep Dive

### Definition
Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js. It manages database schemas, types, built-in and custom validation rules, virtual fields, custom methods, statics, middleware hooks, and plugin attachments.

### Real-world Analogy
Mongoose is like hiring a strict compliance officer for your file room. MongoDB doesn't care what you write in files, but the compliance officer checks every folder tab against the corporate handbook before saving it to the shelves.

### Code Example
```javascript
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 18 }
});

// Virtual Property (not saved in database)
userSchema.virtual("isAdult").get(function() {
  return this.age >= 18;
});

// Pre-save middleware hook (e.g. password hashing)
userSchema.pre("save", async function(next) {
  if (this.isModified("password")) {
    this.password = await hash(this.password);
  }
  next();
});

const User = mongoose.model("User", userSchema);
```

### Common Interview Questions
- What is the difference between a method and a static in Mongoose?
- How do Mongoose pre and post hooks work?
- Why do unique constraints in schemas require a MongoDB index build to work?

### Reference Links
- [Mongoose Docs: Guide](https://mongoosejs.com/docs/guide.html)
- [Mongoose Docs: Middleware](https://mongoosejs.com/docs/middleware.html)

## Mongoose Populate

### Definition
Populate is a Mongoose process that automatically replaces specified paths (containing reference IDs) in a document with the actual documents fetched from another collection.

### Real-world Analogy
Imagine reading a catalog with reference codes (e.g., "See item #A94"). Populate is like having an assistant who runs to the warehouse, fetches item #A94, and places it on your desk right where the reference code was printed.

### Code Example
```javascript
// Schema reference configuration
// commentSchema: { post: { type: Schema.Types.ObjectId, ref: 'Post' } }

// Querying with Populate
const comment = await Comment.findOne({ _id: commentId })
  .populate("post", "title author") // Populate only title and author fields
  .lean();
```

### Common Interview Questions
- Why does using .populate() degrade query performance compared to SQL JOINS?
- How do you implement nested populate queries?
- What are the memory implications of running large populate queries without select constraints?

### Reference Links
- [Mongoose Docs: Populate](https://mongoosejs.com/docs/populate.html)

## Connection Management

### Definition
Connection management handles communication between Mongoose and MongoDB. It uses connection pooling (maintaining reusable socket connections), manages pool sizing, and processes connectivity events (connected, disconnected, error).

### Real-world Analogy
Imagine a busy corporate hotline. Instead of dialling the phone, waiting for a ring, and hanging up after every single sentence (expensive handshake), the company rents 10 dedicated hotlines (connection pool) that stay connected permanently.

### Code Example
```javascript
const mongoose = require("mongoose");

mongoose.connect(process.env.MONGODB_URI, {
  maxPoolSize: 10, // Max sockets in pool
  minPoolSize: 2,
  serverSelectionTimeoutMS: 5000 // Timeout warning
});

mongoose.connection.on("connected", () => console.log("DB connected"));
mongoose.connection.on("error", (err) => console.error("DB error: " + err));
```

### Common Interview Questions
- What is connection pooling and why is it critical for high-throughput applications?
- How do you handle multiple database connections in a single Mongoose application?
- What does the serverSelectionTimeoutMS configuration do?

### Reference Links
- [Mongoose Docs: Connections](https://mongoosejs.com/docs/connections.html)

## MongoDB Atlas

### Definition
MongoDB Atlas is a fully managed cloud database service. It automates cluster setup, connection whitelisting, database user access, scaling, metrics logging, backups, and integrates Atlas Search (Lucene search engine).

### Real-world Analogy
Instead of buying a plot of land, digging a well, and building your own water purification facility (hosting MongoDB on a virtual machine), you subscribe to a utility company that delivers clean water right to your house.

### Code Example
```bash
# Connection string format for Atlas Serverless/Replica Sets
mongodb+srv://<username>:<password>@cluster0.abcde.mongodb.net/dbname?retryWrites=true&w=majority
```

### Common Interview Questions
- What does the mongodb+srv protocol specify in connection strings?
- How does the Atlas network whitelist protect database security?
- Compare database-level backups and point-in-time recovery.

### Reference Links
- [MongoDB Atlas Docs](https://www.mongodb.com/docs/atlas/)

## Scenario-based Interview Questions for MongoDB

### Scenario 1
You are building a social media feed and query posts using populate('author') and populate('comments'). Performance degrades when the posts exceed 1,000 documents. Database CPU utilization spikes to 95%. How do you diagnose and resolve this?

*Expected Approach:*
1. State that .populate() is executed client-side by Mongoose: it triggers separate find queries behind the scenes, leading to an N+1 query pattern.
2. Optimize by using indexing on author and comment parent IDs.
3. Switch from Mongoose populate to MongoDB's native Aggregation Pipeline using the $lookup stage.
4. Add .lean() to queries to bypass Mongoose document instantiation.

### Scenario 2
An online ticket booking system needs to reserve seats. If two users click book at the same second, both get reservations for the same seat, causing double-bookings. How do you prevent this in MongoDB?

*Expected Approach:*
1. Explain that this is a concurrency race condition.
2. Resolve this by using MongoDB Transactions: start a session, read the seat status, verify it is vacant, update the status to reserved, and commit.
3. Alternatively, implement optimistic concurrency control using version key matches or unique compound indexes (e.g. unique index on { flightId: 1, seatNumber: 1 }), which causes the second insert/update to fail automatically.
