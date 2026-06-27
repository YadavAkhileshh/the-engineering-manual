# 00 ROADMAP: Web Development Revision Journey

This roadmap outlines the complete learning and revision path to prepare for full-stack (MERN/PERN) interviews. It provides a structured 6-week revision plan, dependencies between topics, and time estimates to ensure you are fully prepared for technical screens.

## Revision Path and Dependencies

To get the most out of your preparation, study the topics in order of their dependencies. Do not skip ahead unless you are already comfortable with the prerequisite concepts.

### Prerequisite Flow

1. Core Fundamentals (Javascript Fundamentals, Network and Browser Internals)
Must be mastered before moving to Frontend or Backend.

2. Frontend Path (React, Next.js, TypeScript)
Requires Javascript Fundamentals. TypeScript should be learned alongside or after React to type-check components.

3. Backend Path (Node.js and Express, GraphQL)
Requires Javascript Fundamentals. Node.js should be learned before databases.

4. Database Path (MongoDB and Mongoose, PostgreSQL and Prisma)
Requires Backend path concepts.

5. Security, Testing, and Infrastructure (Authentication, Testing, Architecture, Performance, Docker, CI/CD)
Requires Frontend, Backend, and Database paths to be complete.

## Time Estimates per Phase

- Phase 1: Core Fundamentals (JavaScript & Networking) -> 12 Hours
- Phase 2: Frontend (React, Next.js & TypeScript) -> 18 Hours
- Phase 3: Backend & Databases (Node.js, Express, GraphQL, MongoDB, Postgres) -> 24 Hours
- Phase 4: Authentication, Security & Testing -> 15 Hours
- Phase 5: Advanced Optimization, Architecture & Queues -> 15 Hours
- Phase 6: DevOps, Cloud Hosting & Interview Prep (Docker, CI/CD, Scenario Questions) -> 16 Hours

Total Estimated Time: 100 Hours of focused revision and practice.

## The 6-Week Revision Plan

This schedule is designed for 15-18 hours of revision per week.

### Week 1: Core Fundamentals & Browser Internals

Focus: Getting your JavaScript and browser internals solid.

- Day 1: Execution context, call stack, hoisting, and scope chain. (File: 01-JAVASCRIPT-FUNDAMENTALS.md)
- Day 2: Closures, the 'this' keyword, and prototypal inheritance. (File: 01-JAVASCRIPT-FUNDAMENTALS.md)
- Day 3: Event loop deep dive, Promises, and Async/await patterns. (File: 01-JAVASCRIPT-FUNDAMENTALS.md)
- Day 4: Coercion, ES6+ features, garbage collection, and memory leaks. (File: 01-JAVASCRIPT-FUNDAMENTALS.md)
- Day 5: DNS resolution, TCP/UDP handshakes, HTTPS/TLS, and CORS mechanics. (File: 02-NETWORK-AND-BROWSER-INTERNALS.md)
- Day 6: Critical rendering path, Reflow/Repaint, and V8 engine compilation. (File: 02-NETWORK-AND-BROWSER-INTERNALS.md)
- Day 7: Package management (npm/yarn/pnpm), lock files, and node module resolution. (File: 02-NETWORK-AND-BROWSER-INTERNALS.md)

### Week 2: Modern Frontend & Typing

Focus: Declarative UI, routing, and type-safety.

- Day 8: Virtual DOM, reconciliation, JSX compilation, and React State (useState, functional updates). (File: 03-REACT.md)
- Day 9: useEffect lifecycle, useContext, useReducer, useRef, and performance hooks (useMemo, useCallback). (File: 03-REACT.md)
- Day 10: Custom hooks patterns, React Router v6, controlled vs uncontrolled forms. (File: 03-REACT.md)
- Day 11: State management (Redux vs Zustand), data fetching (React Query), and React 19 features. (File: 03-REACT.md)
- Day 12: Next.js App Router, Server Components, and Server Actions. (File: 04-NEXT-JS.md)
- Day 13: Next.js SSR, SSG, ISR, and Middleware route protection. (File: 04-NEXT-JS.md)
- Day 14: TypeScript basics, Generics, Utility Types, and integration with React/Express. (File: 05-TYPESCRIPT.md)

### Week 3: Backend Architecture & Databases

Focus: Server logic, asynchronous scaling, and database queries.

- Day 15: Node.js architecture (V8, libuv, event loop), worker threads, and child processes. (File: 06-NODE-JS-AND-EXPRESS.md)
- Day 16: Streams (backpressure), Event Emitters, and Express middleware patterns. (File: 06-NODE-JS-AND-EXPRESS.md)
- Day 17: Express validation, file uploads, rate limiting, and security middleware. (File: 06-NODE-JS-AND-EXPRESS.md)
- Day 18: GraphQL Schema Definition, Resolvers, and Apollo Client integration. (File: 07-GRAPHQL.md)
- Day 19: MongoDB architecture, indexes, aggregation pipeline, and Mongoose middleware. (File: 08-MONGODB-AND-MONGOOSE.md)
- Day 20: PostgreSQL fundamentals, Joins, CTEs, Window functions, and Prisma ORM. (File: 09-POSTGRESQL-AND-PRISMA.md)
- Day 21: Database migration, transactions, locks, and pgAdmin operations. (File: 09-POSTGRESQL-AND-PRISMA.md)

### Week 4: Security, Authentication & Testing

Focus: Keeping applications secure and proving they work.

- Day 22: Session-based vs Token-based (JWT) auth, cookie flags (HttpOnly, Secure, SameSite). (File: 10-AUTHENTICATION-AND-AUTHORIZATION.md)
- Day 23: Access/Refresh token rotation, OAuth 2.0 flow, and Passport/NextAuth setups. (File: 10-AUTHENTICATION-AND-AUTHORIZATION.md)
- Day 24: Passwords hashing (bcrypt), password reset flows, and Role-Based Access Control (RBAC). (File: 10-AUTHENTICATION-AND-AUTHORIZATION.md)
- Day 25: Prevention of XSS, CSRF, SQL Injection, and NoSQL injection. (File: 10-AUTHENTICATION-AND-AUTHORIZATION.md)
- Day 26: Testing Pyramid, Jest basics, mocking functions, and coverage reports. (File: 11-TESTING.md)
- Day 27: Supertest for API routes, React Testing Library, and MSW for API mocking. (File: 11-TESTING.md)
- Day 28: End-to-end testing with Playwright or Cypress, and Page Object Model (POM). (File: 11-TESTING.md)

### Week 5: System Architecture & Optimization

Focus: Designing for scale, decoupling, and high performance.

- Day 29: SOLID principles, MVC, Repository, and Service layers. (File: 12-ARCHITECTURE-PATTERNS.md)
- Day 30: Microservices, API gateways, event-driven architecture, and Saga pattern. (File: 12-ARCHITECTURE-PATTERNS.md)
- Day 31: N+1 query detection, database index optimization, and caching strategies. (File: 13-PERFORMANCE-OPTIMIZATION.md)
- Day 32: Redis caching, CDN setup, Gzip/Brotli, and Web Vitals (LCP, INP, CLS). (File: 13-PERFORMANCE-OPTIMIZATION.md)
- Day 33: Full-text search in MongoDB/Postgres, Elasticsearch, and Algolia. (File: 14-SEARCH.md)
- Day 34: Message queues, BullMQ with Redis, and dead letter queues. (File: 15-MESSAGE-QUEUES-AND-BACKGROUND-JOBS.md)
- Day 35: Version control, Git Flow, conventional commits, and semantic versioning. (File: 16-VERSION-CONTROL-AND-COLLABORATION.md)

### Week 6: DevOps, Hosting & Interview Prep

Focus: Containers, pipelines, server deployment, and final interview prep.

- Day 36: Docker concepts, Dockerfile optimization, and multi-stage builds. (File: 17-DOCKER-AND-CONTAINERIZATION.md)
- Day 37: Docker Compose setup, networking, and volumes. (File: 17-DOCKER-AND-CONTAINERIZATION.md)
- Day 38: CI/CD principles, GitHub Actions pipelines, caching, and secrets. (File: 18-CI-CD-PIPELINES.md)
- Day 39: Frontend hosting (Vercel, Netlify, Cloudflare Pages) and CDNs. (File: 19-FRONTEND-DEPLOYMENT.md)
- Day 40: Backend hosting (Railway, Render, AWS EC2 with Nginx & PM2). (File: 20-BACKEND-DEPLOYMENT.md)
- Day 41: Database hosting (Atlas, Supabase, Neon) and monitoring tools (Winston, Sentry). (File: 21-DATABASE-HOSTING.md & File: 22-MONITORING-AND-LOGGING.md)
- Day 42: Scenario-based questions practice, cheat sheets, and project ideas. (File: 23-SCENARIO-BASED-INTERVIEW-QUESTIONS.md, File: 24-CHEAT-SHEETS.md, & File: 25-PROJECT-IDEAS.md)
