# 01 Javascript Fundamentals

Welcome to the JavaScript foundation brush-up. Here, we cover the core behavior of the language that interviewers love to test.

## Execution Context and Call Stack

### Definition
An execution context is an environment in which JavaScript code is evaluated and executed. The call stack is a LIFO (Last In, First Out) stack structure used by the JavaScript engine to keep track of function execution order.

### Real-world Analogy
Imagine a stack of dinner plates. You add a new plate to the top when a new task arrives (calling a function) and remove the top plate when that task is finished (returning from a function). The plate on the very top is the context you are actively working on.

### Code Example
```javascript
function greet(name) {
  return "Hello " + name;
}

function welcomeUser() {
  const message = greet("Alice");
  console.log(message);
}

welcomeUser();
```

### Common Interview Questions
- What is the difference between the Global Execution Context and a Function Execution Context?
- How does the JavaScript engine handle stack overflow errors?
- What happens to variables inside an execution context when the function finishes running?

### Reference Links
- [MDN Web Docs: Call Stack](https://developer.mozilla.org/en-US/docs/Glossary/Call_Stack)
- [ECMA-262 Specification: Execution Contexts](https://tc39.es/ecma262/#sec-execution-contexts)

## Hoisting

### Definition
Hoisting is the JavaScript engine's behavior of moving variable, function, and class declarations to the top of their containing scope before code execution. While function declarations are fully hoisted, variables declared with let and const are placed in a Temporal Dead Zone (TDZ) and cannot be accessed before their declaration.

### Real-world Analogy
Think of organizing a theater production. Before the actors start speaking their lines (executing code), the stage manager lists all props and cast members (declarations) on the whiteboard. However, you are not allowed to use a prop until it has been officially brought to the stage (the line of code where the variable is assigned).

### Code Example
```javascript
console.log(greet());

function greet() {
  return "Hi there!";
}

try {
  console.log(secretVar);
} catch (e) {
  console.log("Error: " + e.message); // Cannot access 'secretVar' before initialization
}

let secretVar = "hidden";
```

### Common Interview Questions
- What is the Temporal Dead Zone (TDZ) and why does it exist?
- Compare hoisting behavior between var, let, const, and function declarations.
- What happens if you define a function using a function expression (e.g., const myFunc = () => {})? Is it hoisted?

### Reference Links
- [MDN Web Docs: Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
- [MDN Web Docs: let (Temporal Dead Zone)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)

## Scope Chain and Lexical Scope

### Definition
Lexical scope means that variable access is determined by the physical placement of variables and blocks in the source code at compile time. The scope chain is the path of nested environments that the engine traverses to resolve a variable's value, starting from the current inner scope up to the global scope.

### Real-world Analogy
Imagine a set of Russian nesting dolls. The doll on the inside can look out at all the outer nesting dolls enclosing it and read their labels. However, the outer dolls cannot look inside to see what is written on the inner dolls.

### Code Example
```javascript
const globalVal = "global";

function outer() {
  const outerVal = "outer";
  
  function inner() {
    const innerVal = "inner";
    console.log(innerVal + " " + outerVal + " " + globalVal);
  }
  
  inner();
}

outer();
```

### Common Interview Questions
- What is the difference between lexical scope and dynamic scope?
- How does the scope chain resolution work when two variables have the same name in nesting scopes?
- How do block-scoped variables (let/const) differ from function-scoped variables (var) in the scope chain?

### Reference Links
- [MDN Web Docs: Scope](https://developer.mozilla.org/en-US/docs/Glossary/Scope)
- [MDN Web Docs: Lexical scoping](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#lexical_scoping)

## Closures

### Definition
A closure is the combination of a function bundled together with references to its surrounding state (the lexical environment). In other words, a closure gives an inner function access to the outer function's scope even after the outer function has returned.

### Real-world Analogy
Imagine a backpack that holds items you gathered while visiting a park. When you leave the park (the outer function returns) and return home, you still have access to the items inside your backpack (variables in the outer scope) because they are physically bound to you.

### Code Example
```javascript
// Data privacy example
function createCounter() {
  let count = 0;
  return {
    increment() {
      count++;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.getCount()); // 1
```

### Common Interview Questions
- How do you use closures to implement the Module Pattern for data privacy?
- What is currying and how does it rely on closures?
- What are the memory implications of creating nested closures (e.g., potential memory leaks)?

### Reference Links
- [MDN Web Docs: Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)
- [JavaScript.info: Closure](https://javascript.info/closure)

## The this Keyword

### Definition
The value of the 'this' keyword in JavaScript is determined dynamically based on how a function is called, rather than how it is defined. In global context, it refers to the global object; in method execution, it refers to the owner object; in arrow functions, it retains the 'this' of its enclosing lexical context.

### Real-world Analogy
Think of the word "here". If you say "here" while standing inside your house, you mean your living room. If you say "here" while standing in a coffee shop, you mean the coffee shop. The meaning of "here" changes depending on where you are when you speak it.

### Code Example
```javascript
const person = {
  name: "Bob",
  greetNormal() {
    return "Hi, " + this.name;
  },
  greetArrow: () => {
    return "Hi, " + this.name; // 'this' is bound to global context or undefined
  }
};

console.log(person.greetNormal()); // Hi, Bob
const unboundGreet = person.greetNormal;
console.log(unboundGreet.call({ name: "Charlie" })); // Hi, Charlie
```

### Common Interview Questions
- Explain the difference in 'this' binding between arrow functions and regular functions.
- What do the call, apply, and bind methods do, and how do they differ?
- What does the 'this' keyword point to inside an HTML event listener?

### Reference Links
- [MDN Web Docs: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
- [MDN Web Docs: bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

## Prototypal Inheritance

### Definition
Prototypal inheritance is a language feature where objects inherit properties and methods directly from other objects via a prototype link. Every object has an internal link (represented by [[Prototype]]) to another object, forming a prototype chain that ends at null.

### Real-world Analogy
Think of a family recipe book passed down through generations. If you want to make a dish, you first check your personal notes. If the recipe is not there, you check your parents' recipe book. If it is not there, you check your grandparents' book. You inherit all their recipes unless you write your own version.

### Code Example
```javascript
const animal = {
  eats: true,
  walk() {
    return "Walking...";
  }
};

const rabbit = Object.create(animal);
rabbit.jumps = true;

console.log(rabbit.eats); // true (inherited)
console.log(rabbit.walk()); // Walking... (inherited)
console.log(rabbit.hasOwnProperty("jumps")); // true
console.log(rabbit.hasOwnProperty("eats")); // false
```

### Common Interview Questions
- How does the prototype chain work when resolving a property access?
- What is the difference between the prototype property of a constructor function and the [[Prototype]] of an instance?
- Explain how ES6 classes are just syntactic sugar over prototypal inheritance.

### Reference Links
- [MDN Web Docs: Inheritance and the prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
- [JavaScript.info: Prototypal inheritance](https://javascript.info/prototypes)

## The Event Loop

### Definition
The event loop is a runtime mechanism that coordinates code execution, events, and sub-tasks in JavaScript. It executes synchronous code on the call stack first, then empties the microtask queue (Promises), and finally executes macrotasks (setTimeout, event listeners) one by one.

### Real-world Analogy
Think of an amusement park ride coordinator. The coordinator lets people in the main queue (call stack) board first. If anyone has a fast-pass ticket (microtask queue), they get served immediately after the main queue is clear. Regular ticket holders waiting in lines outside the ride (macrotask queue) are let in one group at a time between runs.

### Code Example
```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout (Macrotask)");
}, 0);

Promise.resolve().then(() => {
  console.log("Promise (Microtask)");
});

console.log("End");
// Output order: Start -> End -> Promise -> Timeout
```

### Common Interview Questions
- Explain the execution priority between the Call Stack, Microtask Queue, and Macrotask Queue.
- What is the purpose of requestAnimationFrame and when does it run relative to style updates and layout?
- Why can a long-running synchronous loop block the UI, and how do you resolve it?

### Reference Links
- [MDN Web Docs: Concurrency model and the Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Event_loop)
- [HTML Standard: Event loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

## Promises

### Definition
A Promise is an object representing the eventual completion or failure of an asynchronous operation and its resulting value. It exists in one of three states: pending (initial state), fulfilled (operation completed successfully), or rejected (operation failed).

### Real-world Analogy
You order food at a restaurant counter, and they hand you a buzzer. The buzzer represents a promise. It is pending while your food is being cooked. It will flash red and buzz when your food is ready (fulfilled), or beep differently if they run out of ingredients (rejected).

### Code Example
```javascript
const fetchData = () => {
  return new Promise((resolve, reject) => {
    const success = true;
    if (success) {
      resolve({ data: "API payload" });
    } else {
      reject("Network timeout");
    }
  });
};

fetchData()
  .then(res => console.log(res.data))
  .catch(err => console.error(err));
```

### Common Interview Questions
- How does Promise.all differ from Promise.allSettled, Promise.race, and Promise.any?
- What is promise chaining and how do you return values/promises from a .then() handler?
- How do you handle unhandled promise rejections in a production application?

### Reference Links
- [MDN Web Docs: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN Web Docs: Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

## Async / Await

### Definition
Async/await is a syntactic feature built on top of Promises that allows you to write asynchronous code that reads like synchronous code. Functions marked with 'async' always return a Promise, and the 'await' keyword pauses execution until the promise settles.

### Real-world Analogy
Imagine writing a letter to a colleague and waiting next to their desk for their written response before you proceed to the next step of your work, rather than putting the letter in the mailbox and registering a callback reminder to check for mail later.

### Code Example
```javascript
async function getDashboardData() {
  try {
    const userPromise = Promise.resolve({ id: 1, name: "Alice" });
    const statsPromise = Promise.resolve({ logins: 42 });
    
    // Parallel execution using Promise.all
    const [user, stats] = await Promise.all([userPromise, statsPromise]);
    console.log(user.name + " has " + stats.logins + " logins.");
  } catch (error) {
    console.error("Failed to load dashboard: ", error);
  }
}

getDashboardData();
```

### Common Interview Questions
- What happens under the hood when JavaScript encounters an 'await' statement?
- Explain the performance difference between executing multiple async calls sequentially versus in parallel.
- What is an async iterator and how does the for-await-of loop work?

### Reference Links
- [MDN Web Docs: async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN Web Docs: await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)

## Coercion and Equality

### Definition
Coercion is the automatic or implicit conversion of values from one data type to another during operations. JavaScript provides loose equality (==) which performs type coercion before comparison, and strict equality (===) which requires both value and type to be identical.

### Real-world Analogy
Imagine a cash register that accepts both coins and paper vouchers. If you use a loose cashier (==), they will count a $5 paper voucher as equal to a $5 bill. If you use a strict cashier (===), they will refuse the voucher because it is a different physical medium than cash.

### Code Example
```javascript
console.log(5 == "5"); // true (coerced)
console.log(5 === "5"); // false (strict)
console.log([] == false); // true (complex coercion rules)

// Object.is compares strict values without negative/positive zero and NaN quirks
console.log(Object.is(NaN, NaN)); // true
console.log(NaN === NaN); // false
```

### Common Interview Questions
- Why is NaN === NaN false, and how does Object.is resolve this?
- Explain the step-by-step coercion rules when comparing an object and a primitive using ==.
- List at least five falsy values in JavaScript.

### Reference Links
- [MDN Web Docs: Type coercion](https://developer.mozilla.org/en-US/docs/Glossary/Type_coercion)
- [MDN Web Docs: Equality comparisons and sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)

## ES6+ Features

### Definition
ES6+ refers to modern ECMAScript standard updates introduced from 2015 onward, which added features like destructuring, rest/spread operators, template literals, optional chaining, nullish coalescing, symbols, generators, and new array helper methods.

### Real-world Analogy
Think of upgrading a standard kitchen knife block to a multi-functional Swiss Army knife. The new tool does the same cooking job but adds specific shortcuts like bottle openers and built-in scissors that save you from executing long, manual processes.

### Code Example
```javascript
const user = { name: "Alice", details: { age: 30 } };

// Optional chaining & Nullish coalescing
const location = user?.details?.location ?? "Unknown";

// Spread operator and destructuring
const updatedUser = { ...user, active: true };
const { name } = updatedUser;

// Array group-by (ES2024 example)
const inventory = [
  { name: "apples", quantity: 5 },
  { name: "bananas", quantity: 0 }
];
// const grouped = Object.groupBy(inventory, item => item.quantity > 0 ? "inStock" : "outOfStock");
```

### Common Interview Questions
- What is the difference between the spread (...) operator and the rest (...) parameter?
- How does the nullish coalescing operator (??) differ from the logical OR (||) operator?
- What are generators and how do they maintain state across multiple yield statements?

### Reference Links
- [MDN Web Docs: Modern JavaScript features](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference)
- [MDN Web Docs: Optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining)

## Garbage Collection Basics

### Definition
Garbage collection is an automatic memory management process in JavaScript that monitors allocated memory and frees up memory blocks that are no longer referenced or reachable from the root execution contexts. The primary algorithm used is "Mark-and-Sweep".

### Real-world Analogy
Think of a trash collection service. Every night, the collector starts at the town square (the root context) and traces all paths leading out. Houses that have functional roads leading to them are marked "keep". Houses that are completely cut off by collapsed bridges are demolished (swept) to make space.

### Code Example
```javascript
function createLeakyReference() {
  let innerObj = { value: "some data" };
  // When this function returns, if there are no external references to innerObj,
  // the memory allocated for it will be marked for sweeping during the next GC run.
  return "Done";
}

createLeakyReference();
```

### Common Interview Questions
- Explain how the Mark-and-Sweep algorithm works in modern JS engines.
- What is the reference counting garbage collection algorithm and what is its main limitation?
- How does the garbage collector handle circular references?

### Reference Links
- [MDN Web Docs: Memory management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management)
- [V8: Trash talk: the garbage collector](https://v8.dev/blog/trash-talk)

## Memory Leaks in JavaScript

### Definition
A memory leak is a condition where a computer program incorrectly manages memory allocations, failing to release memory that is no longer needed by the application. Common sources in JavaScript include forgotten timers, active event listeners, detached DOM nodes, and global variable pollution.

### Real-world Analogy
Imagine leaving a garden hose running in your backyard while you go inside. You forget to turn the spigot off (clearing the interval/listener), causing water (memory) to pool up indefinitely and flood the yard (crash the browser).

### Code Example
```javascript
// Leaky timer example
function startLeakyTask() {
  const largeData = new Array(1000000).fill("data");
  
  setInterval(() => {
    // If this interval is never cleared, largeData remains locked in memory
    // because the callback closure references it.
    console.log("Timer ticking for data: " + largeData.length);
  }, 1000);
}
```

### Common Interview Questions
- What are detached DOM nodes and how do they cause memory leaks?
- How do closures contribute to memory leaks if not managed correctly?
- How do you profile and detect memory leaks using Chrome DevTools?

### Reference Links
- [MDN Web Docs: Common memory leaks](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_management#common_types_of_memory_leaks)
- [Chrome DevTools: Fix memory problems](https://developer.chrome.com/docs/devtools/memory-problems/)

## WeakMap and WeakSet

### Definition
WeakMap and WeakSet are collections that hold weak references to objects, meaning that if an object has no other references outside the WeakMap/WeakSet, it remains eligible for garbage collection. WeakMap keys must be objects, and they are not enumerable.

### Real-world Analogy
Think of a visitor register at a museum. If a guest writes their name on a piece of paper, but then leaves the building and ceases to exist, their entry in the register is automatically erased because there is no living person associated with it.

### Code Example
```javascript
let user = { id: 101, name: "Alice" };

const userMetadata = new WeakMap();
userMetadata.set(user, { lastLogin: Date.now() });

console.log(userMetadata.get(user)); // { lastLogin: ... }

// Nullify the only strong reference to the user object
user = null;
// The user object and its metadata in WeakMap will be garbage collected automatically
```

### Common Interview Questions
- Why are the keys of a WeakMap not enumerable (cannot run keys() or forEach())?
- What are the main use cases for WeakMap over a regular Map (e.g. DOM node metadata storage)?
- How does WeakSet help with tracking instance references without preventing cleanup?

### Reference Links
- [MDN Web Docs: WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
- [JavaScript.info: WeakMap and WeakSet](https://javascript.info/weakmap-weakset)

## Proxy and Reflect

### Definition
A Proxy object wraps target objects to intercept and customize low-level operations (like property lookup, assignment, enumeration). Reflect is a built-in object that provides static methods representing those interceptable operations, simplifying handler forwarding.

### Real-world Analogy
Think of a personal assistant standing in front of an executive. When a visitor asks the executive a question (getting a property), the assistant intercepts the question, checks if it is appropriate, validates the response, and forwards it using Reflect.

### Code Example
```javascript
const targetUser = { name: "Bob", role: "guest" };

const handler = {
  get(target, prop, receiver) {
    console.log("Reading property: " + prop);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    if (prop === "role" && value !== "guest") {
      throw new Error("Cannot change role directly!");
    }
    return Reflect.set(target, prop, value, receiver);
  }
};

const proxyUser = new Proxy(targetUser, handler);
console.log(proxyUser.name); // Reading property: name -> Bob
```

### Common Interview Questions
- What is a trap in a Proxy handler, and name three common traps.
- Why should you use Reflect methods inside Proxy trap handlers instead of executing target[prop]?
- How does Vue 3 use Proxies to implement its reactivity system?

### Reference Links
- [MDN Web Docs: Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)
- [MDN Web Docs: Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

## Modules

### Definition
Modules are self-contained files that encapsulate code blocks and export specific variables, functions, or classes. JavaScript supports two major module formats: CommonJS (synchronous, require/module.exports used in Node.js) and ESM (asynchronous, import/export, native to modern browsers and ES modules).

### Real-world Analogy
Think of Lego bricks. Each brick is made in its own factory and exposes specific connectors (exports). You import these bricks and snap them together to build a house, without needing to know the manufacturing secrets of each individual brick.

### Code Example
```javascript
// ES Module syntax (math.js)
export function add(a, b) {
  return a + b;
}

// Dynamic import example
async function loadMath() {
  const { add } = await import("./math.js");
  console.log(add(5, 3));
}
```

### Common Interview Questions
- Compare CommonJS and ES Modules regarding execution time, hoisting, and static analysis.
- What is tree shaking and how do ES Modules make it possible?
- How does dynamic import() work and when should you use it?

### Reference Links
- [MDN Web Docs: JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Node.js Documentation: Modules: CommonJS modules](https://nodejs.org/api/modules.html)

## Scenario-based Interview Questions for JavaScript

### Scenario 1
You are writing a script that fetches data for 100 products from an API. If you run them in a loop with await fetch(url) inside, users complain that the UI blocks or it takes too long. How do you resolve this?

*Expected Approach:*
1. Identify that sequential execution of await statements inside a loop blocks progress until the previous promise finishes.
2. Group the network requests into batches or execute them concurrently using Promise.all() or Promise.allSettled() to leverage network bandwidth.
3. Suggest a batching mechanism if 100 concurrent requests overload the API endpoint, implementing a limit using custom helpers or libraries.

### Scenario 2
You have a single-page application with a sidebar listing users. When users click a name, details load in the panel. After running for a few hours, the browser tab crashes. How do you investigate?

*Expected Approach:*
1. Look for active event listeners on user list items that are not cleaned up when items are re-rendered.
2. Check for closures holding references to older DOM elements (detached DOM nodes).
3. Open Chrome DevTools, capture a Heap Snapshot, perform the user-switching action 10 times, capture another snapshot, and compare the delta for lingering objects.
