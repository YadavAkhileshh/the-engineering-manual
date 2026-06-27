# 06 Node.js and Express

This section covers Node.js internals (event loop, libuv, child processes), Express middleware design, error patterns, file uploads, background tasks, and process management.

## Node.js Architecture

### Definition
Node.js is a single-threaded runtime environment built on Chrome's V8 engine. It uses an event-driven, non-blocking I/O model backed by libuv, which provides an asynchronous event loop and manages a background thread pool for system and file operations.

### Real-world Analogy
Imagine a busy restaurant with one single waiter. Instead of standing next to a table waiting for the chef to cook (blocking), the waiter takes your order, sends it to the kitchen team (thread pool), and moves to serve the next table. When the kitchen bells ring (I/O completion event), the waiter serves the food.

### Code Example
```javascript
// Demonstrating asynchronous non-blocking file read vs synchronous blocking read
const fs = require("fs");

// Non-blocking read (delegated to thread pool)
fs.readFile("example.txt", "utf8", (err, data) => {
  if (err) throw err;
  console.log("Async: " + data);
});

console.log("Next tasks run first");
```

### Common Interview Questions
- How does libuv handle asynchronous operations that the operating system doesn't natively support?
- What are the different phases of the Node.js event loop?
- Explain the role of V8 and how it communicates with libuv.

### Reference Links
- [Node.js Docs: Node.js Architecture](https://nodejs.org/en/about)
- [libuv Official Docs](https://libuv.org/)

## When Node.js is Not Suitable

### Definition
Node.js is not suitable for CPU-bound tasks (like image processing, cryptography, video transcoding). Because JavaScript runs on a single main thread, long-running CPU calculations block the event loop, preventing all other network requests from being processed.

### Real-world Analogy
Imagine the single waiter in the restaurant decides to go into the kitchen and spend 45 minutes baking a complex wedding cake. While they are baking the cake, no other customer in the dining room can order drinks, pay their bills, or be seated.

### Code Example
```javascript
// CPU-bound task that blocks the entire single thread
function blockMainThread() {
  let count = 0;
  for (let i = 0; i < 1e10; i++) {
    count++;
  }
  return count;
}
```

### Common Interview Questions
- Why do CPU-bound operations freeze network request handling in Node.js?
- How do you detect if the event loop is blocked in a production Node.js environment?
- What are the alternatives to Node.js when building a CPU-intensive service?

### Reference Links
- [Node.js Docs: Don't Block the Event Loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)

## Worker Threads

### Definition
Worker Threads (via the worker_threads module) allow Node.js to execute CPU-intensive tasks on parallel threads, enabling concurrent JavaScript execution and sharing memory using SharedArrayBuffer instances without blocking the main event loop.

### Real-world Analogy
The restaurant waiter hires an assistant chef specifically to handle baking cakes (CPU-intensive tasks). The main waiter continues taking client orders at the front, while the assistant chef works independently in the kitchen, notifying the waiter when the cake is complete.

### Code Example
```javascript
// main.js
const { Worker } = require("worker_threads");

function runWorker(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker("./worker.js", { workerData: data });
    worker.on("message", resolve);
    worker.on("error", reject);
  });
}
```

### Common Interview Questions
- What is the difference between Worker Threads and the Cluster module?
- How do Worker Threads communicate with the main thread (parentPort)?
- Why are Worker Threads lighter than Child Processes?

### Reference Links
- [Node.js Docs: Worker Threads](https://nodejs.org/api/worker_threads.html)

## Child Processes

### Definition
The child_process module allows Node.js to spawn entirely new operating system processes. It supports running shell commands (exec, spawn), creating isolated Node.js instances (fork), and establishing Inter-Process Communication (IPC) channels.

### Real-world Analogy
Imagine the restaurant owner opening a separate delivery bakery next door. The main restaurant (parent process) sends orders to the bakery (child process) via a direct phone line (IPC channel). The bakery runs under its own roof and has its own staff.

### Code Example
```javascript
// Forking a child process and sending messages via IPC
const { fork } = require("child_process");

const child = fork("child-task.js");

child.send({ task: "start" });
child.on("message", (msg) => {
  console.log("Received from child: ", msg);
  child.kill();
});
```

### Common Interview Questions
- Compare spawn, exec, execFile, and fork in terms of buffering and usage.
- How does Inter-Process Communication (IPC) work between parent and child processes?
- How do child processes handle standard streams (stdin, stdout, stderr)?

### Reference Links
- [Node.js Docs: Child Processes](https://nodejs.org/api/child_process.html)

## Streams

### Definition
Streams are collections of data that might not be available all at once or fit in memory. Node.js supports four stream types: Readable, Writable, Duplex (both readable/writable), and Transform (modify data during read/write). Backpressure handles flow speed differences.

### Real-world Analogy
Streams are like watching a movie on Netflix. Instead of waiting to download the entire 5GB video file to your hard drive before hitting play (buffering), you download and play small chunks of the movie sequentially as they arrive.

### Code Example
```javascript
const fs = require("fs");

const readStream = fs.createReadStream("large-input.txt");
const writeStream = fs.createWriteStream("output.txt");

// Piping handles backpressure automatically
readStream.pipe(writeStream);
```

### Common Interview Questions
- Explain what backpressure is and how streams handle it when a writer is slower than a reader.
- What is the difference between a Duplex stream and a Transform stream?
- Why do streams prevent heap out-of-memory errors when processing large files?

### Reference Links
- [Node.js Docs: Streams](https://nodejs.org/api/stream.html)

## Event Emitters

### Definition
The EventEmitter class is a core Node.js module used to handle custom event patterns. It allows objects to emit named events that trigger registered listener callback functions synchronously in the order they were registered.

### Real-world Analogy
Think of a store mailing list. Customers register their email addresses (listeners). When the store gets new stock, they announce the new release (emit event), which triggers emails to be sent to all subscribed customers.

### Code Example
```javascript
const EventEmitter = require("events");

class OrderLogger extends EventEmitter {}
const logger = new OrderLogger();

logger.on("order-placed", (orderId) => {
  console.log(`Log: Order ${orderId} was placed`);
});

logger.emit("order-placed", 9482);
```

### Common Interview Questions
- What happens if an event emitter emits an 'error' event and there are no listeners registered for it?
- How does the max listeners limit protect against memory leaks, and how do you change it?
- Why are event emitter callbacks executed synchronously inside the event loop?

### Reference Links
- [Node.js Docs: Events](https://nodejs.org/api/events.html)

## Cluster Module

### Definition
The Cluster module allows you to easily create a network of worker processes that run on individual CPU cores but share the same server port. The master process distributes incoming HTTP connections using a round-robin scheduling algorithm.

### Real-world Analogy
Imagine a bank branch with 8 teller windows. Instead of forcing all customers to queue in front of one teller, a branch manager (master process) stands at the door and directs arriving customers to tellers (worker processes) as they become free.

### Code Example
```javascript
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
} else {
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end("Served by worker process");
  }).listen(8000);
}
```

### Common Interview Questions
- How does the cluster module share a single port among multiple worker processes?
- What are sticky sessions and why are they necessary when using clusters with WebSockets?
- Compare Node.js clusters to PM2 Cluster Mode.

### Reference Links
- [Node.js Docs: Cluster](https://nodejs.org/api/cluster.html)

## Express Fundamentals

### Definition
Express is a minimal web application framework for Node.js. It organizes code into a middleware pipeline where incoming HTTP requests pass sequentially through helper functions that inspect, mutate, or terminate the request-response cycle.

### Real-world Analogy
Think of an assembly line at a car factory. The raw metal chassis (request) travels down the belt. Each worker (middleware) adds a part or inspects the build. At the end, the car is painted and rolled out of the factory (response sent).

### Code Example
```javascript
const express = require("express");
const app = express();

app.use(express.json()); // Built-in parsing middleware

app.get("/health", (req, res) => {
  res.status(200).send("OK");
});

app.listen(3000);
```

### Common Interview Questions
- Explain the role of the next() function inside Express middleware.
- What happens if you do not call next() or send a response inside a middleware function?
- Describe the Express request-response lifecycle.

### Reference Links
- [Express: Getting Started](https://expressjs.com/en/starter/installing.html)
- [Express: Writing Middleware](https://expressjs.com/en/guide/writing-middleware.html)

## Express Routing

### Definition
Express Routing defines how application endpoints respond to client requests. It supports HTTP method mappings, route parameters, wildcards, regular expressions, and Router-level modular paths.

### Real-world Analogy
Imagine a post office sorting desk. Letters are sorted by target labels: letters marked "Airmail" (GET methods) go to planes, letters marked "Parcel" (POST methods) go to trucks, and zip code sub-addresses (route parameters) route packages to specific mailbags.

### Code Example
```javascript
const express = require("express");
const router = express.Router();

// Dynamic route parameters
router.get("/users/:id", (req, res) => {
  res.send(`User ID: ${req.params.id}`);
});

module.exports = router;
```

### Common Interview Questions
- How do you write regex route paths in Express?
- What is the difference between req.params, req.query, and req.body?
- How does router-level middleware help organize large application codebases?

### Reference Links
- [Express: Routing Guide](https://expressjs.com/en/guide/routing.html)

## Middleware Deep Dive

### Definition
Middleware includes Application-level, Router-level, Built-in, Third-party, and Error-handling functions. Error-handling middleware is unique: it is declared with four arguments (err, req, res, next) instead of three.

### Real-world Analogy
Think of a secure laboratory check-in. Application-level check is verifying your ID at the gate. Router-level check is wearing a biohazard suit before entering Room B. Error-handling is the emergency clean-up team that is only called if a beaker breaks.

### Code Example
```javascript
const express = require("express");
const app = express();

// Custom Middleware
app.use((req, res, next) => {
  req.timestamp = Date.now();
  next();
});

// Error handling middleware (must have 4 parameters)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: "Internal Server Error" });
});
```

### Common Interview Questions
- Why must error-handling middleware contain exactly four arguments?
- How does the declaration order of middleware affect request processing?
- How do third-party middlewares like helmet secure Express apps?

### Reference Links
- [Express: Using Middleware](https://expressjs.com/en/guide/using-middleware.html)

## Error Handling Patterns

### Definition
Error handling patterns secure asynchronous routes. Because Express 4 does not catch unhandled async rejections automatically, developers wrap async handlers in try-catch blocks or use async handler wrappers to forward errors to next().

### Real-world Analogy
Imagine a trapeze act. If a performer drops (an error occurs) during a regular jump (synchronous task), the safety net catches them. If they jump in the dark (asynchronous task), you must install a custom guide line (async wrapper) to direct them safely to the net.

### Code Example
```javascript
// Async wrapper pattern
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get("/data", asyncHandler(async (req, res) => {
  const result = await fetchThirdPartyData(); // Throws error
  res.send(result);
}));
```

### Common Interview Questions
- Why do unhandled async errors crash Express 4 servers, and how does Express 5 address this?
- What is the purpose of an AppError custom class extending the Error class?
- Describe a professional, standardized JSON error response structure.

### Reference Links
- [Express: Error Handling](https://expressjs.com/en/guide/error-handling.html)

## Request Validation

### Definition
Request validation is the process of inspecting incoming request payloads (body, query, params) against a predefined schema. Libraries like Zod compile schema rules and sanitize inputs before routing requests to business controllers.

### Real-world Analogy
Think of a quality checker at a factory assembly line. Before a batch of raw plastic (input data) is fed into the molding machine, the checker verifies its composition. If it contains wood or metal particles, it is rejected before it can damage the mold.

### Code Example
```javascript
const { z } = require("zod");

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
});

const validate = (schema) => (req, res, next) => {
  const result = schema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.errors });
  }
  req.validatedBody = result.data;
  next();
};
```

### Common Interview Questions
- Why should you validate client input on the server even if you already validate it on the frontend?
- How do you parse and format Zod schema errors into a user-friendly format?
- What is the difference between schema validation and schema sanitization?

### Reference Links
- [Zod Official Documentation](https://zod.dev/)

## File Uploads

### Definition
File uploading in Node.js handles multipart/form-data. Libraries like Multer intercept files, stream them into disk storage or memory buffers, filter file extensions, and enforce file size limits.

### Real-world Analogy
Imagine a mailroom receiving packages. Multer is the clerk who sorts letters from boxes. If a package contains a dangerous substance or is too heavy, the clerk rejects it. Otherwise, they log the contents and place them in the storage room.

### Code Example
```javascript
const multer = require("multer");
const upload = multer({
  dest: "uploads/",
  limits: { fileSize: 1024 * 1024 * 5 }, // 5MB limit
  fileFilter: (req, file, cb) => {
    if (file.mimetype === "image/png") cb(null, true);
    else cb(new Error("Only PNGs allowed"), false);
  }
});

app.post("/profile/avatar", upload.single("avatar"), (req, res) => {
  res.send("File uploaded successfully");
});
```

### Common Interview Questions
- Compare Memory Storage vs Disk Storage options in Multer.
- How do you handle multiple file uploads in a single request?
- How do you stream file uploads directly to AWS S3 without filling local server disks?

### Reference Links
- [Multer GitHub Repository](https://github.com/expressjs/multer)

## Rate Limiting

### Definition
Rate limiting restricts the number of requests a client can make to an API within a specified timeframe, protecting the system against abuse, brute-force attacks, and Denial of Service (DoS) crashes.

### Real-world Analogy
Think of a free water fountain. To prevent one person from holding the button down and filling up a huge tank while others wait, the park installs a valve that automatically shuts off water flow for 30 seconds after 2 liters are dispensed.

### Code Example
```javascript
const rateLimit = require("express-rate-limit");

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each IP to 100 requests per window
  message: "Too many requests, please try again later."
});

app.use("/api/", apiLimiter);
```

### Common Interview Questions
- Explain the difference between Fixed Window and Sliding Window rate limiting algorithms.
- How do you configure rate limiters to run across server clusters using Redis storage?
- How do you bypass rate limits for trusted automated tests or internal systems?

### Reference Links
- [express-rate-limit GitHub](https://github.com/express-rate-limit/express-rate-limit)

## Security Middleware Setup

### Definition
Security middleware protects Express applications by setting appropriate HTTP headers, configuring CORS parameters, sanitizing NoSQL injection vectors, and stripping cross-site scripting (XSS) inputs.

### Real-world Analogy
Imagine preparing a building for a storm. You lock the doors (CORS), board up the windows (Helmet HTTP headers), install gutters to divert mud (express-mongo-sanitize), and check visitors for wet boots before they step on the rug (xss-clean).

### Code Example
```javascript
const helmet = require("helmet");
const cors = require("cors");
const mongoSanitize = require("express-mongo-sanitize");

app.use(helmet()); // Sets protective security headers
app.use(cors({ origin: "https://trustedapp.com" }));
app.use(mongoSanitize()); // Prevents query selector injection
```

### Common Interview Questions
- Name three security headers that Helmet configures out of the box.
- How does NoSQL operator injection work and how does mongoSanitize protect against it?
- What are HTTP Parameter Pollution (HPP) attacks and how do you prevent them?

### Reference Links
- [Helmet JS Docs](https://helmetjs.github.io/)
- [OWASP Node.js Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html)

## Authentication Middleware

### Definition
Authentication middleware verifies the identity of the user making a request by parsing incoming authorization headers (Bearer token), decoding and validating JWT signatures, and adding user metadata to the Request context.

### Real-world Analogy
Think of an airline boarding gate. The gate agent (auth middleware) inspects your passport and boarding pass (JWT token). If it is authentic and unexpired, they write your seat number (user meta) on the manifest and let you board.

### Code Example
```javascript
const jwt = require("jsonwebtoken");

const authenticateToken = (req, res, next) => {
  const authHeader = req.headers["authorization"];
  const token = authHeader && authHeader.split(" ")[1];

  if (!token) return res.sendStatus(401);

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};
```

### Common Interview Questions
- What is the difference between authentication and authorization in Express middleware?
- How do you implement Role-Based Access Control (RBAC) middleware?
- How do you handle token expiration and refresh checks on backend routes?

### Reference Links
- [JWT Introduction](https://jwt.io/introduction)

## Logging

### Definition
Logging is the practice of outputting application events. Winston provides configurable log storage (transports to files, consoles, or databases) and formats. Morgan is HTTP request-logging middleware that runs automatically inside the routing chain.

### Real-world Analogy
Imagine a ship captain's log. Morgan is the automation system that automatically prints entry and exit logs of cargo. Winston is the captain's diary: you write personal notes about engine issues (warnings), radar scans (info), or major deck leaks (errors).

### Code Example
```javascript
const winston = require("winston");
const morgan = require("morgan");

const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: "combined.log" })
  ]
});

// Integration with morgan
app.use(morgan("combined", { stream: { write: (msg) => logger.info(msg.trim()) } }));
```

### Common Interview Questions
- Why is console.log avoided in production, and how do Winston transports resolve this?
- What are log levels (error, warn, info, debug) and how do they help filter logs?
- Explain the concept of log rotation and why it is critical for production disk storage.

### Reference Links
- [Winston GitHub](https://github.com/winstonjs/winston)
- [Morgan GitHub](https://github.com/expressjs/morgan)

## API Versioning Strategies

### Definition
API versioning guarantees route compatibility when APIs change. Strategies include URL paths (/v1/users), request headers (Accept: version=1.0), or query parameters (?version=1).

### Real-world Analogy
Think of a city bus map. If the city remodels the streets, they do not erase the old routes overnight. They run the "Route 5 - Legacy Version" and the "Route 5 - Express Route" at the same time to avoid leaving commuters stranded.

### Code Example
```javascript
const express = require("express");
const app = express();

const v1Router = require("./routes/v1");
const v2Router = require("./routes/v2");

// URL path-based API versioning
app.use("/api/v1", v1Router);
app.use("/api/v2", v2Router);
```

### Common Interview Questions
- Compare URL path versioning vs Header-based versioning regarding cache efficiency and developer experience.
- When is a breaking change considered severe enough to justify bumping an API version?
- How does API routing logic scale when you have many versions active?

### Reference Links
- [Microsoft API Design: API Versioning](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#api-versioning)

## Pagination Patterns

### Definition
Pagination breaks down huge database collections. Offset-based pagination uses LIMIT and SKIP but slows down on large datasets. Cursor-based pagination uses unique resource identifiers (cursors) to fetch subsequent items, ensuring constant search speeds.

### Real-world Analogy
Offset-based is like turning to page 450 in a textbook by counting every single page from page 1. Cursor-based is using a bookmark: you know exactly where you stopped reading, and you read the next 10 pages immediately.

### Code Example
```javascript
// Offset-based pagination endpoint
app.get("/items", async (req, res) => {
  const limit = parseInt(req.query.limit) || 10;
  const offset = parseInt(req.query.offset) || 0;

  const items = await db.collection("items").find().skip(offset).limit(limit).toArray();
  res.json({ data: items, metadata: { count: items.length, offset, limit } });
});
```

### Common Interview Questions
- Why does offset pagination (SKIP) degrade in performance when skipping millions of records?
- How does cursor-based pagination prevent missing or duplicated items when database rows are inserted/deleted?
- What metadata parameters should you return in a standard paginated response?

### Reference Links
- [Slack API: Offset vs Cursor Pagination](https://api.slack.com/docs/pagination)

## Socket.io Integration

### Definition
Socket.io is a library that enables low-latency, bidirectional, and event-driven communication between web clients and Node.js servers, using WebSocket connections under the hood with HTTP long-polling fallback.

### Real-world Analogy
Socket.io is like establishing an open, active intercom line between two security guards. Instead of paging the other guard and waiting for a dial-up handshake on every check, you keep the channel open and speak instantly.

### Code Example
```javascript
const express = require("express");
const http = require("http");
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server, { cors: { origin: "*" } });

io.on("connection", (socket) => {
  console.log("Client connected: " + socket.id);
  socket.on("chat-message", (msg) => {
    io.emit("message-received", msg); // Broadcast message
  });
});

server.listen(3000);
```

### Common Interview Questions
- How does Socket.io differ from native HTML5 WebSockets?
- Explain the role of rooms and namespaces in Socket.io.
- How do you scale Socket.io across multiple server nodes using the Redis adapter?

### Reference Links
- [Socket.io Official Documentation](https://socket.io/docs/v4/)

## Email Sending with Nodemailer

### Definition
Nodemailer is a Node.js module that allows applications to send emails using SMTP mail servers, managing HTML templates, attachments, and asynchronous transmission queues.

### Real-world Analogy
Nodemailer is like dropping letters into a corporate mailroom slot. You write the envelope addresses and compile the document. Nodemailer handles packaging the paper sheets, weighing the package, and dispatching it to the mail carrier truck.

### Code Example
```javascript
const nodemailer = require("nodemailer");

const transporter = nodemailer.createTransport({
  host: "smtp.example.com",
  port: 587,
  auth: { user: "user", pass: "pass" }
});

async function sendMail() {
  await transporter.sendMail({
    from: "sender@example.com",
    to: "receiver@example.com",
    subject: "Hello",
    html: "<b>Welcome to our platform</b>"
  });
}
```

### Common Interview Questions
- Why is it bad practice to send transactional emails synchronously within the HTTP request-response cycle?
- How do you compile dynamic templates (e.g. using EJS or Handlebars) with Nodemailer?
- Explain the difference between SMTP login and using transactional email APIs like SendGrid.

### Reference Links
- [Nodemailer Documentation](https://nodemailer.com/)

## Background Jobs with BullMQ

### Definition
BullMQ is a message queue system for Node.js based on Redis. It allows developers to offload slow operations (like email dispatch or PDF exports) to isolated background workers, supporting retry, delays, and scheduling.

### Real-world Analogy
Think of a fast-food drive-thru. If you order 50 burgers (a slow job), the cashier does not freeze the queue and make everyone wait. They tell you to park in a side slot (job queued). A separate kitchen worker brings the burgers to your car when ready (worker completes job).

### Code Example
```javascript
// producer.js
const { Queue } = require("bullmq");
const emailQueue = new Queue("emails", { connection: { host: "127.0.0.1", port: 6379 } });

async function addEmailJob() {
  await emailQueue.add("send-welcome", { email: "user@test.com" }, { attempts: 3 });
}

// worker.js
const { Worker } = require("bullmq");
const worker = new Worker("emails", async (job) => {
  console.log(`Processing email job: ${job.id} for ${job.data.email}`);
}, { connection: { host: "127.0.0.1", port: 6379 } });
```

### Common Interview Questions
- Why does BullMQ require Redis to store job queue states?
- What are dead letter queues (or failed states) and how do you handle unresolved jobs?
- How do you implement cron-like recurring jobs in BullMQ?

### Reference Links
- [BullMQ Official Docs](https://docs.bullmq.io/)

## Process Management with PM2

### Definition
PM2 is a production process manager for Node.js. It keeps applications alive forever, reload paths with zero downtime, monitors metrics, and facilitates simple clustering without code modifications.

### Real-world Analogy
PM2 is like hiring an automated facility manager for your server. If the server trips and crashes (throws uncaught exceptions), the manager lifts it back up instantly. If a power outage occurs, they restart the app on system reboot automatically.

### Code Example
```bash
# Start an app in cluster mode using all available CPU cores
pm2 start app.js -i max

# Monitor running processes and resource usage
pm2 monit

# Save the process list to revive on system reboot
pm2 save
```

### Common Interview Questions
- Compare 'pm2 restart' and 'pm2 reload' in terms of application downtime.
- How does PM2 cluster mode coordinate port sharing on a production machine?
- What is an ecosystem.config.js file and how do you manage environment variables with it?

### Reference Links
- [PM2 Documentation](https://pm2.keymetrics.io/)

## Scenario-based Interview Questions for Node.js and Express

### Scenario 1
A reporting API endpoint aggregates data from multiple external sources. Sometimes, external calls take up to 20 seconds to resolve. During this time, other users complain they cannot load simple profile pages from the Express server. Why is this happening and how do you fix it?

*Expected Approach:*
1. Explain that since Node.js is single-threaded, if the reporting API performs heavy CPU calculation on the response, it blocks the event loop.
2. If the delay is strictly network I/O waiting, Node.js can handle other requests. However, if the server resources (connection sockets or memory pool) are exhausted due to holding open 20-second connections, the app becomes unresponsive.
3. Solve this by offloading the reporting task to a background queue (BullMQ), immediately returning status 202 (Accepted) to the user, and notifying them via WebSockets or polling when the report is ready.

### Scenario 2
Your Node.js app uploads CSV files containing 500,000 rows. The server crashes with a "Javascript heap out of memory" error when two users run the upload concurrently. How do you resolve this?

*Expected Approach:*
1. Identify that reading the entire file into a memory buffer or array at once (e.g. fs.readFile or buffering multer memory) exceeds the V8 engine's heap limit.
2. Fix this by using Node.js Streams (specifically Readable streams) and piping them into a CSV parser stream (like csv-parser) that processes rows sequentially chunk-by-chunk.
3. This maintains a small, constant memory footprint regardless of the file size.
