# 24 Scenario-Based Interview Questions

This document outlines multi-system engineering scenarios: Collaborative document design, real-time chat, payment gateways, rate limiters, analytics dashboards, and social feed scaling.

## Real-time Collaborative Document Editor

### Definition
A real-time collaborative document editor allows multiple users to edit the same file concurrently. It resolves conflicting edits using Conflict-free Replicated Data Types (CRDTs) or Operational Transformation (OT) over WebSocket channels.

### Real-world Analogy
Imagine multiple painters painting on the same canvas at the same time. If painter A paints a tree on the left and painter B paints a rock on the right, they don't erase each other's work. If they both try to paint on the center square, they use coordinate rules (CRDT coordinates) to blend their brushstrokes.

### Code Example
```javascript
// Server-side Socket.io coordinator for doc synchronization
io.on("connection", (socket) => {
  socket.on("doc-update", (delta) => {
    // Broadcast changes to all other connected clients
    socket.broadcast.emit("doc-sync", delta);
  });
});
```

### Common Interview Questions
- Explain the key differences between Operational Transformation (OT) and Conflict-free Replicated Data Types (CRDTs).
- How do you handle network latency offsets when users edit offline and reconnect?
- Describe the database storage schema needed for fast document recovery and history tracking.

### Reference Links
- [Yjs: Shared editing framework](https://docs.yjs.dev/)
- [Figma: How collaborative editing works](https://www.figma.com/blog/realtime-editing-of-ordered-sequences/)

## Chat System with Message History

### Definition
A chat system coordinates low-latency text deliveries. It uses WebSockets for active chat communication, writes messages to database queues, and aggregates history tables using paginated indexes.

### Real-world Analogy
Think of a walkie-talkie channel. When you speak, everyone listening on your channel hears you instantly. At the same time, a stenographer sitting in the office listens to the channel, types every word down in a diary, and stores the diary pages in a searchable archive cabinet.

### Code Example
```typescript
interface ChatMessage {
  id: string;
  senderId: string;
  roomId: string;
  content: string;
  timestamp: Date;
}
// Query to fetch paginated chat history using database cursors
// SELECT * FROM messages WHERE room_id = $1 AND id < $2 ORDER BY id DESC LIMIT 20;
```

### Common Interview Questions
- How do you scale a chat WebSocket server across multiple server nodes using a Redis adapter?
- Explain the pagination query strategy used to load millions of historical chat messages.
- How do you track user online/offline status in a distributed chat architecture?

### Reference Links
- [Discord: How Discord stores billions of messages](https://discord.com/blog/how-discord-stores-billions-of-messages)
- [Socket.io: Scaling Guide](https://socket.io/docs/v4/using-multiple-nodes/)

## Payment Gateway Integration

### Definition
Payment gateway integration connects web apps to payment processors (Stripe, Razorpay). It uses webhooks to receive transaction notifications and updates database purchase records asynchronously.

### Real-world Analogy
Imagine purchasing a ticket at a train station. Instead of wait at the ticket machine for 15 minutes while the bank verifies your credit card, the machine prints a temporary slip. The bank sends a secure envelope to the train office (webhook verification) 2 minutes later confirming the payment.

### Code Example
```javascript
// Express controller validating Stripe webhook signatures
app.post("/webhook", express.raw({ type: "application/json" }), (req, res) => {
  const sig = req.headers["stripe-signature"];
  let event;
  try {
    event = stripe.webhooks.constructEvent(req.body, sig, endpointSecret);
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }
  
  if (event.type === "payment_intent.succeeded") {
    handlePaymentSuccess(event.data.object);
  }
  res.json({ received: true });
});
```

### Common Interview Questions
- Why should you verify webhook signatures on your backend before updating purchase statuses?
- How do you prevent double-charging credit cards if a webhook is delivered multiple times (idempotency)?
- How do you handle refund operations and payment reconciliation logs?

### Reference Links
- [Stripe: Webhook Signature Verification](https://stripe.com/docs/webhooks/signatures)
- [Razorpay: Webhooks Documentation](https://razorpay.com/docs/webhooks/)

## Distributed Rate Limiter

### Definition
A distributed rate limiter restricts API request counts across multiple application servers. It uses Redis storage and algorithms like Token Bucket or Sliding Window Log to track client counts.

### Real-world Analogy
Think of a subway station turnstile. To prevent overcrowding, the station manager installs a central counter (Redis). Every time a user enters any turnstile across the station, the counter counts down. If the count hits zero, all turnstiles lock for 5 minutes.

### Code Example
```javascript
// Redis Sliding Window Rate Limiter transaction template
async function isRateLimited(userId, limit, windowSizeSeconds) {
  const key = `rate:${userId}`;
  const now = Date.now();
  const clearBefore = now - (windowSizeSeconds * 1000);
  
  const multi = redis.multi();
  multi.zremrangebyscore(key, 0, clearBefore); // Remove stale timestamps
  multi.zadd(key, now, now); // Add current timestamp
  multi.zcard(key); // Count active timestamps
  
  const results = await multi.exec();
  const requestCount = results[2];
  return requestCount > limit;
}
```

### Common Interview Questions
- Compare the Token Bucket and Sliding Window Log rate limiting algorithms.
- How do you coordinate rate limiting across multiple servers without creating a performance bottleneck in Redis?
- Explain the difference between IP-based rate limiting and User-ID-based rate limiting.

### Reference Links
- [Figma: An alternative rate limiting algorithm](https://www.figma.com/blog/an-alternative-approach-to-rate-limiting/)
- [Redis: Pattern: Rate Limiter](https://redis.io/commands/incr/)

## High-throughput Analytics Dashboard

### Definition
A high-throughput analytics dashboard collects, stores, and aggregates millions of event metrics. It offloads ingestion using write-heavy databases (ClickHouse, Cassandra) and caches dashboard summaries in Redis.

### Real-world Analogy
Imagine counting cars on a highway. Instead of writing the model, color, and driver's name in a huge binder for every car (OLTP databases), a radar camera registers raw numbers on a counter clicker (write-heavy database) and sends daily summaries to the city planner.

### Code Example
```sql
-- ClickHouse analytical query schema example
-- ClickHouse aggregates millions of event logs in milliseconds
SELECT event_name, COUNT(*), avg(duration)
FROM system_events
GROUP BY event_name
LIMIT 10;
```

### Common Interview Questions
- Compare OLTP (transactional) and OLAP (analytical) databases regarding query design.
- What is data pre-aggregation and how does it speed up analytics dashboards?
- How do you handle writing spikes in analytics collectors (e.g. message queue buffers)?

### Reference Links
- [ClickHouse: What is Column-oriented Database?](https://clickhouse.com/docs/en/about-us/column-oriented-database/)
- [Grafana: Dashboard Best Practices](https://grafana.com/docs/grafana/latest/dashboards/best-practices/)

## Social Feed Scaling

### Definition
Social feed scaling handles content distribution. It uses Fan-out-on-Write (pre-compiling feeds for active users) or Fan-out-on-Read (aggregating feeds on demand), balancing the modes based on user follower counts.

### Real-world Analogy
Imagine sending newsletter mail. If you write a letter and photocopy it 1,000 times, placing copies in each subscriber's mailbox immediately (Fan-out-on-Write), it is fast for them to read. However, if a celebrity has 10 million fans, copying the letter 10 million times breaks the printer. Instead, fans read the master board in the town square (Fan-out-on-Read).

### Code Example
```javascript
// Fan-out-on-Write feed aggregation write loop
async function publishPost(userId, postData) {
  const post = await db.posts.save(postData);
  const followerIds = await db.users.getFollowerIds(userId);
  
  // Pre-compile feed for followers by inserting into their feed cache
  const pipeline = redis.pipeline();
  followerIds.forEach(followerId => {
    pipeline.lpush(`feed:${followerId}`, post.id);
    pipeline.ltrim(`feed:${followerId}`, 0, 199); // Cap feed size to 200 items
  });
  await pipeline.exec();
}
```

### Common Interview Questions
- Contrast the Fan-out-on-Write and Fan-out-on-Read feed generation models.
- How do you handle celebrity accounts (high follower counts) in feed distribution architectures?
- Explain how caching strategies optimize home feed response latency.

### Reference Links
- [Twitter: Scale feed generation architecture](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2013/the-infrastructure-behind-twitter-scale)
- [Redis: Social Feed Patterns](https://redis.io/solutions/social/)

## Scenario-based Interview Questions for Multi-system Design

### Scenario 1
You are designing a ride-sharing dashboard (like Uber). The map must display 100 nearby drivers moving in real-time. Drivers update their coordinates every 3 seconds. 10,000 clients view the dashboard concurrently. How do you design this?

*Expected Approach:*
1. Suggest using WebSockets for real-time location updates.
2. Drivers stream their coordinates to the backend via lightweight WebSocket or UDP connections.
3. Store coordinate indexes in Redis Geospatial indexes (GEOADD) to allow fast distance searches.
4. Instead of querying the database for every client, the backend coordinates geospatial updates and broadcasts them to clients in specific geographic rooms (Socket.io rooms) every 3 seconds.

### Scenario 2
An online flash sale starts at 12 PM. 100,000 users try to buy 100 available items at the same second. The checkout API is overwhelmed, the database locks up, and users receive server timeout errors. How do you design a system to handle this?

*Expected Approach:*
1. Propose placing a message queue (like RabbitMQ or Redis List) in front of the checkout processing service.
2. When the sale starts, the API immediately puts buy requests in the queue and returns status 202 (Request Accepted) to the client, along with a polling token.
3. Run a restricted pool of background workers that pull tasks from the queue and check out items sequentially.
4. Once the 100 items are sold, the workers reject the remaining jobs in the queue, updating their status to "Sold Out" so polling clients see the result immediately without database lockups.
