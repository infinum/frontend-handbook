*React Concurrency* is a system that introduces a way for React to perform interruptible and cooperative rendering. This system allows React to:

* Pause rendering tasks if something of higher priority arises (e.g., user input).
* Resume tasks without losing progress.
* Throw away outdated renders when new data or interactions supersede them.

Here are some key points about concurrency:

* It’s not multi-threading: React still runs on a single thread in the browser, but concurrency allows React to split and schedule work more intelligently.
* It’s about priority: Some updates are more urgent than others. User input to a text field, for instance, might be higher priority than re-rendering a large data grid.
* It’s built into React 18: You no longer need a special “*Concurrent Mode*” Instead, concurrency is integrated in the standard React rendering pipeline.

In practice, concurrency means you can keep your UI responsive even as the system handles complex state updates or server data fetching. In Next.js, concurrency extends beyond the client to also streamline server rendering and data streaming.

## Next.js concurrency

Next.js encourages usage of *Suspense* and streaming by default, aligning well with concurrency principles, it has multiple benefits:

1. **Performance**: Concurrency prevents large synchronous renders from blocking the main thread. Even if the server is fetching data from multiple APIs, concurrency helps ensure faster hydration and responsive interactions.
2. **Scalability**: Large Next.js applications can break pages into smaller concurrent chunks. For example, a complex dashboard might fetch dozens of data sources in parallel using server components.
3. **Better User Experience**: By streaming content and managing priorities, your app can show partial results or skeleton UIs almost immediately, letting users start interacting faster.
4. **Partial SSR**: The server can start sending the page as soon as some parts are ready.
5. **Faster Interactions**: React can handle user input at higher priority, reducing input lag.
6. **Server-Client Synergy**: The concurrency model is shared across server and client boundaries, thanks to *React Server Components* and Next.js’s build pipeline.

## Key concurrency features

*Concurrency* isn’t a single feature— it’s an umbrella term for *React 18* capabilities that allow more fine-grained rendering control. Let’s break down some highlights, then we’ll add an extra detail about interruption and scheduling.

### Concurrent rendering

React can break rendering into multiple chunks and spread them out. Prior to React 18, an expensive render could block the browser from responding to user input. Now, React can:

* Pause mid-render.
* Perform a quick higher priority update (like a user typing).
* Resume the previously paused render.

In Next.js, concurrent rendering is used during both client-side transitions and server rendering. Although you don’t “opt in” to concurrency specifically, you do need to be mindful of writing code that plays nicely with concurrent rendering. For instance, avoid side-effects that must complete in a single synchronous pass if not absolutely necessary.

### Suspense

*Suspense* has become more powerful with concurrency, especially when combined with Next.js. It allows you to:

* Wrap components that need time to load (e.g., data fetching or lazy-loaded components).
* Show a fallback UI while the component is “pending”.
* Gracefully reveal the loaded component once it’s ready, without blocking other parts of the UI.

For server components, *Suspense* helps with streaming partial HTML, so the user doesn’t stare at a blank screen. For client components, *Suspense* can also be used for code-splitting or data fetching (via libraries that integrate with *Suspense*, such as *React Query*).

### Transitions and `useTransition()`

With the `useTransition()` hook, you can mark certain state updates as lower priority. This is especially useful if the update triggers a large re-render. For example, you might have a complex search result or a big table that you re-render every time the user types in a filter box:

```jsx
const [isPending, startTransition] = useTransition();

function handleSearchInput(e: React.ChangeEvent<HTMLInputElement>) {
  const query = e.target.value;
  startTransition(() => {
    setSearchQuery(query);
  });
}
```

* `isPending`: Tells you if the transition is still in progress. You could show a spinner or a subtle indicator.
* `startTransition()`: Bundles the updates that can be deferred. High-priority tasks (like continuing to accept user keystrokes) won’t be blocked by these updates.

An example of cache revalidation with `useTransition()` can be found [here]() TODO - Link to caching and revalidation recipe...

### Streaming

*Streaming* is a Next.js feature that complements concurrency by allowing the server to send partial responses as soon as they’re ready. Instead of waiting for the entire page’s data to be fetched, the server:

1. Starts generating HTML for the earliest content.
2. Streams that content to the client.
3. Continues rendering and sends subsequent chunks as they are ready, especially if you have multiple *Suspense* boundaries.

This approach drastically improves the [time-to-first-paint](https://developer.mozilla.org/en-US/docs/Web/API/PerformancePaintTiming#:~:text=First%20Paint%20\(FP\)%3A%20Time%20when%20anything%20is%20rendered.%20Note%20that%20the%20marking%20of%20the%20first%20paint%20is%20optional%2C%20not%20all%20user%20agents%20report%20it.) on complex pages. The user can start reading or interacting with loaded sections while slower data or deeper nested components finish fetching on the server.

### Interruption and Scheduling

Beyond *streaming* and *Suspense*, the concurrency model also involves *scheduling*. Under the hood, React uses a cooperative scheduler:

1. It checks if there’s any urgent work. If so, it interrupts ongoing rendering.
2. Once urgent work is handled (e.g., user typed a character), React goes back to the paused render.

For Next.js developers, you mostly see the benefits automatically. However, you can fine-tune how React schedules certain updates with `useTransition()` and other concurrency hooks (`useDeferredValue()`), ensuring your app remains snappy under heavy loads.

## Server Components and Concurrency

*React Server Components* are a paradigm shift that let you run components entirely on the server. The server fetches data, compiles the resulting UI into a lightweight representation, and sends it to the client.

Concurrency helps in two key ways:

1. **Parallel Data Fetching**: The server can fetch multiple data sources concurrently, streaming partial results as soon as they’re ready.
2. **Reduced Client Overhead**: Because *RSC* logic is not bundled to the client, concurrency ensures minimal blocking on the client side.

This synergy means large server-side computations don’t freeze the user’s browser, and concurrency can schedule these computations in parallel, further reducing overall load time.

### Concurrency in the App Router

In the App Router:

* Each route can have its own `page.tsx`, `layout.tsx`, `loading.tsx`, and `error.tsx`.
* Concurrency ensures that if multiple route segments are loading at once, React can handle them in parallel or interrupt them if higher-priority tasks come in.
* Suspense boundaries can be placed at any layer, allowing partial streaming and partial hydration.

For example, you could have a `layout.tsx` that fetches top-level data about a user’s profile, while a nested route fetches more detailed info. Both can render concurrently, with partial results streaming to the client as they’re ready.

## Error Boundaries

*Error Boundaries* are React components (or special Next.js files like `error.tsx`) that catch errors in the render tree below them. In concurrent mode:

* If an error occurs during a partially completed render, React can discard that partial output and switch to the error boundary.
* This prevents a single data-fetching error from crashing your entire page or layout.
* Concurrency ensures the rest of the UI that’s error-free remains interactive and unaffected.

### Combining Error Boundary with Suspense

When you combine *Error Boundaries* with *Suspense*:

* If data is “pending,” you see the `<Suspense>` fallback (e.g., a spinner or skeleton).
* If an error occurs, you see the error boundary fallback (e.g., a friendly error message or “try again” button).
* This layered approach ensures the user always sees something, rather than a blank or broken screen. Coupled with concurrency, your app can remain stable under partial failures and can quickly recover without forcing a full page reload.

## Summary

You may be wondering what does all of this give me in practice... In day-to-day usage, you might not always be aware that concurrency is happening behind the scenes, but *React Concurrency* most often shows up in three main areas:

1. Data Fetching and Streaming
   * When you fetch data for a page or component (especially a *React Server Component* in the `app/` directory), Next.js can begin sending HTML to the client as soon as part of the data is ready. This means users can start seeing and interacting with certain parts of the page almost immediately, rather than waiting for the entire payload.
   * You’ll often place `<Suspense>` boundaries around components that need to fetch data. Next.js will show a `loading.tsx` (or your own custom fallback) while the data is being fetched, and then “stream” in the final component once it’s done.
2. Smoother UI Updates with Transitions
   * On the client side, if you have expensive or complex state updates (for example, filtering a large list or re-rendering a big table), concurrency ensures that user interactions (like typing in a text box) don’t get blocked by rendering work.
   * You’ll use `useTransition()` in your client components to mark certain updates as “transitions,” telling React that it’s okay to do the heavy rendering in the background while keeping the UI responsive.
3. *Partial Rendering* and *Error Boundaries*
   * Because React can pause and resume rendering, you can design your app so that if one part of the page is slow or errors out, it won’t break the rest of the page.
   * You’ll place *Error Boundaries* (or define `error.tsx` in the App Router) and *Suspense Boundaries* around sections of your layout or page that may fail or take longer to render. This way, Next.js and React can gracefully handle slow or failing sections without freezing or crashing the whole app.

## Recipes & Code Snippets

## Best Practices & Common Pitfalls

8. Over-Suspending Your App
   * Placing a `<Suspense>` boundary around every tiny component can lead to multiple fallbacks popping in and out. This can degrade the user experience by showing too many loading spinners or skeletons. Instead, group logically-related components under one boundary so they either load together or stay hidden together.
9. Underestimating Error Boundaries
   * In concurrent rendering, errors can appear more frequently in partial or “in-progress” states.
   * If you lack error boundaries, your entire app might go down with a single data-fetching mistake. Always place error boundaries around components that are more prone to fetch or rendering errors.
10. Transition Overuse
    * Not all updates need to be transitions. For example, real-time text input in a small form shouldn’t be deferred. Users expect immediate feedback.
    * Overusing transitions can make your app feel sluggish because you’re telling React it can “wait” to process certain updates.
11. Large CPU-Bound Operations
    * Concurrency doesn’t introduce actual multi-threading. It splits tasks into chunks, but if you have massive computations (e.g., cryptographic hashing, large image processing), they can still block the main thread. Consider offloading such tasks to Web Workers or the server.
12. Testing Complexity and Timings
    * Concurrency can introduce subtle timing and state issues that didn’t exist in synchronous mode.
    * Use *React Testing Library* and fake timers or libraries like *Mock Service Worker* to carefully test asynchronous states.
    * Be aware that transitions and streaming might require integration tests or end-to-end tests to ensure the final user experience is correct.
13. Security and Data Consistency
    * Since concurrency can interleave fetches, ensure you’re handling secure data carefully.
    * If you rely on user sessions or tokens, confirm that partial renders don’t leak data across boundaries or to unauthorized users.
    * On the server side, concurrency might call multiple APIs in parallel. Make sure your data dependencies don’t cause race conditions or partial data merges.

### Race Conditions

Server Components in Next.js fetch data on the server and return serialized component trees to the client. Under concurrency, multiple fetches and renders can occur in parallel. If multiple parallel fetches modify or depend on the same shared resource (like an in-memory store or a global variable), you might end up with inconsistent data or unexpected overwrites.

Concurrency means React can trigger different parts of the component tree simultaneously (especially with *Suspense boundaries*), so iff your server logic isn’t idempotent or thread-safe, parallel requests might conflict.

⚠️ Example:

```jsx
// app/(dashboard)/_components/SomeServerComponent.tsx
const inMemoryCache: {
  lastRender: number | null;
  [key: string]: any;
} = {
  lastRender: null,
}; // Shared mutable object

export default async function SomeServerComponent() {
  // Suppose each render modifies a global inMemoryCache
  // Two concurrent requests could clash or overwrite data
  inMemoryCache['lastRender'] = Date.now();

  const data = await fetch('https://api.example.com/data').then((res) => res.json());
  inMemoryCache[data.id] = data; // Possibly overwritten by another request in parallel

  // Render the updated data
  return (
    <>
      <p>Data for {data.id}</p>
      <p>Last Render Time: {inMemoryCache['lastRender']}</p>
    </>
  );
}
```

### Zombie UI states

A “*Zombie UI*” state happens when an old, outdated effect or render finishes after a newer update has already been applied, causing older data to overwrite the fresh data. It’s like a zombie rising back up and undoing your latest changes.

React discards outdated renders when it knows they’re obsolete, but certain side-effects or external subscriptions might not be properly canceled if the component code doesn’t account for concurrency. A typical scenario is where an older effect finishes after the newer effect is already rendered.

⚠️ Example:

```jsx
// app/products/page.tsx
import ProductsFilter from "./_components/ProductsFilter";

export default function ProductsPage() {
  return (
    <main style={{ padding: "1rem" }}>
      <h1>Zombie UI Concurrency Example</h1>
      {/* Client-side filter input + product list */}
      <ProductsFilter />
    </main>
  );
}
```

```ts
// app/products/actions.ts

// Simulate a large product database
const ALL_PRODUCTS = [
  { id: 1, title: "Laptop" },
  { id: 2, title: "Camera" },
  { id: 3, title: "Headphones" },
  { id: 4, title: "Smartphone" },
  { id: 5, title: "Tablet" },
  { id: 6, title: "Smartwatch" },
  { id: 7, title: "TV" },
  { id: 8, title: "Gaming Console" },
  { id: 9, title: "External Hard Drive" },
  { id: 10, title: "Monitor" },
  { id: 11, title: "Printer" },
  { id: 12, title: "Keyboard" },
  { id: 13, title: "Mouse" },
  { id: 14, title: "Desk" },
  { id: 15, title: "Office Chair" },
  { id: 16, title: "Webcam" },
  { id: 17, title: "Microphone" },
  { id: 18, title: "USB Hub" },
  { id: 19, title: "Router" },
  { id: 20, title: "Smart Home Device" },
  { id: 21, title: "Drone" },
  { id: 22, title: "Projector" },
  { id: 23, title: "Digital Camera" },
  { id: 24, title: "Action Camera" },
  { id: 25, title: "Fitness Tracker" },
  { id: 26, title: "External SSD" },
  { id: 27, title: "Camera Lens" },
  { id: 28, title: "Tripod" },
  { id: 29, title: "Camera Bag" },
  { id: 30, title: "Gimbal" },
  // ... potentially hundreds more
];

// This server action is intentionally slow to illustrate concurrency
export async function fetchFilteredProducts(filter: string) {
  // Simulate slow fetching (~1-2 seconds)
  const timeout = Math.random() * 1000 + 1000;

  console.log(
    `Fetching products for filter ${filter} with timeout ${timeout}ms`
  );

  await new Promise((resolve) => setTimeout(resolve, timeout));

  // Basic filter
  const lower = filter.toLowerCase();
  const filtered = ALL_PRODUCTS.filter((p) =>
    p.title.toLowerCase().includes(lower)
  );

  return filtered;
}
```

```jsx
// app/products/_components/ProductsFilter.tsx

"use client";

import React, { useState, useTransition } from "react";
import { fetchFilteredProducts } from "../actions"; // Import server action

export default function ProductsFilter() {
  const [filter, setFilter] = useState("");
  const [products, setProducts] = useState<{ id: number; title: string }[]>([]);
  const [isPending, startTransition] = useTransition();

  // We'll store a "requestId" to illustrate how zombie updates happen
  let requestIdCounter = 0;

  async function handleFilterChange(newFilter: string) {
    setFilter(newFilter);

    // We generate a unique ID for this request
    const currentRequestId = ++requestIdCounter;

    // Mark the fetch as a transition (Concurrency!). React can interrupt, reorder, etc.
    startTransition(async () => {
      const result = await fetchFilteredProducts(newFilter);

      // ZOMBIE PITFALL: If this older fetch finishes last, it overwrites newer results
      // We do not guard against that, so let's illustrate what can happen:
      console.log(
        `[Request ${currentRequestId}] Filter "${newFilter}" => ${result.length} products`
      );

      // Overwrites state, even if a newer request has completed
      setProducts(result);
    });
  }

  return (
    <section>
      <h2>Client-Side Filter (Concurrency)</h2>
      <input
        type="text"
        placeholder="Search products..."
        value={filter}
        onChange={(e) => handleFilterChange(e.target.value)}
        style={{ marginBottom: "0.5rem" }}
      />
      {isPending && <p>Loading filtered products...</p>}

      <ul>
        {products.map((p) => (
          <li key={p.id}>{p.title}</li>
        ))}
      </ul>
    </section>
  );
}
```

**How the Zombie occurs**

1. User types "cam" (request #1).
2. Before it finishes, they continue typing "camera" (request #2).
3. Concurrency means React can run both fetch calls in parallel or partial sequence.
4. If request #2 returns first, we set products to `["Camera", "Digital Camera", "Action Camera", "Camera Lens", "Camera Bag"]`.
5. Then request #1 finishes last (zombie!). It calls `setProducts(...)` with results for "cam" — overwriting the correct array with additional `Webcam` product.

Zombie updates did exist before concurrency, but concurrency increases their frequency and can reorder them in ways old React rarely did, making it a bigger pitfall if you don’t handle stale requests carefully. In older React (pre-React 18), if the user typed quickly, the browser often wouldn’t allow an overlapping render or partial updates mid-SSR. The UI might block or become unresponsive, making it less likely to see such a neat out-of-order scenario.

**Fixing the Zombie Issue**

To prevent the outdated fetch from overwriting fresh data, we can:

1. Track a local `requestId`.
2. Abort older requests if concurrency or user input changes.
3. Check if the request is “still valid” before calling `setProducts()`.

✅ Example:

```jsx
// app/products/_components/ProductsFilter.tsx

"use client";

import React, { useState, useTransition, useRef } from "react";
import { fetchFilteredProducts } from "../actions";

/**
 * Demonstrates preventing zombie updates by using an imperative requestIdRef.
 * Each new request increments the ref. The async callback only updates state
 * if the requestId matches the ref's current value.
 */
export default function ProductsFilter() {
  const [filter, setFilter] = useState("");
  const [products, setProducts] = useState<{ id: number; title: string }[]>([]);

  // A ref that tracks the "latest" request ID. We'll increment this each time
  // the user triggers a new fetch.
  const requestIdRef = useRef(0);

  // For concurrency: we mark fetch updates as transitions.
  const [isPending, startTransition] = useTransition();

  // Handler for typing in the filter input
  function handleFilterChange(newFilter: string) {
    setFilter(newFilter);

    // 1) Increment the requestIdRef to represent a brand-new request
    requestIdRef.current += 1;
    const localRequestId = requestIdRef.current;

    // 2) Start a concurrent fetch
    startTransition(async () => {
      const result = await fetchFilteredProducts(newFilter);

      // 3) After the fetch, check if requestIdRef is still the same
      if (localRequestId === requestIdRef.current) {
        console.log(
          `%cRequest #${localRequestId} is the latest, updating products`,
          "color: green;"
        );
        setProducts(result);
      } else {
        console.log(
          `%c[Zombie Prevented] Request #${localRequestId} is stale`,
          "color: orange;"
        );
      }
    });
  }

  return (
    <section>
      <h2>Client-Side Filter (Concurrency, Zombie-Safe via Ref)</h2>
      <input
        type="text"
        placeholder="Search products..."
        value={filter}
        onChange={(e) => handleFilterChange(e.target.value)}
        style={{ marginBottom: "0.5rem" }}
      />

      {isPending && <p>Loading filtered products...</p>}

      <ul>
        {products.map((p) => (
          <li key={p.id}>{p.title}</li>
        ))}
      </ul>
    </section>
  );
}
```

**Why this is just a demonstration**

You may be wondering why `AbortController` isn’t used in the zombie UI fix example... The reason lies in the nature of Next.js Server Actions, which are fundamentally different from standard network requests like `fetch`.

Server Actions operate as a special [“RPC-like” mechanism](https://scastiel.dev/simplest-example-server-actions-nextjs#:~:text=If%20you%20are%20familiar%20with%20distributed%20computing%20patterns%2C%20you%20can%20notice%20that%20server%20actions%20offer%20a%20remote%20procedure%20call%20\(RPC\)%20pattern%20for%20Next.js%20applications.) for calling server-side logic directly from your React components. Unlike traditional API calls, Server Actions don’t produce a plain HTTP request you can intercept or cancel using an `AbortSignal`. As such, `AbortController` isn’t natively supported when working with Server Actions - they’re designed to work seamlessly with React’s concurrency features rather than supporting low-level request cancellation.

If true request cancellation is required for your use case, such as when working with large or expensive data fetching operations, you’ll need to use an API Route Handler (e.g., `app/api/products/route.ts`) or a standard endpoint. These endpoints support `AbortSignal`, allowing you to use `AbortController` to manage request cancellation in a traditional manner.

### Overlapping or Conflicting Transitions

Multiple `useTransition()` calls update the same state concurrently, causing conflicts or unexpected final states.

⚠️ Example:

```jsx
"use client";

import { useState, useTransition } from "react";

export default function ConflictingTransitions() {
  const [count, setCount] = useState(0);
  const [isPending1, startTransition1] = useTransition();
  const [isPending2, startTransition2] = useTransition();

  const incrementBy2 = () => {
    startTransition1(() => setCount((c) => c + 1));
    startTransition2(() => setCount((c) => c + 1));
  };

  return (
    <div>
      <h1>Count: {count}</h1>
      {(isPending1 || isPending2) && <p>Updating...</p>}
      <button onClick={incrementBy2}>Increment by 2</button>
    </div>
  );
}
```

Both transitions run at the same time. React merges the state updates, but if transitions rely on the same state in different contexts, it can lead to unpredictability (though this exact example might still yield the correct `count`, more complex logic can cause issues).

✅ In order to fix this, bundle related updates into one transition:

```jsx
 const incrementBy2 = () => {
    // Use a single transition, bundling all state changes that belong together
    startTransition(() => {
      setCount((c) => c + 1);
      setCount((c) => c + 1);
    });
  };
```

### Debugging Complexity

Concurrency scheduling can make debugging more complicated. Log statements, breakpoints, and timeline events may appear out of order. A piece of code might run partially, pause, resume, and then be discarded, making it hard to follow your app's flow.

⚠️ Example:

```jsx
// app/debugging-concurrency/page.tsx

"use client";

import React, { useState, useEffect, useTransition } from "react";

// Simulate a delay for the count update
function simulateDelay(ms: number) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

export default function ConcurrencyRearrangementDemo() {
  const [count, setCount] = useState(0); // Low-priority state
  const [userInput, setUserInput] = useState(""); // High-priority state
  const [isPending, startTransition] = useTransition();

  console.log(`[Render Start] Count: ${count}, UserInput: "${userInput}"`);

  // Effect to log count changes
  useEffect(() => {
    console.log(`[Effect] Count committed: ${count}`);
  }, [count]);

  // Effect to log user input changes
  useEffect(() => {
    console.log(`[Effect] UserInput committed: "${userInput}"`);
  }, [userInput]);

  // Handle increment with a simulated delay
  function handleIncrement() {
    console.log(`[Event Handler] Increment Clicked. Count: ${count}`);

    startTransition(async () => {
      console.log(`[Transition] Starting low-priority count update.`);
      await simulateDelay(2000); // Simulate a 2-second delay
      setCount((prev) => {
        console.log(
          `[State Update] Updating count from ${prev} to ${prev + 1}`
        );
        return prev + 1;
      });
    });

    console.log(`[Event Handler] Transition initiated.`);
  }

  return (
    <div>
      <p>
        <strong>Count:</strong> {count}
      </p>
      <button onClick={handleIncrement} style={{ marginBottom: "1rem" }}>
        Increment (Low Priority)
      </button>
      <br />
      <input
        type="text"
        placeholder="Type here (High Priority)"
        value={userInput}
        onChange={(e) => setUserInput(e.target.value)}
        style={{ padding: "0.5rem", width: "300px" }}
      />
      {isPending && <p style={{ color: "gray" }}>Updating count...</p>}
    </div>
  );
}
```

When you click the button and type `React` quickly, logs for high-priority input updates may interleave with or appear before logs from the low-priority count transition.

Console output:

```
[Event Handler]   Increment Clicked. Count: 0
[Transition]      Starting low-priority count update.
[Event Handler]   Transition initiated.
[Render Start]    Count: 0, UserInput: ""
[Render Start]    Count: 0, UserInput: "R"
[Effect]          UserInput committed: "R"
[Render Start]    Count: 0, UserInput: "Re"
[Effect]          UserInput committed: "Re"
[Render Start]    Count: 0, UserInput: "Rea"
[Effect]          UserInput committed: "Rea"
[Render Start]    Count: 0, UserInput: "Reac"
[Effect]          UserInput committed: "Reac"
[Render Start]    Count: 0, UserInput: "React"
[Effect]          UserInput committed: "React"
[State Update]    Updating count from 0 to 1
[Render Start]    Count: 1, UserInput: "React"
[Effect]          Count committed: 1
[Render Start]    Count: 1, UserInput: "React"
```

It's hard to debug, because:

* The logs for userInput (e.g., `"R"`, `"Re"`) appear before the count update logs (`"Updating count from 0 to 1"`), even though the button click happened first.
* It's unclear whether the input state is being updated before, during, or after the transition.
* React may retry or discard renders during transitions. This can result in `useEffect` running for states that aren't final, so we may see `[Effect] Count committed: 1` multiple times in the console, so if you rely on side effects (e.g., API calls or analytics logging), duplicate effects can lead to confusing behavior or incorrect data.
* Logs for intermediate states (e.g., `[Render Start]`) can appear even if React discards them, making it unclear which state React is actually committing.

**How to mitigate debugging challenges**

✅ Add timestamps or unique markers to logs to track when each task starts and finishes:

```jsx
console.log(`[${Date.now()}] Count Update Started`);
```

✅ Use a `useRef` to track the last committed state:

```jsx
const lastCommittedCount = useRef(count);

useEffect(() => {
  console.log(`Committed Count: ${lastCommittedCount.current}`);
  lastCommittedCount.current = count;
}, [count]);
```

✅ Debounce high-priority state updates to reduce frequent interruptions:

```jsx
function handleInputChange(e) {
  debounce(() => setUserInput(e.target.value), 300);
}
```

✅ Use [React DevTools Profiler](https://react.dev/learn/react-developer-tools) - it shows how React pauses, resumes, or retries renders, helping you identify why certain logs appear multiple times or out of order.

### Side-Effects during Interruption

Similarly to previous example, in concurrent mode, React might start rendering a component, then pause, then discard that render entirely if a higher priority update appears - if side-effects run at render-time or are triggered prematurely, they might execute even for a half-complete render.

⚠️ Example:

```jsx
// app/interruption/page.tsx:

"use client";

import React, { useState, useEffect, useTransition } from "react";

// Simulate expensive computation
function generateItems(count: number): string[] {
  const items = [];
  for (let i = 0; i < count; i++) {
    items.push(`Item ${i + 1}`);
  }
  return items;
}

const INITIAL_COUNT = 5000;
const INCREMENT_BY = 1000;

export default function ConcurrencyInterruptionExamplePage() {
  const [count, setCount] = useState(INITIAL_COUNT);
  const [items, setItems] = useState(() => generateItems(count));
  const [isPending, startTransition] = useTransition();

  // Simulate a "side effect" that logs list size
  useEffect(() => {
    console.log("Side Effect: List size changed to", items.length);
    // In a real app, this could be an analytics call or database update
  }, [items]);

  function handleAddItems() {
    startTransition(() => {
      // Simulate an expensive state update
      setItems(generateItems(count + INCREMENT_BY));
      setCount((prev) => prev + INCREMENT_BY);
    });
  }

  return (
    <div>
      <h1>Concurrency Interruption example</h1>
      <p>
        <strong>Item Count:</strong> {count}
      </p>
      <button onClick={handleAddItems}>Add {INCREMENT_BY} items</button>
      {isPending && <p>Updating items...</p>}
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

Problems in this example:

1. Separate State Updates:
   * count and items are managed as independent state variables, but they are logically tied together.
   * This separation can cause inconsistencies when React handles updates concurrently.

2. Inconsistent State During Concurrency:
   * React's concurrency allows rendering to pause, resume, or even discard updates.
   * Rapid clicks on "Add 5 Items" may result in count and items.length becoming out of sync, especially under heavy user interaction.

3. Uncontrolled Side Effects:
   * The useEffect hook logs the list size (items.length) on every render where items changes.
   * This can lead to duplicate or premature logs for intermediate states that are discarded during partial renders.

> Note: On fast computers or with a lower `INCREMENT_BY` value, these issues might not be noticeable. To reliably reproduce them, throttle your CPU using tools like Chrome DevTools' "Performance" tab.

✅ Here's an example solution:

```jsx
// app/interruption/page.tsx:

"use client";

import React, { useState, useEffect, useTransition, useRef } from "react";

// Simulate expensive computation to generate a list of items
function generateItems(count: number): string[] {
  const items = [];
  for (let i = 0; i < count; i++) {
    items.push(`Item ${i + 1}`);
  }
  return items;
}

const INITIAL_COUNT = 5000;
const INCREMENT_BY = 1000;

export default function ConcurrencyInterruptionFixedExamplePage() {
  // Unified state: Combine `count` and `items` into a single object
  const [state, setState] = useState(() => ({
    count: INITIAL_COUNT,
    items: generateItems(INITIAL_COUNT),
  }));
  const [isPending, startTransition] = useTransition();

  // Ref to track the last committed count value
  const lastCommittedCount = useRef(state.count);

  // Side effect to log changes in list size (ensures consistency)
  useEffect(() => {
    if (lastCommittedCount.current !== state.count) {
      console.log("Side Effect: List size changed to", state.items.length);
      lastCommittedCount.current = state.count; // Update the committed state
    }
  }, [state]);

  // Handler for adding more items
  function handleAddItems() {
    console.log(`Logging directly: Adding ${INCREMENT_BY} items to the list`);

    startTransition(() => {
      setState((prevState) => {
        const newCount = prevState.count + INCREMENT_BY;
        const newItems = generateItems(newCount);

        return {
          count: newCount,
          items: newItems,
        };
      });
    });
  }

  return (
    <div style={{ padding: "1rem", fontFamily: "Arial, sans-serif" }}>
      <h1>Concurrency Interruption fixed example</h1>
      <p>
        <strong>Item Count:</strong> {state.count}
      </p>
      <button onClick={handleAddItems} style={{ padding: "0.5rem 1rem" }}>
        Add {INCREMENT_BY} Items
      </button>
      {isPending && <p style={{ color: "gray" }}>Updating items...</p>}
      <ul>
        {state.items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

After the changes we have:

1. Unified State
   * `count` and `items` are now managed in a single state object, ensuring that updates to both happen atomically.
   * This eliminates the possibility of `count` and `items.length` diverging during rapid user interactions or concurrent updates.
2. Atomic Updates
   * The `setState` callback (`setState((prevState) => ...)`) ensures that updates are based on the latest committed state, even under React's concurrent rendering.
   * Both count and items are updated together in a single render cycle.
3. `Ref` tracking for Consistent Side Effects
   * The `useRef` (`lastCommittedCount`) tracks the last finalized `count` value.
   * This prevents duplicate or premature execution of side effects for transient states during partial renders.
4. Explicit Side Effects in Event Handlers
   * Logging directly in the `handleAddItems` function ensures critical actions are tied to user interactions, not React's rendering lifecycle.

### Unintended large server workload

With concurrency, Next.js can fetch multiple data sources in parallel for a single server-rendered page. If a page is extremely complex (e.g., a dashboard with many widgets), concurrency might spawn a large number of parallel fetches. Under high traffic, this could spike load on your backend or databases.
