# 03 React

This file covers core React concepts from rendering mechanics and component lifecycles to state management, optimization, and React 19 changes.

## Virtual DOM and Reconciliation

### Definition
The Virtual DOM is a lightweight, in-memory representation of the real DOM. Reconciliation is the algorithm React uses to diff the virtual representation with the actual UI and apply only the minimal necessary updates to the real DOM.

### Real-world Analogy
Imagine editing a blueprint of a house. Instead of tearing down the physical house and rebuilding it every time you want to move a window, you update the blueprint first, compare the changes, and send a builder to move only that single window.

### Code Example
```jsx
// Keys help React identify which items have changed, been added, or been removed
function UserList({ users }) {
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Common Interview Questions
- How does React's Fiber architecture enable incremental rendering and scheduling?
- Why should you avoid using array indexes as keys in dynamic lists?
- What are the main steps in the diffing algorithm (e.g. element type changes vs attribute updates)?

### Reference Links
- [React Docs: Reconciliation](https://legacy.reactjs.org/docs/reconciliation.html)
- [React Architecture: Fiber Deep Dive](https://github.com/acdlite/react-fiber-architecture)

## JSX Compilation

### Definition
JSX is a syntax extension to JavaScript that allows you to write HTML-like structures in React. At build time, tools like Babel compile JSX tags into standard React.createElement (or new jsx runtime) function calls.

### Real-world Analogy
Imagine a shorthand notebook style used by decorators. They write "blue_chair" instead of writing a detailed contract specifying: "Create a chair object, set material to wood, set color to blue". The assistant translates the shorthand into the full contract before sending it to the factory.

### Code Example
```jsx
// Written in JSX
const element = <h1 className="title">Hello World</h1>;

// Compiled JS output (pre-React 17 template)
const compiledElement = React.createElement(
  "h1",
  { className: "title" },
  "Hello World"
);
```

### Common Interview Questions
- What does Babel do when it encounters JSX tags in a code file?
- How does the new JSX Transform (React 17+) differ from older React.createElement methods?
- Can you write a valid React component without using JSX?

### Reference Links
- [React Docs: Introducing JSX](https://react.dev/learn/writing-markup-with-jsx)
- [Babel Docs: React Preset](https://babeljs.io/docs/babel-preset-react)

## Component Lifecycle

### Definition
Component lifecycle refers to the stages a component goes through: Mount (created and added to the DOM), Update (re-rendered due to prop or state changes), and Unmount (removed from the DOM).

### Real-world Analogy
Think of a store kiosk. Mount is setting up the kiosk and opening the register. Update is replenishing products or changing signs on display. Unmount is closing down the kiosk and cleaning up the space.

### Code Example
```jsx
import { useEffect } from "react";

function LifecycleDemo() {
  useEffect(() => {
    console.log("Mounted");
    return () => {
      console.log("Unmounting (Cleanup)");
    };
  }, []); // Empty dependency array matches mount and unmount

  return <div>Component Lifecycle</div>;
}
```

### Common Interview Questions
- Which hook replaces componentDidMount, componentDidUpdate, and componentWillUnmount?
- What is the difference between rendering phases and committing phases in React lifecycle?
- Why should you perform cleanup operations inside the hook return function?

### Reference Links
- [React Docs: Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [React Lifecycle Diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)

## useState Deep Dive

### Definition
useState is a Hook that declares state variables in functional components. It schedules re-renders and handles batching, functional updates, and lazy initialization.

### Real-world Analogy
Imagine keeping score on a tally board. If you want to add 3 points, you don't write "3" immediately; you look at the current score, add 3 to it, and write the new total. If you do it in quick succession, you batch the updates to show only the final score.

### Code Example
```jsx
import { useState } from "react";

function Counter() {
  // Lazy initialization (runs only on mount)
  const [count, setCount] = useState(() => {
    return 0;
  });

  const increment = () => {
    // Functional update ensures correct value from previous state
    setCount(prev => prev + 1);
  };

  return <button onClick={increment}>Count: {count}</button>;
}
```

### Common Interview Questions
- What is state batching and how does React handle multiple setState calls in a single event loop tick?
- When should you use lazy initialization inside useState?
- Why can setting state to the exact same value cause a component to skip re-rendering?

### Reference Links
- [React Docs: useState](https://react.dev/reference/react/useState)
- [React Docs: State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)

## useEffect Complete Guide

### Definition
useEffect is a Hook that runs side effects (fetching data, timers, manual DOM updates) in functional components after rendering. It schedules a cleanup function before the effect runs again or on unmount.

### Real-world Analogy
Think of subscribing to a monthly magazine. When you sign up (effect runs), you get magazines. If you move houses (dependencies change), you must unsubscribe from the old address (cleanup function) before subscribing to the new one.

### Code Example
```jsx
import { useState, useEffect } from "react";

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let isCancelled = false; // Prevent race conditions

    async function fetchData() {
      const res = await fetch(`https://api.com/users/${userId}`);
      const data = await res.json();
      if (!isCancelled) {
        setUser(data);
      }
    }

    fetchData();

    return () => {
      isCancelled = true; // Cleanup cancels state update on unmount or id change
    };
  }, [userId]);

  return <div>{user ? user.name : "Loading..."}</div>;
}
```

### Common Interview Questions
- How do you prevent race conditions in useEffect when fetching data?
- Compare useEffect and useLayoutEffect. When should you use useLayoutEffect?
- What are the rules of hooks and what happens if you put useEffect inside an if statement?

### Reference Links
- [React Docs: useEffect](https://react.dev/reference/react/useEffect)
- [React Docs: You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)

## useContext

### Definition
useContext is a Hook that allows components to subscribe to React context without nesting. It allows data sharing across the component tree, but can trigger re-renders in all consumers when the context value changes.

### Real-world Analogy
Think of a radio station broadcast. Instead of running a copper wire from the station to every house in town (passing props down 10 layers), the station broadcasts a signal. Any house with a radio tuner (useContext) can listen directly.

### Code Example
```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext(null);

function App() {
  const [theme, setTheme] = useState("dark");
  return (
    <ThemeContext.Provider value={theme}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  const theme = useContext(ThemeContext);
  return <div>Current theme: {theme}</div>;
}
```

### Common Interview Questions
- When should you choose context splitting over a single context object?
- What are the performance implications of updating a context provider value?
- How do you optimize context consumers to avoid unnecessary re-renders (e.g. useMemo)?

### Reference Links
- [React Docs: useContext](https://react.dev/reference/react/useContext)
- [React Docs: Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)

## useReducer

### Definition
useReducer is a Hook used to manage complex state transitions in components. It accepts a reducer function and an initial state, returning the current state and a dispatch function.

### Real-world Analogy
Imagine a traffic control tower. Instead of every individual gate manager deciding when to open and close lanes, they send a request status code (action) to the control tower dispatcher (reducer), which modifies the grid status in an ordered sequence.

### Code Example
```jsx
import { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "increment": return { count: state.count + 1 };
    case "decrement": return { count: state.count - 1 };
    default: throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
    </>
  );
}
```

### Common Interview Questions
- When is useReducer preferred over useState?
- What is a reducer function and why must it be a pure function?
- How do you pass dispatch down through context instead of callbacks?

### Reference Links
- [React Docs: useReducer](https://react.dev/reference/react/useReducer)
- [React Docs: Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer)

## useRef

### Definition
useRef is a Hook that returns a mutable ref object whose .current property persists across renders. It does not trigger a re-render when its value changes, and is commonly used for DOM access and tracking mutable values.

### Real-world Analogy
Think of a notebook in your pocket. You can write notes, cross them out, and update numbers anytime you want (mutating .current). The rest of the world (re-renders) does not care or notice when you write in your notebook.

### Code Example
```jsx
import { useRef, useEffect } from "react";

function FocusInput() {
  const inputRef = useRef(null);

  const handleClick = () => {
    inputRef.current.focus(); // Access DOM node directly
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={handleClick}>Focus input</button>
    </>
  );
}
```

### Common Interview Questions
- Why does modifying ref.current not trigger a component re-render?
- How do you use a ref to store the previous value of a state variable?
- How do you forward refs to custom child components (forwardRef)?

#### Reference Links
- [React Docs: useRef](https://react.dev/reference/react/useRef)
- [React Docs: Referencing Values with Refs](https://react.dev/learn/referencing-values-with-refs)

## useMemo and useCallback

### Definition
useMemo is a Hook that caches the result of a calculation between re-renders. useCallback is a Hook that caches a function definition itself between re-renders, preventing function recreation.

### Real-world Analogy
useMemo is like writing down the answer to a long math calculation so you do not have to recalculate it. useCallback is like keeping a copy of a recipe card so you do not have to rewrite the instructions from scratch every time you prepare a meal.

### Code Example
```jsx
import { useState, useMemo, useCallback } from "react";

function CalculationDemo({ list }) {
  const [filter, setFilter] = useState("");

  // Cached calculation
  const filteredList = useMemo(() => {
    return list.filter(item => item.includes(filter));
  }, [list, filter]);

  // Cached function callback
  const handleSelect = useCallback((item) => {
    console.log("Selected: ", item);
  }, []);

  return <Child list={filteredList} onSelect={handleSelect} />;
}
```

### Common Interview Questions
- What is the difference between useMemo and useCallback?
- Why is using these hooks on every variable/function an anti-pattern?
- How does React.memo interact with useCallback to prevent child component re-renders?

### Reference Links
- [React Docs: useMemo](https://react.dev/reference/react/useMemo)
- [React Docs: useCallback](https://react.dev/reference/react/useCallback)

## Custom Hooks

### Definition
Custom hooks are reusable JavaScript functions that encapsulate stateful logic, allowing you to share hook behaviors between different components. Custom hooks must follow the "use" naming convention.

### Real-world Analogy
Think of a power strip. Instead of wiring surge protection, indicator lights, and multiple outlets manually in every room, you bundle all that logic inside a portable power strip (custom hook) and plug it in wherever needed.

### Code Example
```jsx
import { useState, useEffect } from "react";

// Custom hook to track screen width
function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    if (media.matches !== matches) {
      setMatches(media.matches);
    }
    const listener = () => setMatches(media.matches);
    media.addEventListener("change", listener);
    return () => media.removeEventListener("change", listener);
  }, [matches, query]);

  return matches;
}
```

### Common Interview Questions
- Do two components using the same custom hook share state?
- How does custom hook naming affect lint rules and execution?
- Describe the custom hook implementation for debouncing state changes.

### Reference Links
- [React Docs: Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)

## React Router v6

### Definition
React Router v6 is a routing library for React. It introduces declarative nested routing, layout routes, outlet components, loaders for data prefetching, and actions for handling mutations.

### Real-world Analogy
Think of an office building building map. The main entrance gets you in the lobby (root route). The map directs you to specific floors, which have layouts with common hallways (nested routes and Outlets) leading to individual rooms.

### Code Example
```jsx
import { createBrowserRouter, RouterProvider, Outlet } from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,
    children: [
      { path: "dashboard", element: <Dashboard /> },
      { path: "profile", element: <Profile /> }
    ]
  }
]);

function Layout() {
  return (
    <div>
      <nav>Navbar</nav>
      <Outlet /> {/* Renders sub-routes here */}
    </div>
  );
}
```

### Common Interview Questions
- What is the purpose of the Outlet component in nested routing?
- How do loaders and actions in React Router v6 handle data prefetching and page transitions?
- Explain how protected routes are implemented in React Router v6.

### Reference Links
- [React Router: Main Docs](https://reactrouter.com/en/main)

## Form Handling

### Definition
Form handling in React can be controlled (state drives form input values) or uncontrolled (inputs manage their own state, and values are accessed via refs). Libraries like React Hook Form optimize performance by reducing re-renders.

### Real-world Analogy
Controlled handling is like a pilot using a flight controller that updates a dashboard screen before making flight adjustments. Uncontrolled handling is like driving a car where the steering wheel moves directly, and you only read the speedometer when you pass a speed trap.

### Code Example
```jsx
import { useForm } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  email: z.string().email(),
});

function FormDemo() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} />
      {errors.email && <span>{errors.email.message}</span>}
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Common Interview Questions
- Compare controlled vs uncontrolled form inputs regarding performance and updates.
- How does React Hook Form avoid re-rendering the whole form on every keystroke?
- Explain how to integrate Zod validation with React Hook Form.

### Reference Links
- [React Docs: Controlled Components](https://react.dev/reference/react-dom/components/input#controlling-an-input-with-a-state-variable)
- [React Hook Form Docs](https://react-hook-form.com/)

## State Management Comparison

### Definition
State management involves managing app-wide shared data. Context API is suitable for low-frequency updates; Redux Toolkit offers structured unidirectional flow; Zustand provides a minimalist hooks-based store; Jotai focuses on atomic state.

### Real-world Analogy
Context API is like writing news on the community notice board (everyone sees it, but changing it requires repainting). Redux Toolkit is like a corporate registry database with strict transaction logs. Zustand is like a shared spreadsheet that multiple colleagues can read and write to directly using links.

### Code Example
```javascript
// Zustand store example
import { create } from "zustand";

const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useStore((state) => state.bears);
  return <h1>Bears: {bears}</h1>;
}
```

### Common Interview Questions
- When should you choose Zustand over Redux Toolkit?
- Why is Context API not recommended for high-frequency state updates?
- Explain the concept of slice in Redux Toolkit.

### Reference Links
- [Zustand GitHub](https://github.com/pmndrs/zustand)
- [Redux Toolkit Docs](https://redux-toolkit.js.org/)

## Data Fetching Patterns

### Definition
Data fetching patterns handle state synchronization with remote servers. While basic useEffect works, caching libraries like TanStack Query (React Query) manage cache invalidation, background updates, retry states, and optimistic updates automatically.

### Real-world Analogy
useEffect fetching is like walking to the store to get milk every single morning. React Query is like having a refrigerator: you check the fridge first (cache check); if the milk is expired (stale), you use it but schedule a fresh delivery in the background (stale-while-revalidate).

### Code Example
```jsx
import { useQuery } from "@tanstack/react-query";

const fetchUsers = async () => {
  const res = await fetch("https://api.com/users");
  return res.json();
};

function UsersList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: fetchUsers,
  });

  if (isLoading) return <span>Loading...</span>;
  if (error) return <span>Error</span>;
  return <ul>{data.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### Common Interview Questions
- How does staleTime differ from gcTime (cacheTime) in TanStack Query?
- Explain how to implement optimistic updates during list mutations.
- How do you handle cache invalidation after performing a POST request?

### Reference Links
- [TanStack Query Docs](https://tanstack.com/query/latest)

## Error Boundaries

### Definition
Error Boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of crashing the application. They must be class components or use react-error-boundary.

### Real-world Analogy
Think of a firedoor in a building. If a fire starts in the kitchen (an error in a child component), the firedoor shuts automatically. The fire does not destroy the entire building, and people in the lobby can still see safety signs (fallback UI).

### Code Example
```jsx
import React from "react";

class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error("ErrorBoundary caught an error", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```

### Common Interview Questions
- Why can error boundaries only be created using class components (e.g. which lifecycle methods are missing in hooks)?
- What types of errors do React error boundaries fail to catch?
- How does getDerivedStateFromError differ from componentDidCatch?

### Reference Links
- [React Docs: Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)
- [GitHub: react-error-boundary](https://github.com/bvaughn/react-error-boundary)

## Performance Optimization

### Definition
Performance optimization in React involves minimizing unnecessary re-renders and reducing initial bundle size. Techniques include using React.memo, caching via useMemo/useCallback, virtualizing lists, and code splitting with lazy/Suspense.

### Real-world Analogy
Instead of printing the entire encyclopedia and carrying it to a meeting, you carry a tablet (lazy loading). When someone asks for page 52, you load only page 52 (Suspense), and cache the page in memory so you do not download it again (React.memo).

### Code Example
```jsx
import React, { lazy, Suspense } from "react";

// Code splitting
const HeavyComponent = lazy(() => import("./HeavyComponent"));

function App() {
  return (
    <Suspense fallback={<div>Loading component...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### Common Interview Questions
- How does list virtualization work to render a 10,000-item list efficiently?
- What are the rendering changes that occur when you wrap a component in React.memo?
- How do you analyze React bundle size using Webpack Bundle Analyzer or Vite bundle analyzer?

### Reference Links
- [React Docs: Performance](https://react.dev/reference/react/memo)

## Server Components vs Client Components

### Definition
React Server Components (RSC) render exclusively on the server, sending pre-compiled HTML and minimal client-side bundles to the browser. Client Components, marked with "use client", are sent to the client to add interactivity.

### Real-world Analogy
Imagine ordering a desk. A server component is buying a pre-assembled desk shipped in a crate (requires no tools from you). A client component is buying a flat-pack box (you must spend time and energy reading instructions and assembly at home).

### Code Example
```jsx
// Server Component (Default in Next.js App Router)
import { db } from "./db";

export default async function ProductList() {
  const products = await db.select().from("products"); // Direct DB access!
  return (
    <ul>
      {products.map(p => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

### Common Interview Questions
- What are the main benefits of React Server Components over client components?
- Why must you serialize props passed from server components to client components?
- Can a server component be nested inside a client component? How?

### Reference Links
- [React Docs: Server Components](https://react.dev/reference/rsc/server-components)

## React 19 Features

### Definition
React 19 introduces native support for async actions, the 'use' hook for inline promise/context resolution, optimistic state management hooks (useOptimistic), and automatic memoization via the compiler.

### Real-world Analogy
React 19 is like upgrading to a car with an automatic transmission. You no longer need to worry about manually shifting gears (no more manual useMemo and useCallback code additions); the engine handles optimal performance for you.

### Code Example
```jsx
// React 19 use hook resolving a promise during render
import { use } from "react";

function WeatherInfo({ dataPromise }) {
  const weather = use(dataPromise); // Resolves inline without useEffect
  return <div>Temperature: {weather.temp}</div>;
}
```

### Common Interview Questions
- Explain the role of the 'use' hook and how it differs from traditional hooks (like being allowed inside loops).
- What are React 19 Actions and how do they simplify state updates during form submissions?
- What does the React Compiler do and how does it affect useMemo/useCallback?

### Reference Links
- [React 19 Blog Post](https://react.dev/blog/2024/12/05/react-19)

## Common React Mistakes and Anti-patterns

### Definition
Common React mistakes include modifying state variables directly, ignoring dependency arrays in hooks, using unstable object references as keys, and causing layout shifts by incorrect component nesting.

### Real-world Analogy
Modifying state directly is like using a sharp marker to write a change directly on a finished wall paint job rather than mixing a fresh bucket of paint and painting the section. The room does not recognize the modification correctly.

### Code Example
```jsx
// Anti-pattern: Mutating state directly
const [user, setUser] = useState({ name: "Bob" });
// Bad: user.name = "Alice"; setUser(user);

// Correct pattern: Create a new object reference
setUser(prev => ({ ...prev, name: "Alice" }));
```

### Common Interview Questions
- Why must React state be treated as immutable?
- What is an infinite render loop in useEffect and how do you resolve it?
- Why should you avoid nesting component declarations inside another component's body?

### Reference Links
- [React Docs: Updating Objects in State](https://react.dev/learn/updating-objects-in-state)

## Scenario-based Interview Questions for React

### Scenario 1
A search component has an input field. As the user types, the app queries a backend API and updates the list. If they type fast, the list flickers and eventually displays search results for an older keystroke. How do you resolve this?

*Expected Approach:*
1. Identify that this is a race condition: an earlier network query resolved *after* a later query finished.
2. Fix this by using a cleanup flag inside useEffect (cancelling state updates on outdated fetches).
3. Alternatively, implement input debouncing to throttle the frequency of API requests, or use AbortController to cancel previous fetch operations.

### Scenario 2
Your team noticed that typing in a comments input box gets laggy when the comments list grows beyond 50 entries. The comment input box and the comment list share a parent component. What causes this and how do you fix it?

*Expected Approach:*
1. Recognize that on every keystroke, the parent component re-renders, causing all 50 list item children to re-render.
2. Fix this by separating state: move the input state into its own scoped child component, or wrap individual list items in React.memo so they only re-render if their individual props change.
3. Use profiling tools to confirm that re-renders are successfully bypassed.
