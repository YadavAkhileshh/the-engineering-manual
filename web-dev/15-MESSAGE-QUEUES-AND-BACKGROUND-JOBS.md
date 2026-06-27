# 15 Message Queues and Background Jobs

This guide covers message brokers and task queues: point-to-point vs pub-sub models, RabbitMQ exchanges, Kafka logs, BullMQ workers, message reliability, and consumer scaling.

## Message Queues vs Pub-Sub

### Definition
Message Queues operate on a point-to-point pattern where messages are delivered to exactly one consumer. Publish-Subscribe (Pub-Sub) operates on a one-to-many pattern where messages are broadcasted to all consumers subscribed to a topic.

### Real-world Analogy
A Message Queue is like a doctor's check-in desk: patients wait in line, and one patient is called by one doctor (point-to-point). Pub-Sub is like a radio station: the station broadcasts a single signal, and every person who tunes their radio to the frequency hears the music (one-to-many).

### Code Example
```javascript
// Conceptual representation:
// Queue: queue.send(message) -> resolved by single listener
// Pub-Sub: topic.publish(event) -> broadcast to all active listeners
```

### Common Interview Questions
- Describe when you would choose a Message Queue over a Pub-Sub broker.
- What happens to messages in a Pub-Sub model if there are no active subscribers?
- Compare RabbitMQ (can support both) and Apache Kafka regarding data persistence.

### Reference Links
- [RabbitMQ: Queue and Pub-Sub Concepts](https://www.rabbitmq.com/tutorials/amqp-concepts.html)
- [Enterprise Integration Patterns: Message Channels](https://www.enterpriseintegrationpatterns.com/patterns/messaging/MessageChannel.html)

## RabbitMQ Fundamentals

### Definition
RabbitMQ is a message broker that uses the AMQP protocol. Producers send messages to Exchanges (Direct, Fanout, Topic, Headers), which route them to Queues based on Bindings and Routing Keys. Consumers pull messages and send acknowledgments (ACK).

### Real-world Analogy
Imagine a corporate mailroom. You drop letters in different slots (Exchanges). Slot A is "Fanout": the clerk photocopies the letter and puts it in every single mail bin. Slot B is "Topic": the clerk looks at the routing key ("billing.invoice") and places it specifically in the accounting department bin (Queue).

### Code Example
```javascript
const amqp = require("amqplib");

async function receiveMessage() {
  const connection = await amqp.connect("amqp://localhost");
  const channel = await connection.createChannel();
  
  await channel.assertExchange("logs", "fanout", { durable: true });
  const q = await channel.assertQueue("", { exclusive: true });
  await channel.bindQueue(q.queue, "logs", "");

  channel.consume(q.queue, (msg) => {
    console.log("Received: " + msg.content.toString());
    channel.ack(msg); // Manual Acknowledgment
  });
}
```

### Common Interview Questions
- Compare the four core exchange types: Direct, Fanout, Topic, and Headers.
- What is the prefetch count (basic.qos) and how does it prevent consumer overload?
- Why are manual message acknowledgments preferred over auto-acknowledgments?

### Reference Links
- [RabbitMQ: Official Tutorials](https://www.rabbitmq.com/getstarted.html)

## Apache Kafka Fundamentals

### Definition
Apache Kafka is a distributed event streaming platform. It stores events in append-only logs structured into Topics and Partitions. Consumer Groups scale reads, tracking progress using stored Offsets.

### Real-world Analogy
Kafka is like a continuous roll of receipt tape in a checkout printer (append-only log). Cashiers write new transactions at the bottom (producers). Bookkeepers read the tape starting from line 100 (offset). The roll is never erased; you can re-read transactions as many times as you want.

### Code Example
```typescript
import { Kafka } from "kafkajs";

const kafka = new Kafka({ clientId: "my-app", brokers: ["localhost:9092"] });
const producer = kafka.producer();

async function sendMessage() {
  await producer.connect();
  await producer.send({
    topic: "user-signups",
    messages: [{ value: JSON.stringify({ userId: "101", email: "test@test.com" }) }]
  });
}
```

### Common Interview Questions
- How does Kafka achieve high horizontal scalability compared to traditional brokers?
- What are consumer groups and how does partition rebalancing work?
- What is the difference between Zookeeper and KRaft metadata modes?

### Reference Links
- [Apache Kafka: Documentation](https://kafka.apache.org/documentation/)
- [KafkaJS: Getting Started](https://kafka.js.org/)

## BullMQ

### Definition
BullMQ is a Node.js message queue and background job manager based on Redis. It handles asynchronous operations, tracking job states (Waiting, Active, Completed, Failed, Delayed) and supporting retry backoffs.

### Real-world Analogy
Imagine a kitchen ordering board. The waiter pins a ticket (job queued). If the ticket says "Prepare soup in 20 minutes" (Delayed), the board holds it. The chef (Worker) pulls active tickets, makes the soup, and marks the ticket completed. If the soup burns (Failed), the board prompts the chef to retry.

### Code Example
```javascript
const { Queue, Worker } = require("bullmq");

const jobQueue = new Queue("transcoding", { connection: { host: "127.0.0.1", port: 6379 } });

// Add delayed job with retry options
async function addJob() {
  await jobQueue.add("video-render", { videoId: "502" }, {
    delay: 5000,
    attempts: 3,
    backoff: { type: "exponential", delay: 1000 }
  });
}
```

### Common Interview Questions
- Why does BullMQ require Redis to operate?
- Explain the difference between exponential and fixed retry backoffs.
- How do you implement Flow/Parent-Child job relationships in BullMQ?

### Reference Links
- [BullMQ Documentation](https://docs.bullmq.io/)

## Redis as Message Broker

### Definition
Redis acts as a lightweight message broker using Pub/Sub (fire-and-forget), Lists (LPUSH/RPOP polling), or Streams (XADD/XREAD append-only logs), offering low latency but limited persistence compared to Kafka.

### Real-world Analogy
Redis Pub/Sub is like a megaphone. If you speak into the megaphone, anyone standing nearby hears you. If a listener walks out of the room for 5 seconds and you speak, they miss the message completely: there is no recording tape (persistence) holding the sound.

### Code Example
```javascript
// Redis List-based queue producer/consumer
// Producer: redis.lpush('task_queue', JSON.stringify(task))
// Consumer: redis.brpop('task_queue', 0) -> blocking pop waits for items
```

### Common Interview Questions
- What are the limitations of using Redis Pub/Sub for critical transactional events?
- How do Redis Streams differ from standard Redis lists?
- Compare Redis-based queues and dedicated AMQP brokers (RabbitMQ).

### Reference Links
- [Redis: Pub/Sub](https://redis.io/docs/manual/pubsub/)
- [Redis: Streams Introduction](https://redis.io/docs/data-types/streams/)

## Message Durability and Reliability

### Definition
Message durability ensures data is not lost on server crashes. It requires marking queues as durable, messages as persistent, enabling publisher confirmations, consumer acknowledgments, and routing failures to Dead Letter Queues (DLQ).

### Real-world Analogy
Imagine mailing a valuable jewelry parcel. You don't use regular mail (non-persistent). You register the parcel (durable), get a receipt slip when the post office scans it (publisher confirms), require a signature on delivery (acknowledgment), and redirect it to a secure vault (DLQ) if the recipient moves.

### Code Example
```javascript
// RabbitMQ channel setup for message durability
async function setupDurableChannel(channel) {
  // 1. Declare exchange as durable
  await channel.assertExchange("orders", "direct", { durable: true });
  // 2. Declare queue as durable
  await channel.assertQueue("order_queue", { durable: true });
  // 3. Publish message with persistent flag
  channel.publish("orders", "new_order", Buffer.from("data"), { persistent: true });
}
```

### Common Interview Questions
- What is a Poison Pill message and how does a Dead Letter Queue (DLQ) prevent it from breaking workers?
- Explain how publisher confirmations differ from consumer acknowledgments.
- How do you handle network partition crashes in clustered message broker environments?

### Reference Links
- [RabbitMQ: Reliability Guide](https://www.rabbitmq.com/reliability.html)

## Scaling Consumers

### Definition
Scaling consumers involves running multiple worker processes to pull tasks from shared queues. Strategies include the Competing Consumers Pattern, Kafka Partition scaling, and consumer backpressure configurations.

### Real-world Analogy
Imagine a shipping warehouse checkout table. As order volume grows, you open 5 more checkout tables. Each worker table (consumer) pulls boxes from the shared conveyor belt. They compete for boxes; when worker A is busy packing a box, worker B grabs the next box.

### Code Example
```javascript
// Worker concurrency configuration template (BullMQ)
// Spawns worker that processes up to 10 jobs concurrently
const worker = new Worker("image_processing", async (job) => {
  await processImage(job.data);
}, { concurrency: 10 });
```

### Common Interview Questions
- How does the Competing Consumers pattern distribute message processing load?
- What happens during a partition rebalance in Kafka when a new consumer joins the group?
- Explain the role of backpressure on consumer execution metrics.

### Reference Links
- [Microsoft: Competing Consumers pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)

## Idempotency in Message Processing

### Definition
Idempotency ensures that processing the same message multiple times produces the same system state as processing it once. This requires tracking message IDs in a deduplication store (Redis) and using unique database constraints.

### Real-world Analogy
Imagine an elevator button. If you press the "Floor 4" button once, the elevator moves to floor 4. If you press it 15 times in a row, the elevator still goes to floor 4, not floor 60: the action is idempotent.

### Code Example
```javascript
// Idempotency check before processing order
async function processOrderJob(job) {
  const { messageId, orderId, amount } = job.data;

  // Use Redis setnx (set if not exists) to claim message key
  const isUnique = await redis.set(`msg:${messageId}`, "processed", "NX", "EX", 86400);
  if (!isUnique) {
    console.log(`Duplicate message skipped: ${messageId}`);
    return; // Already processed
  }

  // Execute database transaction
  await db.orders.create({ id: orderId, amount });
}
```

### Common Interview Questions
- Why can network failures cause duplicate message delivery (e.g. at-least-once delivery)?
- Describe the Deduplication Pattern using a unique key store.
- How do database-level upserts ensure transaction idempotency?

### Reference Links
- [AWS: Idempotent API Design](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotence/)

## Scenario-based Interview Questions for Message Queues

### Scenario 1
You are building an invoice processing backend. When a user submits an invoice, a background worker compiles a PDF, calls a bank payment API, and emails the receipt. Sometimes, the bank gateway is slow, causing the worker to timeout. The message queue retries the task, causing the customer's card to be charged twice. How do you resolve this?

*Expected Approach:*
1. Explain that the issue is due to a lack of idempotency in the payment step.
2. Separate the workflow: compile the PDF and charge the card in separate, decoupled message queues.
3. Before invoking the bank payment API, generate a unique transaction ID (idempotency key) for the invoice and pass it to the payment gateway.
4. The payment gateway checks this key and rejects duplicate charges if the token was already processed.

### Scenario 2
Your background worker pool handles video transcoding. The transcoding queue spikes to 50,000 files during weekends, causing CPU utilization on your server cluster to hit 100% and bringing down the main HTTP server. How do you isolate and scale the system?

*Expected Approach:*
1. Decouple the worker code from the main Express HTTP server: run workers in a separate container/process pool.
2. Implement auto-scaling (e.g. KEDA in Kubernetes or AWS ECS Auto-Scaling) to spawn additional worker instances based on queue size metrics.
3. Configure prefetch limit controls (basic.qos or worker concurrency thresholds) to prevent single worker nodes from taking on more CPU tasks than their capacity limits.
