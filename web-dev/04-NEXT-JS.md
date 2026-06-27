# 04 Next.js

This document covers Next.js, focusing on the App Router, Server Components, data fetching strategies, and middleware configuration.

## Next.js Fundamentals

### Definition
Next.js is a React framework that enables server-side rendering (SSR), static site generation (SSG), incremental static regeneration (ISR), and features zero-configuration code splitting, file-based routing, and asset optimization out of the box.

### Real-world Analogy
Think of a complete catering package. Instead of buying individual ingredients, plates, and stoves and setting them up yourself (like plain React + Webpack), you hire a catering company that brings pre-cooked meals, plates, and servers, ready to set up on site.

### Code Example
```javascript
// Next.js config template showing zero config optimization basics
module.exports = {
  reactStrictMode: true,
  images: {
    domains: ["assets.example.com"],
  },
};
```

### Common Interview Questions
- What are the major differences between Create React App/Vite and Next.js?
- How does routing in Next.js differ from client-side React Router?
- What are the default optimization features Next.js provides automatically?

### Reference Links
- [Next.js Docs: Getting Started](https://nextjs.org/docs)
- [Next.js Docs: Basic Features](https://nextjs.org/docs/app/building-your-application/routing)

## App Router

### Definition
The Next.js App Router (introduced in version 13) is a file-system based router located in the 'app' directory. It uses React Server Components by default and supports layout nesting, templates, loading and error boundaries, and special route groups.

### Real-world Analogy
Think of a structured department store. The paths (folders) like /clothing, /shoes lead customers to sections. Each section has a persistent floor manager layout (layout.tsx), a main display counter (page.tsx), and temporary signs shown while loading items (loading.tsx).

### Code Example
```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <section>
      <nav>Dashboard Nav</nav>
      {children}
    </section>
  );
}
```

### Common Interview Questions
- Describe the difference between layout.tsx and template.tsx.
- What are parallel routes (@folder) and intercepting routes ((.)folder) and when do you use them?
- How do route groups (folders named with parentheses) help organize projects without affecting URLs?

### Reference Links
- [Next.js Docs: App Router](https://nextjs.org/docs/app)
- [Next.js Docs: File Conventions](https://nextjs.org/docs/app/api-reference/file-conventions)

## Server Components vs Client Components

### Definition
Next.js components are Server Components by default. They execute on the server and do not send JavaScript to the client. Adding the "use client" directive at the top of a file converts it into a Client Component, which allows client-side interactivity, state, and event listeners.

### Real-world Analogy
Think of a restaurant. Server Components are dishes cooked in the kitchen and served complete to the customer table. Client Components are tabletop hotpots where the customer receives ingredients and cooks them at the table, requiring active gas burners (JavaScript) at the table.

### Code Example
```tsx
// app/users/page.tsx (Server Component)
import { UserButton } from "./UserButton";

export default async function UsersPage() {
  const users = [{ id: 1, name: "Alice" }]; // Pretend database fetch
  return (
    <div>
      <h1>Users List</h1>
      {users.map(u => (
        <div key={u.id}>
          {u.name} <UserButton userId={u.id} />
        </div>
      ))}
    </div>
  );
}

// app/users/UserButton.tsx (Client Component)
"use client";
export function UserButton({ userId }: { userId: number }) {
  return <button onClick={() => alert(`Clicked ${userId}`)}>Action</button>;
}
```

### Common Interview Questions
- When should you use a Client Component instead of a Server Component?
- Why can you not import server-only code/modules inside a Client Component?
- How do you pass data from a Server Component to a Client Component?

### Reference Links
- [Next.js Docs: Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components)
- [Next.js Docs: Client Components](https://nextjs.org/docs/app/building-your-application/rendering/client-components)

## Server Actions

### Definition
Server Actions are asynchronous functions that run on the server but are triggered from the client, allowing you to handle form submissions, mutations, and database updates directly without creating custom API routes.

### Real-world Analogy
Imagine a magic order button on a restaurant table. Instead of writing an order slip, handing it to a waiter, waiting for them to walk to the kitchen and register it (traditional API endpoints), you press the button, and the kitchen updates your order status directly.

### Code Example
```tsx
// app/actions.ts
"use server";

import { revalidatePath } from "next/cache";

export async function createPost(formData: FormData) {
  const title = formData.get("title");
  // Save to database here: await db.insert(title)
  console.log("Created post: ", title);
  revalidatePath("/posts");
}

// app/posts/page.tsx
import { createPost } from "../actions";

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input type="text" name="title" />
      <button type="submit">Create Post</button>
    </form>
  );
}
```

### Common Interview Questions
- How do you handle validation errors and return state inside a Server Action (e.g. useActionState)?
- Explain how Server Actions enforce security against Cross-Site Request Forgery (CSRF).
- How do you trigger path and tag revalidation within a Server Action?

### Reference Links
- [Next.js Docs: Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)

## Data Fetching in Next.js

### Definition
Data fetching in App Router leverages native fetch extendability to control caching and revalidation. It supports static queries, dynamic queries (no-store), and streaming UI content via Suspense boundaries and loading.tsx.

### Real-world Analogy
Think of ordering fresh fish. SSG is buying canned tuna that is stocked at the factory. SSR is ordering live lobster caught from the tank when you arrive. ISR is the store getting fresh salmon delivered every morning at 6 AM; if you come at noon, you get the morning delivery.

### Code Example
```tsx
// Fetch with revalidation options
async function getPosts() {
  // ISR: Revalidate data every 60 seconds
  const res = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60 },
  });
  return res.json();
}

export default async function FeedPage() {
  const posts = await getPosts();
  return <div>Loaded {posts.length} posts</div>;
}
```

### Common Interview Questions
- What is the difference between fetch options { cache: 'no-store' } and { next: { revalidate: 3600 } }?
- How does generateStaticParams work, and what is its purpose?
- Explain how streaming HTML using Suspense boundaries improves loading performance.

### Reference Links
- [Next.js Docs: Fetching Data](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)

## SSR, SSG, and ISR

### Definition
Server-Side Rendering (SSR) generates HTML on each request. Static Site Generation (SSG) compiles HTML at build time. Incremental Static Regeneration (ISR) updates static pages incrementally in the background after the cache expires without rebuilding the entire app.

### Real-world Analogy
SSR is a tailor sewing a suit from scratch only when a customer walks in. SSG is a shop selling off-the-rack suits made during off-season. ISR is a shop making 100 off-the-rack suits, but tailoring one adjustments copy in the background if a buyer needs a tweak.

### Code Example
```tsx
// Dynamic routing configuration for SSR vs SSG control
export const dynamic = "force-dynamic"; // Forces SSR behavior

export default async function Page() {
  return <div>Rendered dynamically on each request</div>;
}
```

### Common Interview Questions
- Explain when you should use SSG over SSR.
- How does ISR serve stale content to a user while updating the cache in the background?
- What happens if a user accesses an ISR page that is not yet generated?

### Reference Links
- [Next.js Docs: Rendering](https://nextjs.org/docs/app/building-your-application/rendering)

## API Routes

### Definition
API Routes in App Router are defined using route.ts files inside the app directory. They export handler functions named after HTTP methods (GET, POST, PUT, DELETE) and return standard web NextResponse objects.

### Real-world Analogy
Think of a bank drive-thru window. Instead of walking into the lobby, you pull up, put your documents in a canister, and send it to the teller, who processes it and returns your receipt through the pipe.

### Code Example
```typescript
// app/api/users/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({ users: [{ id: 1, name: "Alice" }] });
}

export async function POST(request: Request) {
  const body = await request.json();
  return NextResponse.json({ success: true, received: body }, { status: 201 });
}
```

### Common Interview Questions
- How do you access path parameters inside a route.ts handler?
- What is the difference between Request/Response and NextRequest/NextResponse?
- How do you return custom headers and status codes in API route responses?

### Reference Links
- [Next.js Docs: Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers)

## Middleware

### Definition
Middleware in Next.js is a file (middleware.ts) located at the root of the project that runs before a request is completed. It allows you to redirect requests, rewrite paths, modify headers, and inspect or set cookies.

### Real-world Analogy
Think of a security guard standing in the office lobby. Before visitors can take the elevator to their floor, the guard checks their badge (auth cookie). If they do not have a badge, the guard directs them to the reception desk (redirects to login).

### Code Example
```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("token")?.value;

  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", request.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

### Common Interview Questions
- Where should the middleware.ts file be placed, and what are matcher options?
- Compare NextResponse.redirect and NextResponse.rewrite.
- What are the runtime constraints of Next.js middleware (e.g. running on the Edge)?

### Reference Links
- [Next.js Docs: Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)

## Image Optimization

### Definition
The next/image component extends the HTML <img> element with automatic format conversion (WebP, AVIF), image resizing, lazy loading, and prevention of layout shifts using blur placeholders.

### Real-world Analogy
Imagine a printing press with different templates. Instead of printing a huge wall poster and forcing customers to fold it up to fit in their pockets, the press prints custom pocket-sized versions of the photo for mobile users automatically.

### Code Example
```tsx
import Image from "next/image";
import profilePic from "../public/me.png";

export default function Profile() {
  return (
    <Image
      src={profilePic}
      alt="Profile picture"
      width={500}
      height={500}
      placeholder="blur"
      priority // Load immediately if above the fold
    />
  );
}
```

### Common Interview Questions
- Why does the next/image component require you to specify width and height or layout fill?
- How does the priority prop prevent Largest Contentful Paint (LCP) delays?
- How do you configure next/image to handle external image URLs safely?

### Reference Links
- [Next.js Docs: Image Component](https://nextjs.org/docs/app/api-reference/components/image)

## Metadata API

### Definition
The Metadata API allows you to define page head metadata (title, meta description, openGraph tags) either statically using config objects or dynamically using the generateMetadata function.

### Real-world Analogy
Imagine placing a book on a shelf. The book has a cover displaying the title, author, and description (metadata). When libraries index the book (search engines crawling the site) or someone texts a friend the book title, they look at the cover, not the pages.

### Code Example
```tsx
// Static metadata
export const metadata = {
  title: "Home Page",
  description: "Welcome to our application",
};

// Dynamic metadata
export async function generateMetadata({ params }: { params: { id: string } }) {
  const res = await fetch(`https://api.com/products/${params.id}`);
  const product = await res.json();
  return {
    title: product.name,
    description: product.description,
  };
}

export default function ProductPage() {
  return <h1>Product details</h1>;
}
```

### Common Interview Questions
- What is the difference between static metadata and generateMetadata?
- How do you generate dynamic openGraph and twitter card configurations for sharing?
- Where can you define global fallback metadata configurations?

### Reference Links
- [Next.js Docs: Metadata](https://nextjs.org/docs/app/building-your-application/optimizing/metadata)

## Next.js Deployment on Vercel

### Definition
Vercel is the creator of Next.js and provides optimized zero-config hosting. It deploys API routes and server components as Serverless or Edge functions and handles static caching automatically via its Global Edge Network.

### Real-world Analogy
Imagine renting an apartment from the architect who built the building. They know exactly where the pipes are, how to maximize storage space, and how to maintain utility efficiency without you needing to read the plumbing blueprints.

### Code Example
```bash
# Deploy a Next.js project to Vercel using the Vercel CLI
npm i -g vercel
vercel login
vercel
```

### Common Interview Questions
- Why is Vercel considered the reference platform for Next.js deployments?
- How does Vercel convert dynamic route handlers into serverless/edge functions?
- How do you manage preview deployments for GitHub pull requests?

### Reference Links
- [Vercel Docs](https://vercel.com/docs)
- [Next.js Docs: Deployment](https://nextjs.org/docs/app/building-your-application/deploying)

## Next.js vs Create React App vs Vite

### Definition
Vite and Create React App are client-side bundlers that serve a single HTML file and build dynamic trees entirely in the browser. Next.js is a full-stack framework that generates pages on the server and provides hybrid rendering.

### Real-world Analogy
Vite is selling a DIY desk box: the buyer gets a bundle of parts and must assemble it in their room. Next.js is a furniture company that delivers a fully built desk, placing it in the room ready for use immediately.

### Code Example
```json
// package.json scripts comparing Vite vs Next.js
// Vite uses: "dev": "vite", "build": "tsc && vite build"
// Next.js uses: "dev": "next dev", "build": "next build", "start": "next start"
```

### Common Interview Questions
- What are the search engine optimization (SEO) implications of client-side Vite versus Server-Side Next.js?
- When is a client-only single-page application (using Vite) preferred over Next.js?
- How does the build asset output structure differ between Next.js and Vite?

### Reference Links
- [Next.js Docs: Compare](https://nextjs.org/docs/app/building-your-application/rendering/server-components#server-rendering-with-react)

## Scenario-based Interview Questions for Next.js

### Scenario 1
You are building an e-commerce product page. Product descriptions change occasionally, but page load time is critical. Search engine optimization (SEO) is also a major priority. How do you implement this in Next.js?

*Expected Approach:*
1. Suggest Incremental Static Regeneration (ISR) using fetch with revalidation options (e.g., next: { revalidate: 3600 }) to pre-render the pages statically.
2. This ensures fast load times (SSG speed) and great SEO.
3. The page is automatically rebuilt in the background when requested after the revalidation period, ensuring description updates show up.

### Scenario 2
Users reporting login states flicker when navigating between secure pages in your Next.js application. You are currently checking auth tokens inside client-side components. How do you fix this?

*Expected Approach:*
1. Explain that client-side check requires the JS bundle to load and execute first, causing a flash of unauthenticated UI.
2. Move the authentication check to Next.js Middleware (middleware.ts) or a Server Component layout.
3. Check the authentication cookie on the server before rendering the page, redirecting unauthenticated users immediately.
