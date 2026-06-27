# 05 Typescript

This guide covers TypeScript core mechanics, type-level programming, utilities, narrowing, and integration with React, Node.js, and ORMs.

## TypeScript Fundamentals

### Definition
TypeScript is a strongly typed programming language that builds on JavaScript, adding static type definition capabilities. The TypeScript compiler (tsc) compiles TypeScript code into browser-compatible JavaScript using configuration options defined in a tsconfig.json file.

### Real-world Analogy
Imagine building a Lego tower using a blueprint. JavaScript is assembling blocks by sight alone; you might realize late that a red block does not fit. TypeScript is a smart template tray: you can only place a brick in the slot if it matches the slot shape.

### Code Example
```json
// Common tsconfig.json options
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src"
  }
}
```

### Common Interview Questions
- What is the role of the strict flag in tsconfig.json, and name three checks it enables.
- Explain the compilation process from TypeScript AST down to JavaScript.
- What does esModuleInterop do for CommonJS modules?

### Reference Links
- [TypeScript Docs: What is TypeScript](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Docs: tsconfig.json](https://www.typescriptlang.org/tsconfig)

## Types and Interfaces

### Definition
Types and Interfaces define object shapes. Interfaces can be extended via inheritance and support declaration merging (appending properties to existing declarations), whereas Types support unions, intersections, and mapping, but cannot be merged.

### Real-world Analogy
An Interface is like a physical contract for a house: you can write amendments at the bottom to append rules (merging). A Type is like a chemical formula: you can combine elements together to make new materials (unions/intersections), but you cannot change the base definition once written.

### Code Example
```typescript
interface User {
  id: string;
  name: string;
}

// Declaration merging (exclusive to interface)
interface User {
  role?: string;
}

type Admin = User & {
  permissions: string[];
};
```

### Common Interview Questions
- Compare Interfaces and Type Aliases regarding declaration merging and unions.
- What is an index signature and when do you use it?
- How do you create readonly properties in interfaces?

### Reference Links
- [TypeScript Docs: Types vs Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)

## Primitive and Complex Types

### Definition
Primitive types cover basic values (string, number, boolean, null, undefined). Complex types include arrays, tuples (fixed-length arrays), enums, literal types, and the types 'any' (disables type check), 'unknown' (safe type check), and 'never' (impossible values).

### Real-world Analogy
Primitives are individual coins and bills. A Tuple is a cash-register coin tray designed to hold exactly one nickel, one dime, and one quarter in that order. The 'unknown' type is a mystery package: you must scan it at security before opening, while 'any' is a package you pass through without checking.

### Code Example
```typescript
let status: "active" | "inactive" = "active"; // Literal union
let coordinates: [number, number] = [40.7128, -74.0060]; // Tuple

let value: unknown = "Hello";
// Safe check before use
if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

### Common Interview Questions
- Why is 'unknown' preferred over 'any' in type-safe applications?
- What is the 'never' type and where is it used (e.g. exhaustiveness checks)?
- Explain const assertions (as const) and how they lock object properties as literals.

### Reference Links
- [TypeScript Docs: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Docs: More on Types](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

## Functions in TypeScript

### Definition
Functions in TypeScript define types for parameters and return values. They support optional parameters, default parameters, rest parameters, arrow function signatures, and function overloading to declare multiple call signatures.

### Real-world Analogy
Think of a vending machine input slot. The machine defines parameter inputs: it accepts a nickel, a dime, or a dollar bill. It also guarantees a return type: a carbonated drink or a bottle of water. If you insert a wooden token, it rejects it instantly.

### Code Example
```typescript
// Function with type annotations and optional parameters
function greet(user: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${user}`;
}

// Function overloading
function processInput(input: string): string[];
function processInput(input: number): number;
function processInput(input: any): any {
  if (typeof input === "string") return input.split("");
  return input;
}
```

### Common Interview Questions
- How do you implement function overloading in TypeScript?
- What is the difference between a function type expression and a call signature?
- How do you write typing rules for REST callbacks?

### Reference Links
- [TypeScript Docs: More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)

## Generics

### Definition
Generics are type variables that allow you to write reusable components that operate over a variety of types rather than a single one. They maintain type relationship links between function parameters and return values.

### Real-world Analogy
Think of a shipping shipping box. Instead of building one box designed only to hold apples and a different box designed only to hold books, you make a generic shipping box <T> that securely holds whatever type of cargo you place inside it.

### Code Example
```typescript
function getFirstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const numbers = [1, 2, 3];
const firstNumber = getFirstElement(numbers); // Inferred as number

// Generic constraints
interface HasId { id: string }
function logItem<T extends HasId>(item: T) {
  console.log(item.id);
}
```

### Common Interview Questions
- How do you write generic constraints using the extends keyword?
- What is the default type parameter and when is it useful?
- Explain the typing details of the built-in Promise<T> generic.

### Reference Links
- [TypeScript Docs: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)

## Utility Types

### Definition
Utility types are built-in type transformations that construct new types from existing ones. Examples include Partial (optional properties), Required (non-optional), Readonly (constant), Pick (select subset), Omit (exclude subset), and Record (key-value mapping).

### Real-world Analogy
Think of a photo filter app. You start with a raw photo (base type). You apply a filter to crop it (Omit/Pick), make it black and white (Readonly), or blur some background elements (Partial) to produce a new version.

### Code Example
```typescript
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// Make all fields optional
type UpdateTaskInput = Partial<Task>;

// Pick only title and completed
type TaskPreview = Pick<Task, "title" | "completed">;

// Omit ID
type CreateTaskInput = Omit<Task, "id">;
```

### Common Interview Questions
- What is the difference between Pick<T, K> and Omit<T, K>?
- How do you implement a custom type helper similar to ReturnType<T>?
- Describe how Exclude<T, U> works under the hood.

### Reference Links
- [TypeScript Docs: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## Type Narrowing

### Definition
Type narrowing is the process of moving from a broad type to a more specific type using runtime checks. Tools include typeof, instanceof, the 'in' operator, discriminated unions (matching field tags), and custom type predicates (is keyword).

### Real-world Analogy
Imagine a security officer checking credentials. They look at a general ID badge. If it lists "Admin Role" (discriminated union), they guide the visitor through the secure door. If it is a generic keycard, they test it on the scanner (instanceof) before opening.

### Code Example
```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
  // Discriminated union narrowing
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
  }
}
```

### Common Interview Questions
- What is a type predicate function and how do you write its return value (parameterName is Type)?
- Explain how a discriminated union works in TypeScript switch statements.
- How does the 'in' operator narrow a type containing optional properties?

### Reference Links
- [TypeScript Docs: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)

## Type Assertions

### Definition
Type assertions allow you to tell the compiler to treat a value as a specific type, bypassing static inference. Assertions use the 'as' keyword or angle brackets, but do not perform any runtime casting.

### Real-world Analogy
Imagine a movie set containing prop food. If you tell the director "treat this fake apple as a real apple" (type assertion), the director will let the actor hold it. However, if the actor actually tries to bite into it, they will break their teeth (runtime crash).

### Code Example
```typescript
const canvas = document.getElementById("main-canvas") as HTMLCanvasElement;

// Double assertions for unrelated types (dangerous)
const val = ("hello" as unknown) as number;
```

### Common Interview Questions
- What is the difference between type assertion and type casting?
- Why is using type assertions dangerous and what are the alternatives?
- When are type assertions absolutely necessary (e.g. canvas elements, DOM operations)?

### Reference Links
- [TypeScript Docs: Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)

## TypeScript with React

### Definition
TypeScript integration in React ensures type-safe component props, state variables, references, and event handlers using built-in React type declarations (FC, ReactNode, ChangeEvent, FormEvent).

### Real-world Analogy
Think of building a car on an assembly line. Every connector plug has a key shape: you can only plug the dashboard console cable into the console receiver if the pins align exactly to the specification sheet.

### Code Example
```tsx
import React, { useState, useRef } from "react";

interface ButtonProps {
  label: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
}

export const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  const [clickCount, setClickCount] = useState<number>(0);
  const buttonRef = useRef<HTMLButtonElement>(null);

  const handleInnerClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    setClickCount(prev => prev + 1);
    onClick(e);
  };

  return (
    <button ref={buttonRef} onClick={handleInnerClick}>
      {label} ({clickCount})
    </button>
  );
};
```

### Common Interview Questions
- What is the difference between typing children props as ReactNode vs ReactElement?
- How do you type event handlers like onChange for text inputs and select elements?
- How do you type useRef when using it for DOM reference vs using it as a mutable value container?

### Reference Links
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)

## TypeScript with Express

### Definition
TypeScript with Express requires installing type definitions (@types/express) and typing Request, Response, and NextFunction parameters. Developers extend the Request interface to support custom properties like req.user.

### Real-world Analogy
Think of a delivery driver route map. The logistics system registers the truck's cargo inventory (Request object). If a custom package (authenticated user metadata) is added at a warehouse checkstop (middleware), the delivery sheet must update its definitions to list the new item.

### Code Example
```typescript
import express, { Request, Response, NextFunction } from "express";

// Extend Request interface using declaration merging
declare global {
  namespace Express {
    interface Request {
      user?: { id: string; role: string };
    }
  }
}

const app = express();

const authMiddleware = (req: Request, res: Response, next: NextFunction) => {
  req.user = { id: "123", role: "admin" };
  next();
};

app.get("/profile", authMiddleware, (req: Request, res: Response) => {
  res.json({ userId: req.user?.id });
});
```

### Common Interview Questions
- How do you add a custom parameter (like user or files) to the Express Request interface in TypeScript?
- Why is it recommended to use NextFunction inside async Express route handlers?
- How do you type Express error-handling middleware?

### Reference Links
- [TypeScript Docs: Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)
- [Express with TypeScript Setup Guide](https://blog.logrocket.com/how-to-set-up-node-js-with-express-and-typescript/)

## TypeScript with Mongoose

### Definition
TypeScript with Mongoose uses type declarations to ensure document structures match schemas. Modern Mongoose recommends defining an interface containing document properties and passing it to Schema and Model constructors.

### Real-world Analogy
Think of a registry filing cabinet. The cabinet blueprint (Mongoose Schema) defines folder tab slots. The TypeScript interface is the strict label printer: it prevents you from slipping folders with invalid shapes into the cabinet slots.

### Code Example
```typescript
import { Schema, model, Document } from "mongoose";

interface IUser {
  email: string;
  age: number;
}

const userSchema = new Schema<IUser>({
  email: { type: String, required: true },
  age: { type: Number, required: true }
});

const User = model<IUser>("User", userSchema);
```

### Common Interview Questions
- How do you extract model types and document types from Mongoose models?
- What are the differences between Mongoose schemas and TypeScript interfaces?
- How do you implement schema validations while retaining type safety?

### Reference Links
- [Mongoose: TypeScript Integration](https://mongoosejs.com/docs/typescript.html)

## TypeScript with Prisma

### Definition
Prisma is an ORM that automatically generates TypeScript types from your schema.prisma file. Every query, filter, and database transaction returned by Prisma Client is typed automatically.

### Real-world Analogy
Imagine a magic database catalog. You write down the database structure once on a master sheet (schema.prisma). A digital print machine automatically manufactures custom boxes, tags, and labels (generated types) that fit every item in the catalog.

### Code Example
```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function getUsers() {
  // Prisma generates return type based on selection query
  const users = await prisma.user.findMany({
    select: {
      id: true,
      email: true
    }
  });
  return users;
}
```

### Common Interview Questions
- How do you import specific model payload types generated by Prisma (e.g. Prisma.UserGetPayload)?
- How does Prisma achieve type safety for dynamic select queries without requiring manual interface changes?
- What command compiles your schema.prisma file and regenerates types?

### Reference Links
- [Prisma Docs: TypeScript](https://www.prisma.io/docs/concepts/components/prisma-client/type-safety)

## TypeScript Configuration for Full-stack

### Definition
Full-stack monorepos utilize separate tsconfig.json configurations for frontend (React) and backend (Node.js) development, using path aliases to organize modules.

### Real-world Analogy
Imagine a factory building cars. The metal cutting station (backend) has its own machine instructions (tsconfig.node.json). The detailing and dashboard station (frontend) has a different set of instructions (tsconfig.react.json) suitable for dashboard assemblies.

### Code Example
```json
// tsconfig.json (Extending base configurations)
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "paths": {
      "@server/*": ["./src/server/*"],
      "@shared/*": ["./src/shared/*"]
    }
  }
}
```

### Common Interview Questions
- Why do you need separate tsconfig files for frontend and backend in a monorepos?
- What is path mapping and how do you resolve path aliases during build compilation?
- Explain the role of the references config field in tsconfig.json.

### Reference Links
- [TypeScript Docs: Project References](https://www.typescriptlang.org/docs/handbook/project-references.html)

## Common TypeScript Mistakes and Anti-patterns

### Definition
Common TypeScript mistakes include overusing any, abusing non-null assertions (!), using type assertions instead of narrowing, ignoring strict compiler flags, and excessive type nesting.

### Real-world Analogy
Overusing 'any' is like locking a secure gate but leaving a giant, unlocked dog door next to it: anyone can walk through without checking, defeating the purpose of the security gate.

### Code Example
```typescript
// Anti-pattern: Overusing any
function logData(data: any) {
  // Bad: disables compiler protection
  console.log(data.nonExistentField.name);
}

// Correct: Use unknown or specific shapes
function logDataSafe(data: unknown) {
  if (data && typeof data === "object" && "name" in data) {
    console.log((data as any).name);
  }
}
```

### Common Interview Questions
- Why is using noImplicitAny: false an anti-pattern?
- What is a non-null assertion operator (!) and why should you avoid it?
- How does noUncheckedIndexedAccess protect dictionary lookups?

### Reference Links
- [TypeScript Docs: Do's and Don'ts](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)

## Scenario-based Interview Questions for TypeScript

### Scenario 1
You are building an application with a role-based action matrix. Admin has permissions, Guest has view rights, and Manager has permissions and team size. You write a single function that checks attributes by using optional values. How do you implement this in a type-safe manner?

*Expected Approach:*
1. Recommend using a Discriminated Union with a common tag field (e.g. type: "admin" | "guest" | "manager").
2. Inside the processing function, use switch on type to narrow parameters down to specific interface shapes.
3. This prevents invalid property configurations (like a Guest having team size details) and gives autocomplete.

### Scenario 2
You are integrating a third-party legacy package that has no TypeScript definition files (@types/package-name does not exist). The build compiler fails when you try to import it. How do you resolve this?

*Expected Approach:*
1. Explain that the compiler throws an error because it cannot find the module declaration.
2. Fix this by creating a global declarations definition file (e.g. global.d.ts) in your project.
3. Add declare module "legacy-package-name" inside it to type-satisfy the compiler (which treats imports from it as type any), or write custom basic typings inside it for essential methods.
