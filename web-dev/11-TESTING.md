# 11 Testing

This guide covers testing practices: unit and integration tests (Jest, Vitest), component testing (React Testing Library), MSW mocking, E2E (Playwright, Cypress), and test strategy design.

## Testing Pyramid

### Definition
The testing pyramid is a framework diagram describing the balance of test suites. It recommends a high volume of fast, isolated Unit Tests at the base, a moderate number of Integration Tests in the middle, and a minimal number of slow, comprehensive End-to-End (E2E) tests at the peak.

### Real-world Analogy
Imagine building a passenger airplane. You test every screw, wire, and bolt individually on a bench (Unit Tests). You test the engines and fuel lines hooked up together to check fuel flow (Integration Tests). Finally, you fly the complete airplane with a pilot (E2E Test) before clearing it for passenger service.

### Code Example
```javascript
// A simple unit test demonstrating the base of the pyramid
function add(a, b) {
  return a + b;
}

test("adds two numbers", () => {
  expect(add(2, 3)).toBe(5);
});
```

### Common Interview Questions
- Why is the testing pyramid shaped like a pyramid rather than an inverted funnel?
- What are the cost and speed trade-offs of unit tests versus E2E tests?
- When should you break the pyramid shape (e.g. testing trophy concept)?

### Reference Links
- [Martin Fowler: The Practical Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html)

## Test-Driven Development (TDD)

### Definition
Test-Driven Development (TDD) is a software development process relying on the repetition of a short development cycle: first write an automated test that fails (Red), write the minimal production code to pass the test (Green), and clean up the code (Refactor).

### Real-world Analogy
Imagine building a custom brick wall. First, you set up an empty metal template frame indicating the exact height and width you want (Red). You lay bricks until they touch the frame limits (Green). Finally, you scrape away excess mortar and polish the brick edges (Refactor).

### Code Example
```javascript
// Step 1: Write failing test (Red)
// test('calculates tax', () => { expect(calcTax(100)).toBe(10); });

// Step 2: Write minimal code to pass (Green)
function calcTax(amount) {
  return amount * 0.1;
}

// Step 3: Refactor (clean up internal variables, types, or syntax)
```

### Common Interview Questions
- Describe the steps of the Red-Green-Refactor cycle in TDD.
- What are the main benefits and drawbacks of TDD?
- When is TDD not appropriate (e.g. legacy exploration or fast prototyping)?

### Reference Links
- [Beck, Kent: Test-Driven Development By Example](https://www.oreilly.com/library/view/test-driven-development/0321146530/)

## Jest Fundamentals

### Definition
Jest is a JavaScript testing framework designed to ensure correctness. It provides test runners, assertions (matchers like toBe, toEqual), setup/teardown hooks (beforeEach, afterEach), and support for running mock functions.

### Real-world Analogy
Imagine a school exam supervisor. The supervisor calls out the start of the test (beforeAll), hands out the papers (beforeEach), checks if the student's answer equals the textbook answer (expect().toBe()), collects the sheets, and cleans the desks (afterEach).

### Code Example
```javascript
describe("User Service", () => {
  let user;

  beforeEach(() => {
    user = { id: 1, name: "Alice" };
  });

  test.each([
    [1, "Alice"],
    [2, "Bob"]
  ])("identifies user %i as %s", (id, name) => {
    expect(id).toBeDefined();
    expect(name).toBeTruthy();
  });
});
```

### Common Interview Questions
- What is the difference between the toBe and toEqual matchers in Jest?
- How do you use test.each for data-driven testing?
- Explain the execution order of beforeEach, afterEach, beforeAll, and afterAll hooks.

### Reference Links
- [Jest: Matchers Documentation](https://jestjs.io/docs/using-matchers)
- [Jest: Setup and Teardown](https://jestjs.io/docs/setup-teardown)

## Jest Configuration

### Definition
Jest configuration files (jest.config.js) customize the testing environment, declaring transformer modules (like ts-jest or babel-jest), path mapping resolution (moduleNameMapper), setup script declarations, and testing coverage thresholds.

### Real-world Analogy
Think of a factory assembly machine instruction handbook. The handbook instructs the machine where to find raw components (paths), how to treat non-metal sheets (transformers), and defines what percentage of products must pass validation checkstops before shipping (coverage thresholds).

### Code Example
```javascript
// jest.config.js
module.exports = {
  testEnvironment: "node",
  transform: {
    "^.+\\.(t|j)sx?$": "@swc/jest",
  },
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1"
  },
  coverageThreshold: {
    global: { branches: 80, functions: 80, lines: 80 }
  }
};
```

### Common Interview Questions
- Why is configuring the testEnvironment parameter (node vs jsdom) critical?
- How do you use moduleNameMapper to resolve path aliases in tests?
- What are coverage thresholds and how do they enforce build pipeline safety?

### Reference Links
- [Jest: Configuration Reference](https://jestjs.io/docs/configuration)

## Mocking Functions with Jest

### Definition
Mocking replaces real functions or modules with spy hooks, allowing you to capture arguments, control return values, mock asynchronous promises (mockResolvedValue), and isolate the code unit being tested.

### Real-world Analogy
Imagine testing an automated teller machine (ATM). Instead of connecting the ATM to the real banking network and moving real cash (expensive and slow), you plug it into a simulator box (mock). When the ATM asks "Verify account balance", the simulator returns "$500" instantly.

### Code Example
```javascript
const api = require("./api");
jest.mock("./api"); // Mock the entire module

test("fetches user data securely", async () => {
  api.getUser.mockResolvedValue({ id: 101, name: "Alice" });
  
  const result = await api.getUser(101);
  expect(result.name).toBe("Alice");
  expect(api.getUser).toHaveBeenCalledWith(101);
});
```

### Common Interview Questions
- What is the difference between jest.fn(), jest.spyOn(), and jest.mock()?
- Compare mockClear, mockReset, and mockRestore. When do you use each?
- How do you mock timers (e.g. setTimeout) using jest.useFakeTimers()?

### Reference Links
- [Jest: Mock Functions](https://jestjs.io/docs/mock-functions)
- [Jest: Mocking Modules](https://jestjs.io/docs/manual-mocks)

## Vitest

### Definition
Vitest is a Vite-native testing framework. It features fast execution times by leveraging Vite's dev server, out-of-the-box ESM and TypeScript support, and exports a Jest-compatible API.

### Real-world Analogy
If Jest is a reliable family station wagon, Vitest is a modern electric sports car built on the same frame. It starts instantly, uses the same controls and dashboard dials, but accelerates much faster because of its lightweight engine.

### Code Example
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: true, // Enables describe, test, expect without imports
    environment: "jsdom",
  },
});
```

### Common Interview Questions
- Why is Vitest faster than Jest in projects that already use Vite as a bundler?
- What is in-source testing in Vitest and when is it useful?
- How does Vitest's UI mode improve the developer feedback loop?

### Reference Links
- [Vitest: Official Guide](https://vitest.dev/guide/)

## Integration Testing

### Definition
Integration testing is the phase of software testing where individual software modules are combined and tested as a group, ensuring that they interact correctly and databases or APIs sync as expected.

### Real-world Analogy
Imagine assembling a sink. You verify the faucet and the pipe threads fit independently. An integration test is connecting the faucet to the water pipe, turning the handle, and checking if water flows into the drain without leaks.

### Code Example
```javascript
// Basic integration test structure using real modules
const UserService = require("./UserService");
const UserRepository = require("./UserRepository");

test("creates user and verifies database entry", async () => {
  const repo = new UserRepository();
  const service = new UserService(repo);
  const user = await service.register("test@test.com");
  
  const stored = await repo.findById(user.id);
  expect(stored.email).toBe("test@test.com");
});
```

### Common Interview Questions
- How does integration testing differ from unit testing?
- What is a test double and what are the types (stub, spy, mock, fake)?
- How do you isolate tests when sharing a physical development database?

### Reference Links
- [Martin Fowler: Integration Test](https://martinfowler.com/bliki/IntegrationTest.html)

## Supertest for API Testing

### Definition
Supertest is a library used to test Node.js HTTP servers. It acts as an HTTP client, sending requests (GET, POST, etc.) to your Express app routes and verifying status codes, response headers, and JSON payloads.

### Real-world Analogy
Supertest is like hiring a mock customer inspector to test a drive-thru window. The inspector drives up, orders a specific menu item, checks the price on the receipt (status code), and verifies the bag contains the correct food weight (response body).

### Code Example
```javascript
const request = require("supertest");
const app = require("./app"); // Express app (not listening yet)

test("GET /api/users returns list", async () => {
  const response = await request(app)
    .get("/api/users")
    .expect("Content-Type", /json/)
    .expect(200);

  expect(Array.isArray(response.body.users)).toBe(true);
});
```

### Common Interview Questions
- Why should you export your Express app without calling app.listen() before importing it into Supertest?
- How do you handle authentication (e.g. sending bearer tokens or cookies) in Supertest requests?
- How do you assert error responses (like 400 Bad Request) using Supertest?

### Reference Links
- [Supertest GitHub Repository](https://github.com/ladjs/supertest)

## In-Memory MongoDB for Testing

### Definition
In-Memory MongoDB (mongodb-memory-server) runs a real MongoDB instance entirely in local system RAM. It speeds up integration testing by avoiding external network hops and guarantees a clean database state for each test run.

### Real-world Analogy
Imagine testing a vending machine. Instead of rolling the heavy metal machine into your office and plugging it into the wall, you play with a small, identical miniature model made of plastic that operates in your lap and can be crushed and replaced in seconds.

### Code Example
```javascript
const mongoose = require("mongoose");
const { MongoMemoryServer } = require("mongodb-memory-server");

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const uri = mongoServer.getUri();
  await mongoose.connect(uri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});
```

### Common Interview Questions
- Why is an in-memory database preferred over a production staging database for CI pipelines?
- How do you clean up database collections between individual test runs to prevent data contamination?
- Explain the role of globalSetup and globalTeardown hooks in Jest when configuring memory servers.

### Reference Links
- [mongodb-memory-server GitHub](https://github.com/nodkz/mongodb-memory-server)

## React Testing Library (RTL)

### Definition
React Testing Library is a set of utilities for testing React components based on their DOM output rather than internal implementation details. It encourages queries based on accessibility markers (getByRole, getByLabelText).

### Real-world Analogy
RTL is like checking a television set. You do not open the case to measure wire voltages (implementation). You sit on the couch, press the power button on the remote, and verify if the correct channel image appears on the screen (behavior).

### Code Example
```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { SubmitButton } from "./SubmitButton";

test("renders button and triggers click callback", async () => {
  const handleClick = jest.fn();
  render(<SubmitButton onClick={handleClick} label="Submit" />);

  const button = screen.getByRole("button", { name: /submit/i });
  expect(button).toBeInTheDocument();

  await userEvent.click(button);
  expect(handleClick).toHaveBeenCalledTimes(1);
});
```

### Common Interview Questions
- Why does RTL encourage querying by accessibility role instead of using CSS selectors or test IDs?
- What is the difference between getBy, queryBy, and findBy query variants?
- Why is the user-event library preferred over fireEvent for simulating interactions?

### Reference Links
- [React Testing Library: Queries](https://testing-library.com/docs/queries/about)
- [React Testing Library: Guiding Principles](https://testing-library.com/docs/guiding-principles)

## Testing React Hooks

### Definition
Testing React Hooks involves isolating custom hooks from component code. The renderHook utility mounts a wrapper component to run the hook, returning a 'result' ref and an 'act' utility to wrap state-updating operations.

### Real-world Analogy
Testing a hook is like testing a car's engine control chip on a diagnostic test stand. Instead of placing the chip in a completed car and driving it on a track, you plug it into a standalone testing socket (renderHook) and read output voltages directly.

### Code Example
```javascript
import { renderHook, act } from "@testing-library/react";
import { useState } from "react";

function useCounter() {
  const [count, setCount] = useState(0);
  const increment = () => setCount(prev => prev + 1);
  return { count, increment };
}

test("should increment counter state", () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

### Common Interview Questions
- Why do you need to wrap state-modifying hook calls in the act() wrapper?
- How do you test a custom hook that depends on Context Providers (e.g. wrapper options)?
- Describe how to test hooks that perform asynchronous updates.

### Reference Links
- [React Testing Library: Hook Testing](https://testing-library.com/docs/react-testing-library/api#renderhook)

## MSW (Mock Service Worker)

### Definition
Mock Service Worker (MSW) is an API mocking library that intercepts outgoing network requests at the browser or Node.js network layer using Service Workers (in browsers) or request interception (in Node), returning mock responses defined in mock handler files.

### Real-world Analogy
MSW is like setting up a dummy mail redirection box at the post office. When the application mails a letter to api.example.com, the redirection box intercepts it, reads the envelope, prints a pre-made mock answer, and mails it back, keeping the real servers out of the loop.

### Code Example
```typescript
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

// Define request handlers
export const handlers = [
  http.get("https://api.com/user", () => {
    return HttpResponse.json({ name: "Alice" });
  })
];

const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterAll(() => server.close());
```

### Common Interview Questions
- Why is MSW preferred over stubbing global.fetch or mocking axios modules?
- Explain the difference between running MSW in the browser vs running it in Node.js unit tests.
- How do you configure dynamic mock responses or error overrides in individual tests?

### Reference Links
- [MSW: Getting Started](https://mswjs.io/docs/getting-started)

## Snapshot Tests

### Definition
Snapshot tests take a serialized render tree of a component and compare it against a reference snapshot file saved alongside the test. The test fails if the output does not match the saved snapshot.

### Real-world Analogy
Imagine taking a high-definition photograph of a garden on day 1 (reference snapshot). On day 10, you take another photo and compare them pixel-by-pixel. If a single flower has moved, the comparison alarm triggers, forcing you to verify if the movement was deliberate.

### Code Example
```jsx
import { render } from "@testing-library/react";
import { Title } from "./Title";

test("renders title correctly matching snapshot", () => {
  const { container } = render(<Title text="Hello World" />);
  expect(container.firstChild).toMatchSnapshot();
});
```

### Common Interview Questions
- What are the drawbacks of relying heavily on snapshot testing?
- How do you update a failing Jest snapshot file?
- When are inline snapshots (toMatchInlineSnapshot) preferred over separate snapshot files?

### Reference Links
- [Jest: Snapshot Testing](https://jestjs.io/docs/snapshot-testing)

## Playwright

### Definition
Playwright is a modern E2E testing framework that automates Chromium, Firefox, and WebKit browsers. It features native auto-waiting assertions, parallel execution, isolated browser contexts, and trace viewing.

### Real-world Analogy
Playwright is like a robot sitting in front of a computer. The robot physically opens Google Chrome, types a username in the input field, clicks submit, and waits patiently for the dashboard screen to load before asserting success.

### Code Example
```typescript
// playwright.config.ts / sample test
import { test, expect } from "@playwright/test";

test("has title and navigates to dashboard", async ({ page }) => {
  await page.goto("https://myApp.com/login");
  
  await page.getByLabel("Username").fill("admin");
  await page.getByRole("button", { name: "Login" }).click();
  
  await expect(page).toHaveURL(/.*dashboard/);
});
```

### Common Interview Questions
- How does Playwright's auto-waiting mechanism prevent flaky E2E tests?
- Explain the concept of Browser Contexts in Playwright and why they make tests fast.
- What is the Page Object Model (POM) and how does it improve test maintainability?

### Reference Links
- [Playwright: Getting Started](https://playwright.dev/docs/intro)
- [Playwright: Page Object Models](https://playwright.dev/docs/pom)

## Cypress

### Definition
Cypress is a frontend E2E testing framework that executes inside the browser process, providing a developer test runner, real-time command logs, time-travel debugging, and built-in network stubbing.

### Real-world Analogy
Cypress is like an inspection console built directly into the dashboard of a car. As the car drives, the console captures engine logs, record video feeds, and allows the mechanic to pause execution and inspect parts inline.

### Code Example
```javascript
// cypress.config.js / sample test
describe("Login Flow", () => {
  it("logs in successfully", () => {
    cy.visit("/login");
    cy.get("input[name=username]").type("admin");
    cy.get("button").click();
    cy.url().should("include", "/dashboard");
  });
});
```

### Common Interview Questions
- Why does Cypress run inside the browser and how does this affect cross-origin navigation limitations?
- What are custom Cypress commands and where do you define them?
- How does cy.intercept work for mocking API requests in Cypress?

### Reference Links
- [Cypress Documentation](https://docs.cypress.io/)

## Scenario-based Interview Questions for Testing

### Scenario 1
Your continuous integration (CI) test suite is flaky: some tests fail randomly on GitHub Actions but always pass locally on developer machines. How do you diagnose and fix this?

*Expected Approach:*
1. Explain that flaky tests are often caused by asynchronous timing issues (like tests finishing before an API mock resolves), sharing global state, or timezone offsets on CI.
2. Investigate by looking for hardcoded timeouts (replace with async wait helpers like RTL's waitFor or Playwright's auto-wait assertions).
3. Ensure each test cleans up its database and mocks (use mockRestore or database rollback fixtures) to prevent side-effects.

### Scenario 2
You are asked to design a testing strategy for a new e-commerce application. The budget is tight, and you have to deliver a reliable system in a short timeframe. How do you distribute the test suites?

*Expected Approach:*
1. Propose following the Testing Pyramid: write a wide base of unit tests for helper functions and utility calculators (high speed, low cost).
2. Create integration tests targeting critical API routes using Supertest and in-memory databases (ensuring controllers and models communicate).
3. Implement 3-5 E2E tests covering the most critical revenue flow: adding items to the cart and completing checkout. This guarantees the checkout flow works without the cost of writing 100 flaky E2E tests.
