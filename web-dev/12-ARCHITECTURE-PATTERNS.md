# 12 Architecture Patterns

This guide covers software architecture patterns: Layered/MVC structures, Repository & Service layers, Dependency Injection, SOLID design principles, Microservices, BFF, and Event-Driven designs.

## MVC (Model-View-Controller) Pattern

### Definition
MVC is an architectural pattern that separates an application into three main logical components: the Model (handles data structure and business logic), the View (manages user interface presentation), and the Controller (intercepts user requests, interacts with models, and coordinates view updates).

### Real-world Analogy
Think of ordering food at a restaurant. The waiter (Controller) takes your order slip and carries it to the kitchen. The kitchen staff (Model) cook the meal using raw ingredients. The waiter picks up the cooked food and places it on a nice plate on your table (View) for you to eat.

### Code Example
```javascript
// Express MVC folder structure mapping
// routes/userRoutes.js -> coordinates with UserController
// controllers/userController.js -> Handles req/res, calls models
// models/userModel.js -> Schema representation

const UserController = {
  getProfile: async (req, res) => {
    const user = await UserModel.findById(req.params.id);
    res.render("profileView", { user });
  }
};
```

### Common Interview Questions
- Explain how MVC separates concerns in a full-stack application.
- What are the main drawbacks of placing business logic inside controllers?
- How does the View-Controller relationship differ in Single Page Applications (SPAs)?

### Reference Links
- [MDN Web Docs: MVC Architecture](https://developer.mozilla.org/en-US/docs/Glossary/MVC)

## Repository Pattern

### Definition
The Repository Pattern abstracts the data access layer, exposing a collection-like interface (findAll, findById) to business logic, which decouples the application from specific database implementations (MongoDB, PostgreSQL) and simplifies unit testing.

### Real-world Analogy
Imagine a librarian managing books. The library users do not know if books are stored in basement drawers, local shelves, or downloaded from a digital cloud database. They just ask the librarian: "Find book 'X'". The librarian (Repository) knows where it is and brings it back.

### Code Example
```typescript
interface IUserRepository {
  findById(id: string): Promise<User | null>;
  create(user: User): Promise<User>;
}

// Concrete Mongoose implementation
class MongoUserRepository implements IUserRepository {
  async findById(id: string) {
    return await UserModel.findById(id).lean();
  }
  async create(user: User) {
    return await UserModel.create(user);
  }
}
```

### Common Interview Questions
- What are the benefits of using a Repository pattern when writing unit tests?
- How does the repository pattern prevent database lock-in?
- Describe the difference between a Repository and a Data Access Object (DAO).

### Reference Links
- [Microsoft: Design the infrastructure data access layer](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-web-api/infrastructure-data-access-repository-design-patterns)

## Service Layer

### Definition
The Service Layer encapsulates the core business logic of an application, coordinating workflows, managing database transactions, and resolving business rules independently of HTTP controllers or data schemas.

### Real-world Analogy
Think of a bank transaction service. The action of "transferring money" involves verifying balances, calculating taxes, subtracting funds, updating logs, and emailing receipts. The Service handles this sequence, ensuring all steps succeed or fail together.

### Code Example
```javascript
class PaymentService {
  constructor(paymentGateway, orderRepo) {
    this.paymentGateway = paymentGateway;
    this.orderRepo = orderRepo;
  }

  async processOrder(orderId, paymentToken) {
    const order = await this.orderRepo.findById(orderId);
    if (order.status === "paid") throw new Error("Already paid");
    
    await this.paymentGateway.charge(order.total, paymentToken);
    await this.orderRepo.updateStatus(orderId, "paid");
  }
}
```

### Common Interview Questions
- Why should controllers never contain business calculations or database queries?
- How does the Service Layer coordinate transactions across multiple repositories?
- Compare the "anemic domain model" and the "rich domain model" inside service structures.

### Reference Links
- [Martin Fowler: Service Layer](https://martinfowler.com/eaaCatalog/serviceLayer.html)

## Dependency Injection

### Definition
Dependency Injection (DI) is a design pattern where a class receives its dependencies from an external source (injector) rather than instantiating them internally, enabling decoupling, modular configurations, and easy test mocking.

### Real-world Analogy
Imagine building a computer. Instead of soldering the hard drive directly onto the motherboard at the factory (hardcoded dependency), you design slot connectors. This allows you to slide in any hard drive (inject dependency) when assembling the computer.

### Code Example
```javascript
// Manual Dependency Injection in Node.js
class UserController {
  // Dependencies are injected via constructor
  constructor(userService) {
    this.userService = userService;
  }

  async getUser(req, res) {
    const user = await this.userService.getUserProfile(req.params.id);
    res.json(user);
  }
}

// Composition Root Setup
const userService = new UserService(userRepository);
const userController = new UserController(userService);
```

### Common Interview Questions
- Explain how Dependency Injection improves component testability.
- What is a Dependency Injection Container (IOC) and how does it manage component lifecycles?
- What are the differences between constructor injection and setter/property injection?

### Reference Links
- [Martin Fowler: Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)

## Clean Architecture

### Definition
Clean Architecture (by Robert C. Martin) organizes code into concentric rings where dependency directions point strictly inwards. The innermost ring is the Domain Entities (business rules), surrounded by Use Cases, Interface Adapters (controllers), and Infrastructure (DB, Web).

### Real-world Analogy
Think of a castle. The king and queen (Domain Entities) live in the innermost keep. The guards and advisors (Use Cases) coordinate tasks. The castle gates and drawbridges (Adapters) communicate with external trading merchants (Infrastructure). The king doesn't care what carriage design merchants use.

### Code Example
```
// Folder layout representing Clean Architecture boundaries:
// src/domain/entities/User.ts
// src/use-cases/RegisterUser.ts
// src/adapters/controllers/UserController.ts
// src/infrastructure/database/MongoUserRepository.ts
```

### Common Interview Questions
- Describe the Dependency Rule in Clean Architecture.
- Why should the core business domain have zero dependencies on external libraries (like Express or Mongoose)?
- How do you handle database entities vs domain entities mapping?

### Reference Links
- [The Clean Code Blog: Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)

## SOLID Principles

### Definition
SOLID represents five object-oriented design principles: Single Responsibility (one reason to change), Open/Closed (open for extension, closed for modification), Liskov Substitution (subtypes must be substitutable), Interface Segregation (small interfaces), and Dependency Inversion (depend on abstractions).

### Real-world Analogy
Interface Segregation is like a charging cable: you don't buy one mega-plug containing electric prongs, water hoses, and gas tubes. You want a small, focused cable that only carries power. Dependency Inversion is using wall outlets: your lamp plugs into the outlet socket (abstraction) rather than hardwiring directly into the city's power grid.

### Code Example
```typescript
// Single Responsibility Principle (SRP) Example
class Invoice {
  // Bad: class calculates total AND prints PDF
  calculateTotal() {}
}
class InvoicePrinter {
  // Good: separated printer logic
  printPdf(invoice: Invoice) {}
}
```

### Common Interview Questions
- Give an example of violating the Open/Closed Principle (OCP) and how to resolve it.
- Explain the Liskov Substitution Principle (LSP) with an example.
- How does the Dependency Inversion Principle (DIP) relate to Dependency Injection (DI)?

### Reference Links
- [SOLID Principles Explained](https://en.wikipedia.org/wiki/SOLID)

## Microservices Basics

### Definition
Microservices is an architectural style that structures an application as a collection of small, loosely coupled, independently deployable services organized around business domains. Services communicate via lightweight protocols (REST, gRPC, Message Queues).

### Real-world Analogy
A Monolith is a multi-talented handyman who does plumbing, wiring, painting, and roofing: if they catch the flu, all renovations stop. Microservices is hiring a team of 4 independent contractors: the painter works in room A while the plumber works in room B, and if the painter leaves, the plumbing still works.

### Code Example
```
// Monolith API Endpoint
// GET /api/users-orders -> SQL Joins across tables

// Microservices Aggregator Route (API Gateway Pattern)
// GET /api/users-orders ->
//   1. fetch('user-service/users/1')
//   2. fetch('order-service/orders?userId=1')
//   3. Merge data and return JSON
```

### Common Interview Questions
- Compare Monoliths and Microservices regarding deployment and scalability.
- What is the Saga pattern and how does it manage transactions across distributed microservice databases?
- What are the drawbacks and operational complexities of microservice architectures?

### Reference Links
- [Microservice Architecture Website](https://microservices.io/)
- [Saga Pattern](https://microservices.io/patterns/data/saga.html)

## Backend For Frontend (BFF) Pattern

### Definition
The Backend for Frontend (BFF) pattern defines separate backend services tailored specifically for individual client interfaces (e.g. one BFF for a mobile app, one for web app), aggregating multiple internal microservices.

### Real-world Analogy
Imagine a catalog store. Instead of giving the same 1,000-page catalog book to both a mailroom shopper and a high-speed smartphone viewer, the store prints a small pamphlet for mobile users showing only quick codes, and a large book for web users showing full reviews.

### Code Example
```javascript
// Web BFF Server Example (Returns detailed widgets and sidebars)
app.get("/dashboard", async (req, res) => {
  const data = await fetchDashboardAggregatedDetails();
  res.json({ sidebar: data.sidebar, widgets: data.widgets, user: data.user });
});

// Mobile BFF Server Example (Returns light weight version to save data bandwidth)
app.get("/dashboard", async (req, res) => {
  const data = await fetchDashboardLightDetails();
  res.json({ user: data.user, status: data.status });
});
```

### Common Interview Questions
- Why do companies use the BFF pattern instead of exposing a single unified API Gateway?
- How does the BFF pattern optimize mobile battery life and data usage?
- Who should maintain the BFF codebase: the backend team or the frontend team?

### Reference Links
- [Microsoft: Backend for Frontend pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends)

## Event-Driven Architecture

### Definition
Event-Driven Architecture (EDA) is a design pattern where services react to, produce, and consume events representing state changes, using message brokers (RabbitMQ, Apache Kafka) to decouple producers and consumers.

### Real-world Analogy
Think of ordering online. When you buy a shirt, the ordering system fires an event: "Order Placed" (produces event). The warehouse printer hears the event and prints a box packing slip (consumes event). The shipping courier system hears the event and reserves a delivery truck. The ordering service has no idea the delivery truck was reserved.

### Code Example
```javascript
// Simple Event Publisher template
class EventBus {
  constructor() {
    this.listeners = {};
  }
  subscribe(event, callback) {
    this.listeners[event] = this.listeners[event] || [];
    this.listeners[event].push(callback);
  }
  publish(event, data) {
    if (this.listeners[event]) {
      this.listeners[event].forEach(cb => cb(data));
    }
  }
}
```

### Common Interview Questions
- Explain the difference between an Event and a Command.
- What is Event Sourcing and how does it record application state?
- Describe the CQRS (Command Query Responsibility Segregation) pattern.

### Reference Links
- [AWS: What is an Event-Driven Architecture?](https://aws.amazon.com/event-driven-architecture/)

## Layered Architecture

### Definition
Layered Architecture (N-Tier) divides applications into horizontal layers where each layer has a distinct role (Presentation, Application/Service, Domain, Data Access/Infrastructure). Lower layers are closed to access from outer layers.

### Real-world Analogy
Think of the postal shipping system. The shipping center has layers: Customer desk writes letter -> Regional sorting warehouse categorizes routes -> Truck drivers deliver packages. The desk clerk doesn't jump in the truck and drive to another city; they only pass documents to the sorting warehouse layer.

### Code Example
```
// Layered architectural flow:
// Presentation Layer (Controller handles API input/output)
//     v
// Application Layer (Service validates business calculations)
//     v
// Infrastructure Layer (Repository reads PostgreSQL row)
```

### Common Interview Questions
- What are the differences between an "Open Layer" and a "Closed Layer"?
- Explain the key benefits and drawbacks of Layered Architecture.
- How do database modifications ripple through Layered structures?

### Reference Links
- [O'Reilly: Layered Architecture Pattern](https://www.oreilly.com/library/view/software-architecture-patterns/9781491971437/ch01.html)

## Scenario-based Interview Questions for Architecture Patterns

### Scenario 1
You are building an application with a monolithic structure. Users complain that changing a database field requires modification of controllers, service helpers, and models, causing long deployment cycles. How do you design the application layers to prevent this coupling?

*Expected Approach:*
1. Propose decoupling the layers by introducing Interfaces for the data access layer (Repository Pattern) and Service Layers.
2. Implement Domain Entities that are decoupled from Mongoose/Prisma schemas.
3. Map database schema outputs to clean domain entities inside the repository boundary, preventing database schema changes from rippling up to the business services and controllers.

### Scenario 2
An e-commerce company wants to split a monolithic server into microservices. The checkouts system needs to verify stock in inventory, charge credit cards in payments, and deduct reward points in loyalty. How do you handle a scenario where payments succeed but loyalty system fails?

*Expected Approach:*
1. Explain that traditional ACID transactions do not work across microservice database boundaries.
2. Resolve this using the Saga Pattern (specifically Orchestrator-based or Choreography-based).
3. If a step fails, the orchestrator triggers Compensating Transactions (rollback actions), such as voiding the payment card charge and restoring inventory items, maintaining eventual consistency.
