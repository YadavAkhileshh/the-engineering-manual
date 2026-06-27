# 02 Network and Browser Internals

This section covers how the internet works, how browsers parse and display content, and how node developers manage packages and dependencies.

## Network Fundamentals

### DNS Resolution

#### Definition
Domain Name System (DNS) resolution is the process of translating a human-readable domain name (like google.com) into a machine-readable IP address. This is achieved via recursive resolvers, root name servers, TLD name servers, and authoritative name servers.

#### Real-world Analogy
Think of the contacts app on your phone. You do not remember the 10-digit phone number of every friend; you just click their name. The phone looks up the name in its memory (DNS cache) or queries the service provider directory to find the actual number (IP address).

#### Code Example
```bash
# Resolve DNS records using the nslookup command
nslookup google.com

# Fetch specific DNS records like A or CNAME
nslookup -type=ns github.com
```

#### Common Interview Questions
- Explain the difference between recursive and iterative DNS queries.
- What is the difference between an A record, a CNAME, and an MX record?
- How does local DNS caching on browsers and operating systems improve performance?

#### Reference Links
- [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/)
- [MDN Web Docs: DNS](https://developer.mozilla.org/en-US/docs/Glossary/DNS)

### TCP vs UDP

#### Definition
Transmission Control Protocol (TCP) is a connection-oriented protocol that ensures reliable, ordered, and error-checked delivery of packets via a three-way handshake. User Datagram Protocol (UDP) is a connectionless protocol that sends packets instantly without checking if they arrive.

#### Real-world Analogy
TCP is like sending a registered letter through the post office: you get a receipt, the mail carrier verifies the recipient, and you receive confirmation. UDP is like throwing flyers into the crowd: it is fast, cheap, and you do not care if some fly away in the wind.

#### Code Example
```javascript
// Minimal Node.js TCP Server using net module
const net = require("net");

const server = net.createServer((socket) => {
  socket.write("Hello TCP Client!\n");
  socket.pipe(socket);
});

server.listen(1337, "127.0.0.1");
```

#### Common Interview Questions
- Detail the steps of the TCP three-way handshake (SYN, SYN-ACK, ACK).
- Why does HTTP/1.1 and HTTP/2 rely on TCP, and why does HTTP/3 transition to UDP?
- In what scenarios would you choose UDP over TCP?

#### Reference Links
- [RFC 9293: Transmission Control Protocol](https://datatracker.ietf.org/doc/html/rfc9293)
- [MDN Web Docs: TCP](https://developer.mozilla.org/en-US/docs/Glossary/TCP)

### HTTP/1.1 vs HTTP/2 vs HTTP/3

#### Definition
HTTP protocols govern data exchange on the web. HTTP/1.1 uses persistent TCP connections but suffers from head-of-line blocking. HTTP/2 introduces multiplexing over a single TCP connection with binary framing. HTTP/3 replaces TCP with QUIC (built on UDP) to eliminate connection-level blocking.

#### Real-world Analogy
HTTP/1.1 is a supermarket checkout with one lane; if one customer has a slow lookup, everyone behind them waits. HTTP/2 is a single wide checkout lane where the cashier scans items from multiple baskets at the same time. HTTP/3 is a system of multiple delivery runners delivering individual small boxes directly to customer cars.

#### Code Example
```javascript
// Checking the protocol version of a response in native Node.js
const http2 = require("http2");

const client = http2.connect("https://www.google.com");
client.on("error", (err) => console.error(err));

const req = client.request({ ":path": "/" });
req.on("response", (headers) => {
  console.log("HTTP version matches HTTP/2 specs");
  client.close();
});
req.end();
```

#### Common Interview Questions
- What is HTTP/2 head-of-line blocking and how does HTTP/3 resolve it using QUIC?
- Explain the role of HPACK in HTTP/2 header compression.
- What are server push and multiplexing in HTTP/2?

#### Reference Links
- [HTTP Working Group: HTTP/2 Specification](https://httpwg.org/specs/rfc7540.html)
- [Cloudflare: What is HTTP/3?](https://www.cloudflare.com/learning/performance/what-is-http3/)

### HTTPS and TLS

#### Definition
HTTPS is HTTP wrapped in a Transport Layer Security (TLS) encryption layer. It secures client-server communication using asymmetric encryption for the initial handshake and symmetric encryption for actual data transfer.

#### Real-world Analogy
Imagine sending someone a metal lockbox. You send the open lockbox to them (public key). They put their letter inside, lock it shut, and send it back to you. Only you can open it because you hold the physical key (private key). Once opened, you both agree to use a simpler secret code (symmetric key) for quick chats.

#### Code Example
```javascript
// HTTPS client request in Node.js
const https = require("https");

https.get("https://encrypted.google.com/", (res) => {
  console.log("Status Code: " + res.statusCode);
}).on("error", (e) => {
  console.error(e);
});
```

#### Common Interview Questions
- Explain the difference between symmetric and asymmetric encryption in HTTPS.
- Describe the steps of a TLS handshake.
- What is a Certificate Authority (CA) and how does a browser verify SSL certificates?

#### Reference Links
- [MDN Web Docs: HTTPS](https://developer.mozilla.org/en-US/docs/Glossary/HTTPS)
- [Cloudflare: What is a TLS handshake?](https://www.cloudflare.com/learning/ssl/what-happens-in-a-tls-handshake/)

### CORS Underlying Mechanics

#### Definition
Cross-Origin Resource Sharing (CORS) is a browser security mechanism that uses HTTP headers to restrict cross-origin requests. Browsers send an HTTP OPTIONS request (preflight) before unsafe requests to check if the server permits the operation.

#### Real-world Analogy
Imagine a security guard standing at the door of a private club. Before you walk inside carrying tools, the guard stops you and makes a phone call to the owner (preflight OPTIONS request). The owner gives the OK, and only then are you allowed inside to perform your work.

#### Code Example
```javascript
// Raw preflight response headers from an Express Server
const express = require("express");
const app = express();

app.options("/api/data", (req, res) => {
  res.setHeader("Access-Control-Allow-Origin", "https://trustedclient.com");
  res.setHeader("Access-Control-Allow-Methods", "GET, POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");
  res.sendStatus(204);
});
```

#### Common Interview Questions
- What triggers a CORS preflight request, and what HTTP method does it use?
- How does the Access-Control-Allow-Credentials header affect origin wildcard options?
- Why does CORS protection not block curl requests or server-to-server calls?

#### Reference Links
- [MDN Web Docs: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [HTML Standard: CORS settings attributes](https://html.spec.whatwg.org/multipage/urls-and-fetching.html#cors-settings-attributes)

## Browser Internals

### Critical Rendering Path

#### Definition
The Critical Rendering Path (CRP) is the sequence of steps the browser takes to convert HTML, CSS, and JavaScript code into actual pixels on the screen. It moves through parsing HTML (DOM), parsing CSS (CSSOM), building the Render Tree, Layout calculation, Painting, and Compositing.

#### Real-world Analogy
Think of building a house. You read the blueprint (HTML to DOM), look at the decorator style guide (CSS to CSSOM), choose only the active designs (Render Tree), measure the exact coordinates of the rooms (Layout), paint the walls (Paint), and layer the roof tiles (Composite).

#### Code Example
```html
<!-- Optimizing CRP by using async or defer tags -->
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="styles.css">
  <script src="non-blocking.js" defer></script>
</head>
<body>
  <h1>Hello CRP Optimization</h1>
</body>
</html>
```

#### Common Interview Questions
- What is the difference between parsing javascript with async versus defer tags?
- How do stylesheets block DOM parsing and rendering?
- What are layout, paint, and composite phases, and which is the most expensive?

#### Reference Links
- [MDN Web Docs: Critical rendering path](https://developer.mozilla.org/en-US/docs/Web/Performance/Critical_rendering_path)
- [web.dev: Critical rendering path](https://web.dev/critical-rendering-path/)

### Reflow vs Repaint

#### Definition
Reflow (or Layout) is the browser process of recalculating the positions and geometries of elements on the page. Repaint is the process of drawing the visual pixels of elements (colors, shadows) onto the screen without changing their layout or structure.

#### Real-world Analogy
Reflow is like moving a couch to the opposite wall in your room: you have to rearrange the coffee table, chairs, and rugs to fit. Repaint is like throwing a blue throw blanket over the same couch: the position remains unchanged, but the visual surface changes.

#### Code Example
```javascript
const element = document.getElementById("box");

// This triggers a Reflow (changing layout geometry)
element.style.width = "200px";

// This triggers a Repaint only (changing visual style)
element.style.backgroundColor = "blue";
```

#### Common Interview Questions
- Mention three specific CSS properties that trigger reflow when modified.
- How do you use requestAnimationFrame to minimize layout thrashing?
- Why are CSS transforms (translate, scale) handled on the GPU instead of triggering reflow?

#### Reference Links
- [MDN Web Docs: Layout](https://developer.mozilla.org/en-US/docs/Glossary/Layout)
- [web.dev: Avoid large, complex layouts and layout thrashing](https://web.dev/avoid-large-complex-layouts-and-layout-thrashing/)

### Browser Garbage Collection

#### Definition
Browsers manage engine memory using a combination of the Mark-and-Sweep algorithm and Generational Garbage Collection. Generational GC divides objects into "young" (short-lived) and "old" (long-lived) spaces, running quick collections on the young space to keep performance high.

#### Real-world Analogy
Think of cleaning a house. You empty the small compost bin (young generation) every day because it fills up fast. You only sort through the basement storage boxes (old generation) once a year because things there are expected to stay long-term.

#### Code Example
```javascript
// Memory allocation pattern
function runAnimationLoop() {
  // Objects created inside the loop are young generation
  let frameData = { time: Date.now() };
  requestAnimationFrame(runAnimationLoop);
}
```

#### Common Interview Questions
- What is the difference between a major GC cycle and a minor GC cycle in V8?
- How does the browser determine if an object in the heap is reachable?
- Why is garbage collection in browsers single-threaded and how does it block rendering?

#### Reference Links
- [V8: Garbage collection](https://v8.dev/blog/trash-talk)
- [MDN Web Docs: Garbage Collection](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management#garbage_collection)

### V8 Engine Basics

#### Definition
V8 is Google's open-source high-performance JavaScript and WebAssembly engine. It compiles JavaScript code directly into native machine code using an interpreter (Ignition) and an optimizing compiler (TurboFan), employing "hidden classes" (shapes) to optimize object property access.

#### Real-world Analogy
Imagine a translation team. One translator translates the speech live line-by-line (Ignition interpreter). Meanwhile, a senior translator notes down repetitive phrases and writes down a polished, super-fast pre-written script (TurboFan compiler) to swap in whenever those phrases occur.

#### Code Example
```javascript
// Monomorphic vs Polymorphic V8 optimization example
function getArea(rect) {
  // V8 expects rect to share the same hidden class (shape)
  return rect.w * rect.h;
}

const r1 = { w: 10, h: 20 }; // Hidden Class A
const r2 = { w: 15, h: 25 }; // Hidden Class A
getArea(r1);
getArea(r2); // Fast monomorphic call path
```

#### Common Interview Questions
- Explain the role of the Ignition interpreter and the TurboFan optimizing compiler in V8.
- What are hidden classes (or shapes) in V8 and how do they speed up property access?
- Why is it recommended to initialize object properties in the constructor in the exact same order?

#### Reference Links
- [V8 Official Docs](https://v8.dev/)
- [V8: Hidden classes](https://v8.dev/blog/fast-properties)

## Package Management and Tooling

### package.json Deep Dive

#### Definition
The package.json file is the manifest of a Node.js project. It contains configuration metadata, scripts, and splits dependencies into dependencies (runtime), devDependencies (build/test time), and peerDependencies (required plugin dependencies).

#### Real-world Analogy
Think of a cake mix box. The front list shows the brand and name (metadata). The side panel lists ingredients that come in the box (dependencies), tools you need in your kitchen like a whisk to make it (devDependencies), and external ingredients you must supply like eggs (peerDependencies).

#### Code Example
```json
{
  "name": "my-web-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.19.2"
  },
  "devDependencies": {
    "typescript": "^5.4.5"
  },
  "peerDependencies": {
    "react": "^18.3.1"
  }
}
```

#### Common Interview Questions
- What is the difference between devDependencies, dependencies, and peerDependencies?
- What do the caret (^) and tilde (~) symbols mean in package version specifications?
- What are the engines and bin fields used for in package.json?

#### Reference Links
- [npm Docs: package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json)
- [Node.js: package.json handling](https://nodejs.org/api/packages.html)

### npm vs yarn vs pnpm

#### Definition
These are package managers for Node.js. npm is the default manager; yarn introduced lock files and parallel installations; pnpm optimizes disk space and speed using a content-addressable storage model where files are hard-linked to a global store, avoiding duplicate node_modules.

#### Real-world Analogy
npm is a grocery shopper who drives to the store for every single recipe, buying duplicate bags of flour. yarn is a shopper who prints a shopping checklist. pnpm is a professional warehouse: they buy one massive bag of flour and use teleporters to link that same flour to every recipe kitchen instantly.

#### Code Example
```bash
# Install dependencies using pnpm
pnpm install

# Run a project script
pnpm run dev
```

#### Common Interview Questions
- How does pnpm's content-addressable store prevent duplicate package installations on your disk?
- Explain the concept of phantom dependencies and how pnpm prevents them.
- Compare pnpm and npm installation times and disk space savings.

#### Reference Links
- [pnpm Docs: Motivation](https://pnpm.io/motivation)
- [Yarn Docs](https://yarnpkg.com/)

### Lock Files

#### Definition
Lock files (package-lock.json, yarn.lock, pnpm-lock.yaml) lock down the exact version of every dependency and sub-dependency installed in node_modules, ensuring consistent installations across different machines and environments.

#### Real-world Analogy
Imagine a blueprint that lists the exact serial numbers of every steel girder and rivet used to build a bridge. This ensures that when a copy of the bridge is built in another town, it uses the exact same parts, preventing structural failure due to part changes.

#### Code Example
```bash
# Installs packages strictly matching the lockfile
npm ci

# Updates dependencies and generates a new package-lock.json
npm install
```

#### Common Interview Questions
- Why must you always commit package-lock.json or yarn.lock to version control?
- What is the difference between npm install and npm ci?
- How do you resolve a git conflict in package-lock.json?

#### Reference Links
- [npm Docs: package-lock.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-lock-json)
- [npm Docs: npm-ci](https://docs.npmjs.com/cli/v10/commands/npm-ci)

### npx vs npm exec

#### Definition
npx (or npm exec) is a CLI tool that executes binaries from local node_modules or downloads and runs a package temporarily from the registry without installing it globally on the system.

#### Real-world Analogy
npx is like renting a pressure washer for one afternoon to clean your driveway. You do not want to buy it, store it in your garage, and maintain it forever; you just use it once and return it.

#### Code Example
```bash
# Execute create-vite without installing it globally
npx create-vite@latest my-app --template react-ts
```

#### Common Interview Questions
- What is the difference between npm run command and npx command?
- Where does npx store the temporary packages it downloads for execution?
- Can you use npx to run custom scripts defined in package.json?

#### Reference Links
- [npm Docs: npx](https://docs.npmjs.com/cli/v10/commands/npx)
- [npm Docs: npm-exec](https://docs.npmjs.com/cli/v10/commands/npm-exec)

### Node.js Module Resolution Algorithm

#### Definition
The Node.js module resolution algorithm determines the file path Node looks up when a module is imported or required. It checks built-in modules first, then files relative to the current path, and finally traverses up the folder tree searching node_modules folders.

#### Real-world Analogy
Imagine looking for a physical file folder in an office. First, you check your desk drawers (local files). If you do not find it, you check the cabinet outside your room. If it is still missing, you go up floor-by-floor checking the main storage room of the building.

#### Code Example
```javascript
// Resolution lookup order for: const express = require('express')
// 1. Core modules check (e.g. 'express' is not core)
// 2. Looks in current directory node_modules/express
// 3. Traverses up parent directories: ../node_modules/express, etc.
const express = require("express");
```

#### Common Interview Questions
- Describe the directory lookup sequence Node performs when you run require('my-package').
- How does the 'exports' field in package.json change modern ESM module resolution?
- Why do absolute path mappings or custom aliases improve module resolution times?

#### Reference Links
- [Node.js Documentation: Module Resolution](https://nodejs.org/api/modules.html#modules_all_together)
- [TypeScript Docs: Module Resolution](https://www.typescriptlang.org/docs/handbook/module-resolution.html)

## Scenario-based Interview Questions for Network & Browser Internals

### Scenario 1
A frontend application makes an API request to api.example.com. The browser console shows: "Access to XMLHttpRequest at 'api.example.com' from origin 'example.com' has been blocked by CORS policy". The backend engineer claims, "I tested the API with curl and it returns status 200". What is the problem and how do you fix it?

*Expected Approach:*
1. Explain that curl is not a browser; it does not enforce the Same-Origin Policy or run preflight checks.
2. The browser is blocking the request because the server response does not contain the Access-Control-Allow-Origin header matching the client's origin.
3. Fix the issue by configuring the backend server to return appropriate CORS headers (Access-Control-Allow-Origin, Access-Control-Allow-Methods, Access-Control-Allow-Headers) in response to the browser's requests, including OPTIONS.

### Scenario 2
Your client-side web app features a page with a table containing 5,000 rows. When users type into a search input to filter rows, the page freezes for a second on every keystroke. How do you identify the cause and solve it?

*Expected Approach:*
1. State that the frequent layout updates and DOM insertions trigger massive Reflows and Repaints on every keypress.
2. Use Chrome DevTools Performance panel to record typing and identify long-running Javascript tasks and layout recalculation blocks.
3. Solve it by implementing input Debouncing or Throttling, virtualizing the list (rendering only visible rows using react-window), or performing computations off the main thread.
