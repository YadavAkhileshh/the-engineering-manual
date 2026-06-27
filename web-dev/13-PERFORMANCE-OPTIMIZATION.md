# 13 Performance Optimization

This guide covers full-stack performance optimization: Frontend metrics (Web Vitals, code splitting), Backend optimization (caching, profiling), caching strategies, and media delivery.

## Frontend Performance Optimization

### Definition
Frontend performance optimization involves minimizing load time, rendering latency, and browser resource consumption. Strategies include code splitting, dynamic imports, tree shaking, asset compression, and Critical Rendering Path optimization.

### Real-world Analogy
Imagine traveling to a campground. Instead of carrying a massive 50kg backpack containing your entire winter wardrobe, stove, and tent (monolithic bundle), you carry only a light backpack with snacks. You set up a delivery driver to bring the heavy sleeping bag (dynamic import) only when night arrives.

### Code Example
```jsx
// React code splitting with lazy loading and dynamic imports
import React, { Suspense, lazy } from "react";

const HeavyChart = lazy(() => import("./components/HeavyChart"));

function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<div>Loading Chart...</div>}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}
```

### Common Interview Questions
- Describe the concept and benefits of Tree Shaking in modern bundlers like Webpack/Vite.
- What is Critical CSS and how does it speed up page rendering?
- How do async and defer attributes affect the execution of script tags?

### Reference Links
- [web.dev: Fast Load Times](https://web.dev/fast/)
- [Webpack: Code Splitting](https://webpack.js.org/guides/code-splitting/)

## Backend Performance Optimization

### Definition
Backend performance optimization targets database query speeds, memory footfalls, network latency, and concurrency. Strategies include index configuration, connection pooling, payload compression, caching, and CPU/IO load balancing.

### Real-world Analogy
Think of a warehouse shipping company. Instead of the picker driving across the warehouse to look for one envelope on every single order (Seq Scan), they construct a card index (Database Index), print smaller package receipts (Compression), and keep popular products on a desk next to the dispatch door (Redis cache).

### Code Example
```javascript
// Express application gzip/brotli compression middleware configuration
const compression = require("compression");
const express = require("express");
const app = express();

// Compresses all outgoing responses automatically
app.use(compression());
```

### Common Interview Questions
- How do you diagnose memory leaks in a production Node.js application using heap dumps?
- What is the difference between CPU-bound bottlenecks and I/O-bound bottlenecks?
- Explain how database connection pooling optimizes backend systems.

### Reference Links
- [Node.js: Memory Leaks Profile](https://nodejs.org/en/learn/diagnosing-page-errors/debugging-memory-leaks)
- [Express: Performance Best Practices](https://expressjs.com/en/advanced/best-practice-performance.html)

## Caching Strategies

### Definition
Caching stores data copies in temporary storage layers. Strategies include HTTP cache headers (Cache-Control, ETag), CDN caching, and Redis caching patterns (Cache-Aside, Write-Through, Write-Back) with validation routines (TTL, active eviction).

### Real-world Analogy
Cache-Aside is like keeping a telephone book on your desk. When you need a number, you look on your desk first (Cache-Aside check). If the number is in the book, you call it (Cache hit). If not, you walk to the main archive room (database read), write the number down in your desk book, and return to call.

### Code Example
```javascript
// Cache-Aside Pattern with Redis
async function getUserData(userId) {
  const cacheKey = `user:${userId}`;
  const cachedData = await redis.get(cacheKey);
  
  if (cachedData) {
    return JSON.parse(cachedData); // Cache Hit
  }
  
  const dbUser = await db.users.findById(userId); // Cache Miss
  if (dbUser) {
    await redis.setex(cacheKey, 3600, JSON.stringify(dbUser)); // Cache entry with 1 hr TTL
  }
  return dbUser;
}
```

### Common Interview Questions
- What is a Cache Stampede (Thundering Herd) and how do you mitigate it?
- Compare the Cache-Aside and Write-Through caching patterns.
- How do ETag headers and 304 Not Modified status codes optimize asset delivery?

### Reference Links
- [Redis Caching Patterns](https://redis.io/solutions/caching/)
- [MDN Web Docs: HTTP Caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)

## Image and Media Optimization

### Definition
Image and Media optimization reduces visual asset payloads using modern web formats (WebP, AVIF), responsive breakpoints (srcset), lazy loading attributes, and content delivery networks.

### Real-world Analogy
Imagine a gallery displaying artwork. Instead of framing a massive billboard-sized painting and hanging it on a tiny hallway wall, the curator mounts a postcard-sized copy of the painting for walk-by visitors, letting them order the full canvas only if they buy a frame.

### Code Example
```html
<!-- Responsive image implementation with srcset and lazy loading -->
<img 
  src="photo-600.jpg"
  srcset="photo-300.jpg 300w, photo-600.jpg 600w, photo-1200.jpg 1200w"
  sizes="(max-width: 600px) 300px, 600px"
  loading="lazy"
  alt="Optimized landscape"
/>
```

### Common Interview Questions
- Why are WebP and AVIF formats preferred over JPEG/PNG?
- How does the loading="lazy" attribute optimize image loading in the browser?
- Explain the role of Image CDNs (like Cloudinary or Imgix).

### Reference Links
- [web.dev: Image Optimization](https://web.dev/optimize-images/)

## CSS and JS Delivery

### Definition
CSS and JS delivery optimization removes render-blocking paths. Techniques include inline extraction of Critical CSS, script loading flags (defer/async), font-display rules, and code splitting configurations.

### Real-world Analogy
Imagine a theatrical play. Instead of making the audience sit in total darkness for 20 minutes while actors assemble the entire set scenery (render-blocking), you start the play on a bare stage with two spotlight actors (Critical rendering), and roll the background trees out while the play runs.

### Code Example
```html
<!-- Inline Critical CSS and Defer external JavaScript -->
<head>
  <style>
    /* Critical CSS: Styles required to render above-the-fold content */
    body { font-family: sans-serif; background: #fff; }
    h1 { color: #333; }
  </style>
  <script src="bundle.js" defer></script>
</head>
```

### Common Interview Questions
- How does the font-display: swap rule prevent the Flash of Invisible Text (FOIT) anomaly?
- Explain the difference between render-blocking assets and parser-blocking assets.
- Describe how to optimize script delivery using code splitting.

### Reference Links
- [web.dev: Eliminate Render-Blocking Resources](https://web.dev/render-blocking-resources/)

## Web Vitals

### Definition
Web Vitals are quality signals key to delivering great user experiences. Core Web Vitals measure speed (Largest Contentful Paint - LCP), responsiveness (Interaction to Next Paint - INP), and visual stability (Cumulative Layout Shift - CLS).

### Real-world Analogy
LCP is how long it takes for the headline newspaper to load on your screen (main page paint). INP is how fast the page reacts if you tap a button. CLS is if the text blocks jump down on the page while you are reading because an ad banner loaded late.

### Code Example
```javascript
// Reporting Core Web Vitals using web-vitals library
import { onLCP, onINP, onCLS } from "web-vitals";

onLCP(console.log);
onINP(console.log);
onCLS(console.log);
```

### Common Interview Questions
- What are the target threshold values for LCP, INP, and CLS to pass Core Web Vitals?
- How does assigning explicit dimensions (width/height) to images prevent layout shifts (CLS)?
- What is First Contentful Paint (FCP) and how does it relate to LCP?

### Reference Links
- [web.dev: Core Web Vitals](https://web.dev/vitals/)

## Node.js Event Loop Monitoring

### Definition
Event loop monitoring measures system delay and blocks inside Node.js. Tools (like Clinic.js, Chrome DevTools profiling) check event loop delay, garbage collection cycles, and blockages.

### Real-world Analogy
Think of monitoring a traffic intersection. If cars wait at the light for 1 millisecond (low loop delay), traffic flows smoothly. If a massive slow truck blocks the middle of the road for 5 seconds (event loop blockage), the entire line of cars behind it piles up.

### Code Example
```javascript
// Measuring event loop lag manually
let lastTime = Date.now();

setInterval(() => {
  const currentTime = Date.now();
  const lag = currentTime - lastTime - 100; // Expected delay is 100ms
  console.log(`Event loop lag: ${lag}ms`);
  lastTime = currentTime;
}, 100);
```

### Common Interview Questions
- Explain how garbage collection runs can cause event loop lag spikes.
- What profiling tools can you use to locate CPU hot spots in a Node.js process?
- How do you read V8 flame graphs?

### Reference Links
- [Node.js Docs: Event Loop Lag](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop)

## Network-Level Optimization

### Definition
Network-level optimization optimizes data packaging and transport layers. It uses HTTP/2 multiplexing, HTTP/3 (QUIC protocols), Brotli/Gzip compression algorithms, and preconnect connections.

### Real-world Analogy
HTTP/1.1 is like carrying boxes over a bridge one bike at a time: you must wait for the bike to return. HTTP/2 is a single high-speed train carriage containing 20 separate drawers that transport all boxes together. Brotli is vacuum-packing clothes to fit 5x more items inside the carriage.

### Code Example
```html
<!-- DNS Prefetching and Preconnecting to trusted third party domains -->
<head>
  <link rel="dns-prefetch" href="https://api.example.com" />
  <link rel="preconnect" href="https://api.example.com" crossorigin />
</head>
```

### Common Interview Questions
- What is the difference between Gzip and Brotli compression algorithms?
- How does HTTP/2 multiplexing solve the Head-of-Line (HoL) blocking limitation?
- What are DNS-prefetch and preconnect attributes?

### Reference Links
- [MDN Web Docs: Link Prefetching](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/prefetch)

## Scenario-based Interview Questions for Performance

### Scenario 1
A product page has poor LCP metrics. The hero image loads 4 seconds after the page loads. When it loads, the layout shifts, causing a bad CLS rating. How do you resolve these issues?

*Expected Approach:*
1. To fix LCP: Preload the hero image using `<link rel="preload" as="image">` or set the fetchpriority="high" attribute. Ensure the image is optimized using WebP/AVIF format and responsive size paths.
2. To fix CLS: Assign explicit width and height dimensions to the HTML `<img>` tag or use CSS aspect-ratio configurations, reserving display space for the image before it loads.

### Scenario 2
Your backend service occasionally suffers from database spikes. During peek sales, the connection pool runs out of resources, throwing "Too many connections" errors and crashing the Express API. How do you optimize the backend path?

*Expected Approach:*
1. Implement a Cache-Aside pattern using Redis to cache product details, reducing database query frequency.
2. Configure database connection pool parameters (e.g. set maxPoolSize to a safe limit matching database capacity) and configure pgBouncer poolers if deploying in serverless systems.
3. Apply API rate limiting to block scrapers, and configure load balancing to distribute connection load across multiple application nodes.
