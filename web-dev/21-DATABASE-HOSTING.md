# 21 Database Hosting

This guide covers database hosting: managed vs self-hosted databases, relational setups (RDS, failover), NoSQL clusters, replication scaling, backup strategies, and connection pooling.

## Database Hosting Fundamentals

### Definition
Database hosting provides the infrastructure required to run database engines. Unlike application servers, database hosting requires persistent, high-I/O storage disks (SSD/NVMe), specialized memory configurations, and stateful lifecycles.

### Real-world Analogy
Imagine running a bank vault. An application server is like the teller window desk: you can swap the desks, repaint them, or replace them in minutes. The database host is the actual concrete vault room with heavy steel walls and high security locks: you cannot demolish and rebuild it every time a new client walks in.

### Code Example
```
// Databases require stateful storage configurations (e.g. AWS EBS gp3 volumes with high IOPS)
// If hosted on ephemeral containers (like serverless functions or raw Docker without volumes),
// files vanish on container restarts:
// docker run -d -p 5432:5432 postgres (WARNING: No volume mapping = transient data!)
```

### Common Interview Questions
- Why are databases rarely hosted on ephemeral serverless containers?
- What are IOPS and why are they critical for database hosting performance?
- Explain the difference between stateful execution and stateless execution.

### Reference Links
- [AWS: Cloud Databases Overview](https://aws.amazon.com/products/databases/)

## Managed vs Self-Hosted Databases

### Definition
Self-hosted databases are installed manually on virtual private servers (VPS). Managed database hosting (AWS RDS, MongoDB Atlas) delegates OS updates, backups, scalability configuration, and high availability setup to the cloud provider.

### Real-world Analogy
Self-hosted is like buying a farm: you dig the wells, buy the tractors, repair the fence, and guard the crop. Managed hosting is like shopping at a premium grocery store: you pay a fee and immediately get clean vegetables, and if a crop fails, the store sources them from another farm.

### Code Example
```bash
# Self-hosted manual database management script example
# Developer must write cron scripts to run backups:
pg_dump -U username -h localhost dbname > backup.sql
# ...then manage upload of backup.sql to S3, test updates, and patch Linux.
```

### Common Interview Questions
- What are the major cost and operational differences between self-hosting and managed hosting?
- What is High Availability (HA) and how do managed databases achieve it?
- Describe the administrative overhead associated with updating a self-hosted database version.

### Reference Links
- [AWS RDS Features](https://aws.amazon.com/rds/features/)
- [MongoDB Atlas Features](https://www.mongodb.com/cloud/atlas/features)

## Relational Database Hosting (AWS RDS)

### Definition
AWS Relational Database Service (RDS) is a managed database service. It supports automated database setup, Multi-AZ deployments (for synchronous master-standby replication and automatic failover), and Read Replicas (for scaling read workloads).

### Real-world Analogy
Multi-AZ is like having a copy of the vault records kept in building A (master) and building B (standby). If building A catches fire, the bank manager automatically diverts customer calls to building B (automatic failover) without the clients realizing.

### Code Example
```
// AWS RDS Multi-AZ Replication Path:
// App Server -> Writes to Primary RDS (AZ-1)
//                 v (Synchronous Replication)
//               Writes to Standby RDS (AZ-2)
// If Primary AZ-1 crashes, RDS updates DNS records to route traffic to Standby AZ-2.
```

### Common Interview Questions
- Explain how Multi-AZ deployment differs from Read Replicas in AWS RDS.
- How does AWS RDS handle automatic failover when a master instance crashes?
- What are the write performance implications of using synchronous replication in Multi-AZ?

### Reference Links
- [AWS RDS Multi-AZ Deployments](https://aws.amazon.com/rds/features/multi-az/)

## NoSQL Hosting (MongoDB Atlas)

### Definition
MongoDB Atlas is a fully managed cloud database service for MongoDB. It automates replica set clustering (ensuring data is copied across at least three nodes), handles sharding (horizontal partitioning), and configures network firewalls.

### Real-world Analogy
Imagine a school registrar. Instead of having one registrar write all records, Atlas has a team of three registrars (Replica Set). When registrar 1 writes a grade (Primary), registrar 2 and 3 copy it (Secondaries). If registrar 1 leaves, the remaining registrars elect a new primary writer immediately.

### Code Example
```bash
# MongoDB Atlas Connection URI with replica set configuration
mongodb+srv://admin:secret@cluster0.mongodb.net/prod?retryWrites=true&w=majority
```

### Common Interview Questions
- What is a MongoDB Replica Set and why does it require an odd number of nodes (e.g. minimum 3)?
- How does database sharding distribute write operations across clusters?
- Explain how to configure network IP access control lists (whitelisting) in MongoDB Atlas.

### Reference Links
- [MongoDB Atlas: Architecture](https://www.mongodb.com/docs/atlas/getting-started/)

## Database Backups and Restoration

### Definition
Database backups protect against data loss. Managed databases support Automated Snapshots, Point-in-Time Recovery (PITR, using database transaction logs to restore states to specific seconds), and validate backup files.

### Real-world Analogy
Automated snapshots are like taking a photo of a whiteboard at the end of every day. Transaction logs (PITR) are like recording a video of the whiteboard: if someone wipes out a section at 2:05 PM, you can rewind the video to 2:04 PM and reconstruct the drawing.

### Code Example
```
// Restore process concept:
// 1. Restore the last nightly snapshot (e.g. 12:00 AM)
// 2. Play forward the write-ahead logs (WAL) or oplog events up to 2:04 PM.
// 3. System database state is restored successfully.
```

### Common Interview Questions
- What is Point-in-Time Recovery (PITR) and how does it utilize transaction write logs?
- Contrast snapshot backups and logical data dumps (e.g. pg_dump).
- Why is it critical to test database backup restorations periodically?

### Reference Links
- [AWS RDS Backup and Restore](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Common_Tasks.BackupRestore.html)

## Read Replicas vs Write Replicas

### Definition
Read Replicas are copy instances of a primary database database. The primary handles write operations and streams changes asynchronously to read replicas, which handle read requests to optimize query throughput.

### Real-world Analogy
Imagine a busy newspaper office. One chief editor writes the master article (Primary: handles writes). Instead of having 10,000 readers crowd around the editor's desk to read the page, the office prints 10,000 paper copies (Read Replicas) and hands them to readers.

### Code Example
```javascript
// Application router directing query traffic based on operation type
const writeDb = connect("primary-db-url");
const readDb = connect("read-replica-url");

async function getUserProfile(userId) {
  return await readDb.query("SELECT * FROM users WHERE id = $1", [userId]);
}

async function updateUserProfile(userId, data) {
  return await writeDb.query("UPDATE users SET name = $1 WHERE id = $2", [data.name, userId]);
}
```

### Common Interview Questions
- What is replication lag and how does it cause read-after-write consistency anomalies?
- How do you configure application routing to direct reads to replicas and writes to primaries?
- When do read replicas fail to optimize write-bound database systems?

### Reference Links
- [AWS RDS Read Replicas](https://aws.amazon.com/rds/features/read-replicas/)

## Connection Pooling in Production

### Definition
Connection pooling maintains a cache of database socket connections that are reused for subsequent queries. Connection proxies (pgBouncer) prevent connection failures in high-volume or serverless environments.

### Real-world Analogy
Imagine a bank branch lobby. Opening a new database socket is like checking a customer's ID, print a fresh application form, verify their address, and signing a pass (costly overhead). A connection pool is having 10 pre-approved, pre-opened channels: when a client walks in, they speak immediately.

### Code Example
```ini
# pgBouncer configuration file example snippet (pgbouncer.ini)
[databases]
mydb = host=127.0.0.1 port=5432 dbname=prod

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction # session, transaction, or statement mode
max_client_conn = 1000
default_pool_size = 20
```

### Common Interview Questions
- Why do serverless functions (like AWS Lambda) easily exhaust database connection limits?
- Compare pgBouncer's Session Pooling and Transaction Pooling modes.
- How do connection limits on database engines affect application scaling configurations?

### Reference Links
- [pgBouncer Documentation](https://www.pgbouncer.org/)
- [Prisma: Connection Management](https://www.prisma.io/docs/concepts/components/prisma-client/connection-management)

## Scenario-based Interview Questions for Database Hosting

### Scenario 1
Your e-commerce application has a database query CPU utilization that hits 90% during promotions. 95% of database operations are clients browsing products, and 5% are users placing orders. You run a single PostgreSQL RDS instance. How do you scale this?

*Expected Approach:*
1. Spawn AWS RDS Read Replicas to handle the 95% browse-only queries, reducing CPU load on the Primary database.
2. Route read queries from the application layer to the read replica URL, and direct write queries (orders) to the Primary RDS URL.
3. Configure a Redis caching layer to store product descriptions, preventing redundant database read operations.

### Scenario 2
Your node application runs inside AWS Lambda. Every time a marketing campaign launches, 5,000 Lambda instances spawn in parallel to handle client requests. The database logs show "Too many connections" errors, and the entire API goes offline. How do you resolve this?

*Expected Approach:*
1. Explain that because Lambdas are stateless, each instance creates a database connection socket on boot, quickly exceeding database engine limits.
2. Implement pgBouncer (or RDS Proxy if deploying on AWS) in front of the PostgreSQL instance.
3. Configure pgBouncer in Transaction Pooling mode, allowing the 5,000 concurrent Lambda instances to share a small, pre-allocated pool of (e.g. 50) database connection sockets safely.
