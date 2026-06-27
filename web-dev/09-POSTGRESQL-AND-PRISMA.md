# 09 PostgreSQL and Prisma

This section covers PostgreSQL fundamentals, advanced SQL querying (Window Functions, CTEs, Joins, JSONB), transactions, stored procedures, and Prisma ORM integration.

## PostgreSQL Fundamentals

### Definition
PostgreSQL is an open-source, object-relational database management system. It relies on MVCC (Multi-Version Concurrency Control) for transaction isolation, uses WAL (Write-Ahead Logging) to ensure durability, and employs VACUUM to reclaim space left by dead tuples.

### Real-world Analogy
Imagine a rental car service. Instead of forcing clients to wait in line until the previous renter returns a car to check it out (locking), the company makes a copy of the car key and paperwork for each driver. The drivers go on their trips (MVCC versions) without blocking each other. The maintenance team washes returned cars during off-hours (VACUUM cleanup).

### Code Example
```sql
-- Query showing isolation level configuration
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE active = true;
COMMIT;
```

### Common Interview Questions
- How does Multi-Version Concurrency Control (MVCC) prevent read-write blocking?
- What is the role of Write-Ahead Logging (WAL) in database crash recovery?
- Why is the VACUUM process necessary in PostgreSQL and how does Autovacuum help?

### Reference Links
- [PostgreSQL Docs: MVC Architecture](https://www.postgresql.org/docs/current/mvcc.html)
- [PostgreSQL Docs: WAL Reliability](https://www.postgresql.org/docs/current/wal-reliability.html)

## Data Types

### Definition
PostgreSQL provides comprehensive native data types including numeric formats, date/time timestamps with timezones, UUIDs for unique keys, custom Enums, arrays, and JSON/JSONB (Binary JSON) formats.

### Real-world Analogy
Imagine a warehouse storage system. Primitives like integers are stored in custom envelope slots. UUIDs are stored in unique security deposit boxes. JSONB data is stored in a compressed cargo crate: you can inspect the list on the side (indexing) and find nested parts without opening the crate.

### Code Example
```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  tags TEXT[] NOT NULL, -- Array support
  metadata JSONB NOT NULL, -- Binary JSON support
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

### Common Interview Questions
- Compare JSON and JSONB regarding storage footprint and query performance.
- When is using a UUID preferred over a standard auto-incrementing integer key?
- Explain how array-type columns are queried in PostgreSQL.

### Reference Links
- [PostgreSQL Docs: Data Types](https://www.postgresql.org/docs/current/datatype.html)

## DDL (Data Definition Language)

### Definition
DDL includes SQL commands (CREATE TABLE, ALTER TABLE, DROP TABLE, TRUNCATE) used to define and modify database structures, enforce constraints (FOREIGN KEY, UNIQUE, CHECK), and manage temporary tables.

### Real-world Analogy
DDL is like designing the blueprint of an apartment building. You establish structural walls (tables), define door frame measurements (constraints), alter the blueprint to add a balcony (ALTER TABLE), or wipe out the interior walls to start over (TRUNCATE).

### Code Example
```sql
CREATE TABLE users (
  id INT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  age INT CONSTRAINT check_age CHECK (age >= 18)
);
```

### Common Interview Questions
- How does TRUNCATE differ from DELETE FROM Table?
- Explain the role of ON DELETE CASCADE in foreign key constraints.
- When should you use temporary tables, and what is their scope?

### Reference Links
- [PostgreSQL Docs: DDL Introduction](https://www.postgresql.org/docs/current/ddl.html)

## DML and DQL

### Definition
DML (Data Manipulation Language) and DQL (Data Query Language) manipulate and query row data using commands (INSERT, SELECT, UPDATE, DELETE) paired with conditional clauses (WHERE, GROUP BY, HAVING, ORDER BY).

### Real-world Analogy
Think of managing employee files. INSERT is adding a new folder to the cabinet. UPDATE is writing a promotion note inside a folder. DELETE is throwing a folder in the shredder. SELECT is searching for folders matching a specific department.

### Code Example
```sql
INSERT INTO users (email, age) VALUES ('test@test.com', 25)
ON CONFLICT (email) DO UPDATE SET age = EXCLUDED.age;
```

### Common Interview Questions
- Explain the difference between the WHERE clause and the HAVING clause.
- How does INSERT ... ON CONFLICT (UPSERT) work?
- Describe the execution order of clauses inside a SELECT query.

### Reference Links
- [PostgreSQL Docs: Queries](https://www.postgresql.org/docs/current/queries.html)

## Joins

### Definition
Joins combine columns from two or more tables based on matching field keys. Join types include INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN, CROSS JOIN (Cartesian product), and SELF JOIN.

### Real-world Analogy
Imagine comparing guest lists for two parties. INNER JOIN is finding people who attended both parties. LEFT JOIN is finding everyone who came to the first party, along with their table numbers if they also came to the second. CROSS JOIN is matching every food item to every guest plate.

### Code Example
```sql
-- Join query returning users and their order details
SELECT users.email, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id
WHERE orders.status = 'paid';
```

### Common Interview Questions
- What is the difference between LEFT JOIN and INNER JOIN?
- Explain when you would use a SELF JOIN with a practical example.
- How does the database planner select join algorithms (e.g. Nested Loop vs Hash Join)?

### Reference Links
- [PostgreSQL Docs: Joins](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-FROM)

## Subqueries

### Definition
Subqueries are nested SQL queries executed inside a parent query. They can be scalar (returns a single value), row (returns a single row), table (returns a table), correlated (references parent query columns), or paired with conditional existence operators (EXISTS, ANY, ALL).

### Real-world Analogy
Imagine a manager asking: "Bring me the file of the employee who earns the absolute highest salary". The assistant first searches the database to find the highest salary amount (subquery: $150,000), then searches again to find who holds that salary.

### Code Example
```sql
-- Find users who have at least one paid order using EXISTS
SELECT email FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o 
  WHERE o.user_id = u.id AND o.status = 'paid'
);
```

### Common Interview Questions
- How does a correlated subquery differ from a non-correlated subquery?
- Why is EXISTS preferred over IN when querying large datasets?
- Explain how the ANY and ALL operators work in subquery comparisons.

### Reference Links
- [PostgreSQL Docs: Subqueries](https://www.postgresql.org/docs/current/functions-subquery.html)

## Window Functions

### Definition
Window functions perform calculations across a set of table rows that are related to the current row, without collapsing them into a single summary row. They use the OVER clause partition rules (PARTITION BY, ORDER BY).

### Real-world Analogy
Imagine a class of runners. You want to show each runner's time, but next to it, print their rank within their age division (PARTITION BY age ORDER BY time). The individual rows are not grouped; each runner retains their own line on the screen.

### Code Example
```sql
-- Rank orders per user by amount using window functions
SELECT user_id, amount,
       ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY amount DESC) as rank
FROM orders;
```

### Common Interview Questions
- Compare ROW_NUMBER(), RANK(), and DENSE_RANK() ranking functions.
- What do the LAG() and LEAD() window functions do?
- Explain the performance implications of OVER (PARTITION BY ...).

### Reference Links
- [PostgreSQL Docs: Window Functions](https://www.postgresql.org/docs/current/tutorial-window.html)

## Common Table Expressions (CTE)

### Definition
A Common Table Expression (CTE) is a temporary result set defined within the scope of a single SELECT, INSERT, UPDATE, or DELETE query. CTEs improve readability and can be recursive to query hierarchical data.

### Real-world Analogy
Think of a scratchpad. Instead of writing a complex nested math formula, you calculate a sub-total (CTE definition), name it "Taxable Income", and then use that name in the main formula down the page.

### Code Example
```sql
-- CTE example for complex query structures
WITH user_totals AS (
  SELECT user_id, SUM(amount) as total
  FROM orders
  GROUP BY user_id
)
SELECT u.email, ut.total
FROM users u
JOIN user_totals ut ON u.id = ut.user_id
WHERE ut.total > 500;
```

### Common Interview Questions
- What is the difference between a CTE and a Subquery?
- How do you write a recursive CTE to query organizational trees or category hierarchies?
- What does the MATERIALIZED keyword do when defining a CTE?

### Reference Links
- [PostgreSQL Docs: WITH Queries](https://www.postgresql.org/docs/current/queries-with.html)

## Set Operations

### Definition
Set operations combine the results of two or more queries into a single result set. Operators include UNION (returns distinct rows), UNION ALL (includes duplicates), INTERSECT (intersection), and EXCEPT (difference).

### Real-world Analogy
Imagine combining two lists of subscribers. UNION blends both lists, deleting duplicates. UNION ALL merges the sheets directly, even if some names appear twice. EXCEPT returns customers on list A but deletes anyone who is also on list B.

### Code Example
```sql
-- Merge active customer emails and supplier emails
SELECT email FROM customers
UNION
SELECT email FROM suppliers;
```

### Common Interview Questions
- Compare UNION and UNION ALL regarding sorting and execution speed.
- What constraints must be met by queries before applying set operations?
- Explain how INTERSECT is resolved by the query engine.

### Reference Links
- [PostgreSQL Docs: Combined Queries](https://www.postgresql.org/docs/current/queries-union.html)

## Views

### Definition
A View is a virtual table representing the result of a pre-defined SQL query. Simple views can be updatable. Materialized Views physically store the queried data on disk to speed up slow operations and must be refreshed manually.

### Real-world Analogy
A standard view is a window looking into a factory yard. A Materialized View is taking a photograph of the factory yard: the photo is fast to look at, but if the yard configuration changes, you must take a new photo (REFRESH).

### Code Example
```sql
-- Create a Materialized View
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT date_trunc('month', created_at) as month, SUM(amount)
FROM orders
GROUP BY 1;

-- Refresh Materialized View
REFRESH MATERIALIZED VIEW monthly_sales;
```

### Common Interview Questions
- What is the difference between a standard View and a Materialized View?
- How do you handle write updates on updatable views?
- Explain the syntax and locking behavior of REFRESH MATERIALIZED VIEW CONCURRENTLY.

### Reference Links
- [PostgreSQL Docs: Views](https://www.postgresql.org/docs/current/rules-views.html)
- [PostgreSQL Docs: Materialized Views](https://www.postgresql.org/docs/current/rules-materializedviews.html)

## JSONB Operations

### Definition
JSONB is PostgreSQL's binary-stored JSON format. It strips whitespaces and duplicate keys, supporting key-value indexing, existence testing, path navigation, and GIN (Generalized Inverted Index) indexing.

### Real-world Analogy
JSON is a raw textbook PDF document. JSONB is a structured, searchable database where the chapters, terms, and indices are indexed: you search terms instantly without scanning every page of the text.

### Code Example
```sql
-- Query nested JSONB properties
-- Checks if metadata contains: { "active": true }
SELECT * FROM products
WHERE metadata @> '{"active": true}';

-- Access nested field value using ->> operator
SELECT metadata->'details'->>'color' FROM products;
```

### Common Interview Questions
- Compare the -> and ->> operators in JSONB queries.
- How do you index JSONB columns using GIN indexes?
- What are the containment (@>) and existence (?) operators?

### Reference Links
- [PostgreSQL Docs: JSON Types](https://www.postgresql.org/docs/current/datatype-json.html)

## Indexes

### Definition
Indexes speed up database searches. Types include B-tree (default, ranges/equality), Hash (fast equality), GIN (arrays/JSONB), GiST (geospatial), Partial (indexes filtered rows), and Expression (indexes function results).

### Real-world Analogy
B-tree is a telephone book sorted alphabetically. GIN is a book index that lists every page where a specific word appears. Partial index is a small directory containing phone numbers for local emergency services only, bypassing regular residential listings.

### Code Example
```sql
-- Create a partial index for active users
CREATE INDEX idx_active_users ON users (email) WHERE active = true;

-- Explain Analyze to verify query path
EXPLAIN ANALYZE SELECT email FROM users WHERE active = true;
```

### Common Interview Questions
- Why should you avoid creating indexes on columns with low cardinality (like gender)?
- What is the difference between B-tree and GIN indexes?
- Explain how CREATE INDEX CONCURRENTLY prevents write-blocking locks during index creation.

### Reference Links
- [PostgreSQL Docs: Indexes](https://www.postgresql.org/docs/current/indexes.html)

## Transactions and Locks

### Definition
Transactions group commands into ACID units. Locking prevents concurrency issues (deadlocks, dirty reads) using Row-Level locks (SELECT ... FOR UPDATE) and Table-Level locks. Isolation levels dictate visibility.

### Real-world Analogy
Imagine checking out a library book. BEGIN TRANSACTION is pulling the book from the shelf. SELECT FOR UPDATE is writing your name on the booking card: other readers can see the book exists, but they cannot edit or check it out until you return it (COMMIT).

### Code Example
```sql
BEGIN;
-- Row lock to prevent other transactions from modifying these rows
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

### Common Interview Questions
- Describe the four ANSI SQL isolation levels and which anomalies they prevent.
- What is a deadlock and how does PostgreSQL detect and handle it?
- Compare Shared locks, Exclusive locks, and Row locks.

### Reference Links
- [PostgreSQL Docs: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [PostgreSQL Docs: Explicit Locking](https://www.postgresql.org/docs/current/explicit-locking.html)

## Stored Procedures and Triggers

### Definition
Stored Procedures and Functions run PL/pgSQL code inside the database engine. Triggers execute a specific function automatically in response to DML operations (INSERT, UPDATE, DELETE) on a table.

### Real-world Analogy
A function is an automated calculator inside a spreadsheet. A trigger is a security camera: the second a door opens (INSERT event), the camera automatically flashes and records a timestamp entry in the log table.

### Code Example
```sql
-- Trigger function definition
CREATE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger binding
CREATE TRIGGER update_user_modtime
BEFORE UPDATE ON users
FOR EACH ROW EXECUTE FUNCTION update_modified_column();
```

### Common Interview Questions
- What is the difference between a Stored Procedure (can run transactions) and a Function?
- Compare BEFORE triggers and AFTER triggers.
- Explain the role of NEW and OLD records in PL/pgSQL trigger functions.

### Reference Links
- [PostgreSQL Docs: PL/pgSQL](https://www.postgresql.org/docs/current/plpgsql.html)
- [PostgreSQL Docs: Triggers](https://www.postgresql.org/docs/current/triggers.html)

## Full-Text Search

### Definition
Full-text search parses text into search tokens (tsvector) and matches queries (tsquery), ranking results using indexes (GIN) and dictionaries.

### Real-world Analogy
Instead of searching a book by scanning for the word "running", the database parses words into their root forms (stemming: "run"). A search query for "runs" or "ran" matches the text because both point to the same root token.

### Code Example
```sql
-- Simple full-text search match
SELECT title FROM posts
WHERE to_tsvector('english', body) @@ to_tsquery('english', 'database & prisma');
```

### Common Interview Questions
- What is the difference between tsvector and tsquery?
- How does to_tsvector handle word stemming and stop words?
- Explain how to index full-text columns using GIN indexes.

### Reference Links
- [PostgreSQL Docs: Full-Text Search](https://www.postgresql.org/docs/current/textsearch.html)

## Prisma ORM Setup and Schema

### Definition
Prisma is an open-source Next-generation ORM. Developers define database connection settings, generators, and data models inside a schema.prisma file.

### Real-world Analogy
schema.prisma is the architectural blueprint of your entire warehouse. You write the model schemas, relationships, and datatypes down in one clear format, and the ORM builds the database tables and outputs type-safe code blocks.

### Code Example
```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  posts Post[]
}

model Post {
  id       Int    @id @default(autoincrement())
  title    String
  authorId Int
  author   User   @relation(fields: [authorId], references: [id], onDelete: Cascade)
}
```

### Common Interview Questions
- What are generator blocks in schema.prisma and what do they compile?
- How do you manage environment variables inside schema.prisma datasource settings?
- Explain the difference between schema-first ORMs and code-first ORMs.

### Reference Links
- [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)

## Prisma Relations

### Definition
Prisma supports relations (one-to-one, one-to-many, many-to-many) mapped using foreign key relations, generating implicit (managed pivot table) or explicit relations and configuring cascade rules.

### Real-world Analogy
Think of a school database. One-to-many is a class having many students. Many-to-many is courses and students. Prisma implicit relation is a registrar managing courses and students in a ledger behind the scenes without you designing the ledger structure.

### Code Example
```prisma
// Many-to-Many implicit relation in Prisma
model Student {
  id      Int      @id @default(autoincrement())
  name    String
  courses Course[] // Implicit relation resolves pivot tables
}

model Course {
  id       Int       @id @default(autoincrement())
  name     String
  students Student[]
}
```

### Common Interview Questions
- Compare implicit many-to-many relations and explicit many-to-many relations in Prisma.
- What do onDelete: Cascade and onDelete: SetNull parameters enforce?
- How do you implement a self-referential relation in Prisma?

### Reference Links
- [Prisma: Relations](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)

## Prisma CRUD

### Definition
Prisma Client provides methods (findMany, findUnique, findFirst, create, update, delete, upsert, updateMany) with filter rules (where, select, include, orderBy, skip, take).

### Real-world Analogy
Prisma CRUD is like using a digital retrieval claw. You select a specific target box (findUnique), tell the claw to pull only the user email (select), or command it to retrieve the user's boxes of records (include) at the same time.

### Code Example
```typescript
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();

async function queryData() {
  const users = await prisma.user.findMany({
    where: { email: { endsWith: "@test.com" } },
    select: { email: true },
    take: 10,
    orderBy: { id: "desc" }
  });
}
```

### Common Interview Questions
- What is the difference between include and select parameters in Prisma queries? Can you use both in the same model block?
- How do findUnique and findFirst differ regarding index lookups?
- How do take and skip parameters manage offset-based pagination in Prisma?

### Reference Links
- [Prisma Client: CRUD API Reference](https://www.prisma.io/docs/reference/api-reference/prisma-client-reference)

## Prisma Advanced

### Definition
Prisma Advanced includes sequential transactions ($transaction([])), interactive transactions (callbacks), raw SQL executing ($queryRaw), client extensions, and full-text searches.

### Real-world Analogy
Sequential transaction is sending a list of three package orders to a clerk: they process them one-by-one and return the status of all three. Interactive transaction is standing on a live call with the clerk: you ask them to check shelf A first, and based on what they find, tell them to update shelf B.

### Code Example
```typescript
// Interactive Transaction example
const result = await prisma.$transaction(async (tx) => {
  const user = await tx.user.update({
    where: { id: 1 },
    data: { balance: { decrement: 100 } }
  });
  if (user.balance < 0) {
    throw new Error("Insufficient funds");
  }
  return user;
});
```

### Common Interview Questions
- Compare sequential transactions and interactive transactions in Prisma.
- How do you protect against SQL injection when using prisma.$queryRaw?
- Explain the role of Prisma Client Extensions.

### Reference Links
- [Prisma: Transactions](https://www.prisma.io/docs/concepts/components/prisma-client/transactions)
- [Prisma: Raw SQL Queries](https://www.prisma.io/docs/concepts/components/prisma-client/raw-database-access)

## Prisma Migrations

### Definition
Prisma Migrate compiles schema.prisma changes into structured SQL migration scripts, maintaining database status tracking in a _prisma_migrations table.

### Real-world Analogy
Migrations are version control commits for database structures. Instead of editing database tables on production by hand, you write the changes down in a file. The ORM runs the file and logs the release number so the database knows what update version it holds.

### Code Example
```bash
# Create a migration for development environment
npx prisma migrate dev --name init_db

# Run pending migrations in production environment
npx prisma migrate deploy
```

### Common Interview Questions
- Why should you use prisma migrate deploy instead of migrate dev in production pipelines?
- How do you resolve migration drift or conflicts when production changes occur?
- Explain what the prisma db seed command does and when it runs.

### Reference Links
- [Prisma: Migrations](https://www.prisma.io/docs/concepts/components/prisma-migrate)

## Prisma Performance

### Definition
Prisma performance optimization involves solving the N+1 query problem (Prisma handles relation lookups using select/include grouping), using pgBouncer connection pooling, and optimizing raw queries.

### Real-world Analogy
Instead of sending 10 couriers to the bank to fetch 10 records one-by-one, pgBouncer is a security shuttle: it collects the 10 requests, carries them in one van over a shared connection pool, and brings the answers back.

### Code Example
```typescript
// Avoid N+1 by fetching posts along with authors in one query
const posts = await prisma.post.findMany({
  include: {
    author: true // Prisma executes this in a single grouped join/batch query
  }
});
```

### Common Interview Questions
- How does Prisma prevent the N+1 query problem by default under the hood?
- What is pgBouncer and why is it necessary for serverless environments using PostgreSQL?
- How do you configure Prisma connection pooling limits in database URLs?

### Reference Links
- [Prisma: Performance Profiling](https://www.prisma.io/docs/concepts/components/prisma-client/query-optimization)

## Supabase and Neon

### Definition
Supabase and Neon are managed serverless PostgreSQL platforms. Neon supports branching (git-like database branches) and scaling to zero. Supabase adds authentication, storage buckets, and REST API autogeneration.

### Real-world Analogy
Managed Postgres is like renting an apartment where the landlord manages the electricity, plumbing, water, garbage collection, and front desk security. You only need to plug in your devices and live in the space.

### Code Example
```bash
# Neon connection string with pooling parameters (using transaction mode pgBouncer port)
DATABASE_URL="postgres://user:pass@ep-cool-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require"
```

### Common Interview Questions
- Explain Neon's database branching feature and how it benefits CI/CD pipelines.
- What is Row-Level Security (RLS) and how does Supabase use it?
- Describe the connection pool constraints on Supabase/Neon free tiers.

### Reference Links
- [Neon Documentation](https://neon.tech/docs/introduction)
- [Supabase Documentation](https://supabase.com/docs)

## Scenario-based Interview Questions for PostgreSQL & Prisma

### Scenario 1
A query that lists comments grouped by post takes 5 seconds on a database containing 2 million comments. The SQL statement searches comments where post_id matches. The explain plan shows a Seq Scan (sequential table scan). How do you resolve this?

*Expected Approach:*
1. State that a sequential scan means the engine is inspecting all 2 million comments because there is no index on the post_id column.
2. Fix this by creating a B-tree index on the post_id column of the comments table.
3. If using Prisma, this is configured by adding @@index([postId]) or @unique constraints in the model schema, compiling a migration, and running it to create the index.

### Scenario 2
Your node application uses Prisma. In production, you receive a "Transaction failed due to a write conflict or deadlock" error when multiple workers process invoice payments concurrently. How do you investigate and resolve this?

*Expected Approach:*
1. Explain that deadlocks happen when two transactions try to acquire locks on the same database rows in a different order.
2. Mitigate this by ensuring all transactions update rows in the exact same index order (e.g. always sort IDs before updating).
3. Switch from long interactive transactions to fast, single-operation queries, or write retry logic in the backend controller to re-run transactions that fail due to deadlocks.
