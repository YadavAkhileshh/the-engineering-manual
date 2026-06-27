# 07 GraphQL

This document covers GraphQL fundamentals, SDL schemas, resolvers, queries, mutations, subscriptions, Apollo integrations, DataLoader optimization, and security.

## GraphQL Fundamentals

### Definition
GraphQL is an open-source data query and manipulation language for APIs, and a runtime for fulfilling those queries with your existing data. It uses a single endpoint and allows clients to request exactly the data they need, eliminating over-fetching and under-fetching.

### Real-world Analogy
Imagine walking into a custom salad bar. Instead of ordering a pre-made chef salad (REST endpoint) and having to pick out the onions you hate, you write down a checklist specifying exactly which greens, meats, and dressings you want. You get a bowl containing only what you checked.

### Code Example
```graphql
# Client query requesting specific fields from a single endpoint
query GetUserData {
  user(id: "101") {
    name
    email
  }
}
```

### Common Interview Questions
- What are the major differences between GraphQL and REST APIs?
- Explain the concepts of over-fetching and under-fetching.
- Why does GraphQL only require a single POST endpoint (/graphql)?

### Reference Links
- [GraphQL: Official Introduction](https://graphql.org/learn/)
- [Apollo: GraphQL vs REST](https://www.apollographql.com/blog/graphql-vs-rest-555df3fb548c)

## Schema Definition with SDL

### Definition
Schema Definition Language (SDL) is the syntax used to define a GraphQL service's type system. It outlines types, input structures, enums, custom scalar types, and specifies mandatory fields (!) and arrays ([]).

### Real-world Analogy
Think of a real-estate architectural blueprint. The blueprint specifies the types of rooms (living room, bedroom), what inputs are required (must have a front door: !), and which sections can have multiple layouts (e.g. closet lists: [Closet]).

### Code Example
```graphql
enum UserRole {
  ADMIN
  USER
}

type User {
  id: ID!
  email: String!
  role: UserRole!
  posts: [Post!]!
}

type Query {
  getUser(id: ID!): User
}
```

### Common Interview Questions
- What does the exclamation mark (!) represent in a schema definition?
- Explain the difference between type definitions and input types.
- How do you register custom scalar types (like Date or JSON) in a schema?

### Reference Links
- [GraphQL: Schemas and Types](https://graphql.org/learn/schema/)
- [Apollo: Schema Basics](https://www.apollographql.com/docs/apollo-server/schema/schema)

## Resolvers

### Definition
Resolvers are backend functions responsible for returning the data for a specific field in a GraphQL query. They receive four positional parameters: parent (or root), args (input variables), context (shared state like database/auth), and info (query metadata).

### Real-world Analogy
Imagine a supply manager at a factory. When an order arrives for a car, the manager doesn't build it all themselves. They assign individual departments (resolvers): the engine department handles the engine, the wheel department gets the wheels, and they pass the parts back to assemble the final car.

### Code Example
```javascript
const resolvers = {
  Query: {
    getUser: async (parent, args, context, info) => {
      // Fetch user from DB using the ID passed in args
      return await context.db.users.findById(args.id);
    }
  },
  User: {
    posts: async (parent, args, context) => {
      // Resolve nested field: 'parent' contains the User object resolved above
      return await context.db.posts.findByUserId(parent.id);
    }
  }
};
```

### Common Interview Questions
- Describe the four parameters passed into a resolver function.
- What is a default resolver and when does GraphQL run it?
- How do you implement resolver composition to protect specific route queries (authorization)?

### Reference Links
- [GraphQL: Execution](https://graphql.org/learn/execution/)
- [Apollo: Resolvers](https://www.apollographql.com/docs/apollo-server/data/resolvers)

## Queries

### Definition
Queries are read-only operations used to request data. They support field selection, parameters, aliases (to rename return keys), variables (for dynamic inputs), fragments (reusable sets of fields), and directives (conditional rendering).

### Real-world Analogy
Think of pulling files from a cabinet. Instead of pulling the whole folder, you request only the document pages you need. If you need two copies of the same file but with different labels, you use custom sticky notes (aliases) to rename them.

### Code Example
```graphql
# Query using variables, aliases, and fragments
query GetActiveProducts($limit: Int!) {
  discounted: products(filter: "discounted", limit: $limit) {
    ...ProductFields
  }
}

fragment ProductFields on Product {
  id
  name
  price
}
```

### Common Interview Questions
- What are GraphQL fragments and how do they reduce query code duplication?
- How do you use variables to prevent string injection vulnerabilities in queries?
- Explain the role of @include and @skip directives.

### Reference Links
- [GraphQL: Queries and Mutations](https://graphql.org/learn/queries/)

## Mutations

### Definition
Mutations are write operations in GraphQL used to create, update, or delete data on the server. They follow the same syntax as queries but are executed sequentially rather than in parallel to prevent database race conditions.

### Real-world Analogy
Think of submitting a deposit slip at a bank. You fill out the slip containing the cash details (input types), slide it through the window, and the teller updates your account. You wait at the window to receive the printed receipt showing your updated balance (returned fields).

### Code Example
```graphql
# Mutation schema definition
# mutation { createUser(input: { email: "test@test.com" }) { id success } }

const resolvers = {
  Mutation: {
    createUser: async (_, { input }, { db }) => {
      const newUser = await db.users.create(input);
      return {
        success: true,
        user: newUser
      };
    }
  }
};
```

### Common Interview Questions
- Why do mutations run sequentially while queries execute in parallel?
- What are mutation input types and why should you use them instead of loose arguments?
- Describe the best practice pattern for formatting mutation responses.

### Reference Links
- [Apollo: Mutations](https://www.apollographql.com/docs/apollo-server/data/mutations)

## Subscriptions

### Definition
Subscriptions are real-time, long-lasting connections that push data from the server to clients when specific events occur. They are implemented using WebSockets instead of standard HTTP transport.

### Real-world Analogy
Think of subscribing to a storm warning system. Instead of constantly looking out the window or calling the weather station every 5 minutes (polling), you connect to a radio channel. The speaker remains silent until a storm alert occurs, pushing the warning to you instantly.

### Code Example
```javascript
const { PubSub } = require("graphql-subscriptions");
const pubsub = new PubSub();

const resolvers = {
  Subscription: {
    postAdded: {
      subscribe: () => pubsub.asyncIterator(["POST_ADDED"]),
    },
  },
  Mutation: {
    addPost: async (_, args) => {
      const newPost = await savePost(args);
      pubsub.publish("POST_ADDED", { postAdded: newPost });
      return newPost;
    }
  }
};
```

### Common Interview Questions
- When should you choose WebSockets subscriptions over short/long polling?
- Explain the architecture of a PubSub engine in production cluster environments.
- How do you configure authentication protocols for subscriptions in Apollo Server?

### Reference Links
- [Apollo: Subscriptions](https://www.apollographql.com/docs/apollo-server/data/subscriptions)

## Apollo Server Setup

### Definition
Apollo Server is an open-source, spec-compliant GraphQL server that integrates with any Node.js HTTP framework (like Express), resolving queries using declared schemas and resolver maps.

### Real-world Analogy
Apollo Server is like a translation terminal in a train station. Train timetables are printed in GraphQL. The terminal receives request tickets, decodes the passenger names, checks their passports (context), and runs the translation tools (resolvers) to return local directions.

### Code Example
```javascript
const { ApolloServer } = require("@apollo/server");
const { startStandaloneServer } = require("@apollo/server/standalone");

const typeDefs = `#graphql
  type Query { hello: String }
`;

const resolvers = {
  Query: { hello: () => "world" }
};

const server = new ApolloServer({ typeDefs, resolvers });

startStandaloneServer(server, { listen: { port: 4000 } })
  .then(({ url }) => console.log(`Server ready at ${url}`));
```

### Common Interview Questions
- How do you configure shared context (like db connections or auth tokens) in Apollo Server constructor settings?
- Explain how to integrate Apollo Server with an existing Express app using expressMiddleware.
- What is the purpose of plugins in Apollo Server?

### Reference Links
- [Apollo Server: Getting Started](https://www.apollographql.com/docs/apollo-server/getting-started)

## Apollo Client Setup

### Definition
Apollo Client is a state management library that manages GraphQL queries, caching responses automatically in a local InMemoryCache, handling mutations, and updating the React UI using hooks.

### Real-world Analogy
Think of a librarian who keeps a notebook. When you ask for a book, they look in their notebook first (InMemoryCache). If they have a copy, they show it to you instantly. If not, they run to the vault, bring it back, show you, and write the details down in the notebook for next time.

### Code Example
```jsx
import { ApolloClient, InMemoryCache, ApolloProvider, useQuery, gql } from "@apollo/client";

const client = new ApolloClient({
  uri: "http://localhost:4000",
  cache: new InMemoryCache()
});

const GET_HELLO = gql`
  query GetHello { hello }
`;

function HelloMessage() {
  const { loading, data } = useQuery(GET_HELLO);
  if (loading) return <span>Loading...</span>;
  return <h1>{data.hello}</h1>;
}
```

### Common Interview Questions
- How does Apollo Client's InMemoryCache normalize query data?
- Explain the fetchPolicy configurations (e.g. cache-first vs network-only).
- How do you update the local cache manually after performing a mutation?

### Reference Links
- [Apollo Client: React Integration](https://www.apollographql.com/docs/react/get-started)

## DataLoader for N+1 Problem

### Definition
DataLoader is a utility library used to resolve the N+1 database query problem in GraphQL. It batches multiple individual resource requests made in a single event loop tick into a single batch query, caching the results per request.

### Real-world Analogy
Imagine a school field trip where 30 kids each want a soda. Instead of 30 kids walking one-by-one to the store, buying one soda, and walking back (30 network trips: N+1), the teacher writes down all 30 names, walks to the store once, buys a crate of 30 sodas, and hands them out.

### Code Example
```javascript
const DataLoader = require("dataloader");

// Batch loading function (receives array of keys, must return array of results in same order)
const batchUsers = async (keys) => {
  const users = await db.users.findMany({ id: { $in: keys } });
  // Map back to guarantee same index order
  return keys.map(key => users.find(user => user.id === key));
};

const userLoader = new DataLoader(batchUsers);

// Inside post resolver
const resolvers = {
  Post: {
    author: (parent) => userLoader.load(parent.authorId)
  }
};
```

### Common Interview Questions
- Why does the batch loading function require the returned array to match the input keys array in length and order?
- Compare DataLoader batching and cache capabilities. Why should DataLoader caches be cleared on every request?
- How does DataLoader coordinate batching using process.nextTick?

### Reference Links
- [GitHub: DataLoader](https://github.com/graphql/dataloader)

## Error Handling in GraphQL

### Definition
Error handling in GraphQL returns errors inside an 'errors' array alongside the query data, rather than throwing a non-200 HTTP status code. Custom errors use GraphQLError to attach code details.

### Real-world Analogy
Imagine ordering a complex multi-course meal. The waiter delivers the salad and steak, but tells you, "We burnt the dessert" (error object in response). The meal is not cancelled, you still eat the steak (data), but you get a note explaining the dessert failure.

### Code Example
```javascript
const { GraphQLError } = require("graphql");

const resolvers = {
  Query: {
    secureData: (_, __, context) => {
      if (!context.user) {
        throw new GraphQLError("Not authenticated", {
          extensions: { code: "UNAUTHENTICATED", http: { status: 401 } }
        });
      }
      return "Secret data";
    }
  }
};
```

### Common Interview Questions
- Why do GraphQL APIs return HTTP Status 200 even when a query contains validation or authentication errors?
- How do you configure formatError in Apollo Server to hide sensitive stack traces in production?
- How does the client parse the GraphQL errors array?

### Reference Links
- [Apollo: Error Handling](https://www.apollographql.com/docs/apollo-server/data/errors)

## GraphQL Security

### Definition
GraphQL security involves protecting endpoints against abusive queries. Strategies include query depth limiting (rejecting deeply nested trees), query complexity analysis (assigning costs to fields), and disabling introspection in production.

### Real-world Analogy
Imagine a public library. Introspection is handing visitors a full map of the archives (disable in production). Depth limiting is saying: "You can open boxes inside boxes, but you cannot open a box nested 10 layers deep". Complexity analysis is checking: "You can check out 5 books, but reading 50 books in one hour is not allowed".

### Code Example
```javascript
// Disabling Introspection and Sandbox in Apollo Server Production
const { ApolloServerPluginLandingPageDisabled } = require("@apollo/server/plugin/disabled");

const server = new ApolloServer({
  typeDefs,
  resolvers,
  introspection: false, // Disables query schema parsing
  plugins: [ApolloServerPluginLandingPageDisabled()]
});
```

### Common Interview Questions
- What is an introspection query and why should it be disabled in production environments?
- Describe how a nested recursive query can trigger a Denial of Service (DoS) attack.
- Explain the concept of query complexity analysis and how it is enforced.

### Reference Links
- [OWASP: GraphQL Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Security_Cheat_Sheet.html)

## When to Use GraphQL vs REST

### Definition
GraphQL is ideal for complex, nested data requirements, multiple client platforms (web, mobile), and large microservice aggregation. REST is preferred for simple APIs, uniform resource caching, and binary files.

### Real-world Analogy
GraphQL is a high-speed department store delivery truck that gathers items from 5 departments and delivers them in one customized package. REST is a fleet of individual courier bikes that each pick up one specific box from one warehouse.

### Code Example
```javascript
// REST requires multiple fetches:
// 1. fetch('/user/101') -> get userId
// 2. fetch('/user/101/posts') -> get postIds
// 3. fetch('/posts/502/comments') -> get comments

// GraphQL resolves all three in one query:
// query { user(id: 101) { posts { comments { text } } } }
```

### Common Interview Questions
- In what scenarios is REST a better architectural choice than GraphQL?
- How does HTTP caching differ between REST (GET route caches) and GraphQL (POST endpoint challenges)?
- What are the team and API governance implications of transitioning to GraphQL schemas?

### Reference Links
- [GraphQL: Comparison](https://graphql.org/faq/)

## Scenario-based Interview Questions for GraphQL

### Scenario 1
You have a dashboard displaying posts, and each post resolves its author. The database logs show that loading 10 posts runs 1 query for posts and 10 separate queries to fetch authors. How do you identify this issue and fix it?

*Expected Approach:*
1. Identify this as the classic N+1 query problem, caused by the nested resolver structure running database calls in a loop.
2. Resolve it by introducing a DataLoader instance in the request context.
3. The loader batches the 10 individual author IDs, executes one SQL SELECT * FROM users WHERE id IN (...) query, and maps the results back, reducing queries from 11 to 2.

### Scenario 2
Your GraphQL server crashes occasionally because developers or automated scrapers submit nested queries like user { posts { comments { author { posts { comments { text } } } } } }. How do you safeguard the system?

*Expected Approach:*
1. Explain that deeply nested recursive queries consume server CPU and memory resources, causing stack overflow crashes.
2. Mitigate this by installing query depth limiting middleware (like graphql-depth-limit) to reject queries that exceed a depth threshold (e.g. maximum of 4 or 5 levels).
3. Disable schema introspection in production to prevent scrapers from mapping recursive relationships.
