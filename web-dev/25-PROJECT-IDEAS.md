# 26 Project Ideas

This document outlines full-stack projects to apply your revision knowledge: Collaborative document editors, transactional e-commerce engines, microservice chat systems, custom reverse proxies, and CI/CD runner engines.

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

## E-commerce Engine with Distributed Transactions

### Definition
An e-commerce engine handles inventory, checkouts, payments, and notifications. It uses distributed transaction patterns (Saga orchestrators) and message queues to coordinate transactions across decoupled databases.

### Real-world Analogy
Imagine booking a flight and a hotel. You don't buy the flight if the hotel room is sold out. An e-commerce service is a travel coordinator: they book the flight first (service 1), and if booking the hotel (service 2) fails, they cancel the flight (compensating transaction) and refund your cash.

### Code Example
```javascript
// Saga orchestrator state loop concept
async function processOrderCheckout(orderId) {
  try {
    await inventoryService.reserveStock(orderId);
    await paymentService.chargeCard(orderId);
    await orderService.updateStatus(orderId, "completed");
  } catch (err) {
    // If any step fails, trigger compensating rollback transactions
    await inventoryService.releaseStock(orderId);
  }
}
```

### Common Interview Questions
- What is the difference between Orchestrator-based and Choreography-based Saga patterns?
- How do you ensure database transactions are idempotent across message queue retries?
- How do you handle out-of-stock race conditions when multiple users checkout at the same second?

### Reference Links
- [Microservices.io: Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [Stripe: Designing robust payment APIs](https://stripe.com/blog/designing-robust-payment-apis)

## Microservices Chat Platform

### Definition
A microservices chat platform splits functionalities (auth, gateway, message store, push notification) into isolated services. It uses WebSockets, API gateways, and Redis adapters to coordinate real-time delivery across server clusters.

### Real-world Analogy
A monolithic chat server is like a small post office with one worker who sorts letters, registers new users, and drives the delivery truck. A microservices chat platform is hiring dedicated teams: one team verifies ID badges at the door (auth), another distributes letters (gateway), and a third files envelopes in archives (message store).

### Code Example
```typescript
// Gateway service routing WebSocket event to Redis pub/sub channel
import { createClient } from "redis";
const redisPub = createClient();

async function broadcastMessage(roomId: string, message: any) {
  await redisPub.publish(`room:${roomId}`, JSON.stringify(message));
}
```

### Common Interview Questions
- Why do you need a Redis adapter to scale Socket.io across multiple server nodes?
- How does an API Gateway coordinate routing and authentication for microservices?
- Explain the pagination strategy for loading historical messages from Cassandra or MongoDB.

### Reference Links
- [Socket.io: Redis Adapter](https://socket.io/docs/v4/redis-adapter/)
- [Discord: Scaling real-time messaging](https://discord.com/blog/how-discord-stores-billions-of-messages)

## Custom Reverse Proxy Server

### Definition
A custom reverse proxy server intercepts incoming HTTP requests, routes them to internal ports based on target domains, terminates SSL, compresses payloads, and logs requests.

### Real-world Analogy
Imagine a luxury apartment building. Instead of mail carriers running up to individual apartments, Nginx is the lobby front desk clerk. They accept all mail (port 80/443), check envelopes for safety (SSL termination), and distribute them to the appropriate mailboxes (proxy_pass to local ports).

### Code Example
```javascript
// Custom Node.js reverse proxy server using http-proxy
const http = require("http");
const httpProxy = require("http-proxy");

const proxy = httpProxy.createProxyServer({});

const server = http.createServer((req, res) => {
  const host = req.headers.host;
  
  if (host === "api.example.com") {
    proxy.web(req, res, { target: "http://localhost:3000" }); // Route to Node API
  } else {
    proxy.web(req, res, { target: "http://localhost:8080" }); // Route to Frontend
  }
});

server.listen(80);
```

### Common Interview Questions
- What is the difference between a forward proxy and a reverse proxy?
- Why should you configure Host, X-Real-IP, and X-Forwarded-For headers in reverse proxies?
- How does Nginx optimize static file delivery compared to Node.js application servers?

### Reference Links
- [Nginx: Reverse Proxy Guide](https://docs.nginx.com/nginx/admin-guide/web-server/reverse-proxy/)
- [Node HTTP Proxy Library](https://github.com/http-party/node-http-proxy)

## Containerized CI/CD Runner Engine

### Definition
A containerized CI/CD runner engine executes automated build and test pipelines inside isolated Docker containers. It parses YAML workflow definitions, spawns containers, executes commands, and captures test logs.

### Real-world Analogy
Imagine a chemistry test lab. Instead of letting students mix chemicals on the shared lobby floor where they could spill or catch fire (host system execution), the supervisor opens an isolated, self-cleaning glass booth (Docker container) for each test. When the test finishes, the booth is washed down and reset.

### Code Example
```javascript
// Spawning a Docker container runner dynamically via the Docker CLI
const { exec } = require("child_process");

function runTestRunner(repoUrl) {
  // Spawn container, clone repo, install and run tests, then auto-remove (--rm)
  const cmd = `docker run --rm -v /var/run/docker.sock:/var/run/docker.sock alpine:latest sh -c "apk add git nodejs npm && git clone ${repoUrl} /app && cd /app && npm ci && npm test"`;
  
  exec(cmd, (err, stdout, stderr) => {
    if (err) console.error("Runner failed: ", err);
    console.log("Runner logs: ", stdout);
  });
}
```

### Common Interview Questions
- Why do CI/CD runners run build pipelines inside isolated containers rather than the host system?
- Explain how mounting /var/run/docker.sock allows Docker-in-Docker (DinD) container orchestration.
- How do you handle resource allocation (CPU/memory limits) on runners to prevent Denial of Service issues?

### Reference Links
- [Docker Docs: Docker-in-Docker](https://hub.docker.com/_/docker)
- [GitHub Actions: Self-hosted runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)

## Scenario-based Interview Questions for Project Architecture

### Scenario 1
You are building an e-commerce platform where checkouts take up to 10 seconds due to third-party bank verification APIs. During peak holiday sales, users click the "Buy" button multiple times, causing duplicate checkouts and locking database connections. How do you design this?

*Expected Approach:*
1. Implement a client-side block (disable buy button immediately after first click) and enforce a unique idempotency key for the transaction.
2. Route checkout requests to a message queue (BullMQ/Redis or RabbitMQ) instead of processing them synchronously in the API thread.
3. Return status 202 (Accepted) immediately with a polling token.
4. Run a background worker pool to process checkouts sequentially, verifying the idempotency key in Redis before charging the payment gateway.

### Scenario 2
You are designing a distributed chat platform where users frequently drop connections on mobile networks. If connection drops, messages are missed, and when they reconnect, fetching history takes 5 seconds, freezing the UI. How do you handle this?

*Expected Approach:*
1. Suggest using WebSockets for active chat communication, backed by a persistent message queue.
2. Store message logs with sequential IDs (or timestamps) in an indexed database table (PostgreSQL or Cassandra).
3. When client reconnects, instead of requesting all history, they pass their last received message ID or timestamp, and the server fetches only the delta messages since that ID.
4. Cache the last 50 messages of active rooms in Redis to ensure high-speed history fetches on reconnect.
