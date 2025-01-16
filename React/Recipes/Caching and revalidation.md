## Introduction

Caching is essential for building performant, reliable, and scalable applications. In the world of modern web development, being able to store and retrieve data efficiently can often mean the difference between a seamless user experience and a sluggish one. With Next.js’s new App Router and React Server Components, the way we approach caching and revalidation has evolved. The ability to fetch data on the server, cache it strategically, and revalidate content automatically (or *on-demand*) provides developers with powerful tools to keep apps both fast and up-to-date.

This chapter aims to:

* Provide basic knowledge about caching.
* Explain advanced caching and revalidation strategies.
* Cover the various methods of caching (browser, HTTP, in-memory, edge).
* Dive into revalidation (ISR, on-demand revalidation, fallback strategies, and more).
* Include recipes and code snippets to get you started quickly.
* Offer tips, best practices, and pitfalls to watch out for.

## Refresh on fundamentals

Before diving into caching, let’s quickly refresh on how Next.js’s App Router and React Server Components (RSC) work together. Understanding these fundamentals sets the stage for how we’ll discuss caching in Next.js and how we can best leverage these new features to build more performant applications.

### App Router

* The App Router was introduced in Next.js 13 to provide a more flexible and modular approach to routing.
* Routes are now organized under the `app/` directory instead of the traditional `pages/` directory.
* Each folder in the `app/` directory can contain a layout (`layout.tsx`), a page component (`page.tsx`), and a route handler (`route.ts`), among others - You can read more about structuring your project in the [Project Structure chapter](/frontend/react/project-structure).

### React Server Components

* *React Server Components* allow you to fetch data and render components on the server, sending the rendered result to the client.
* The big advantage of *RSC* is that it can reduce the client-side JavaScript bundle size and improve performance, as server-rendered content doesn’t need to bundle all logic for fetching data on the client.

### Data Layer & Caching implications

* Server Components fetch data on the server, which means you can apply server-side caching strategies (like in-memory caches, Redis, edge caching, etc.) more conveniently.
* Because the server is in control, you can also easily manage authentication, environment variables, secrets, and other sensitive data.

## Why do we even cache?

1. Performance
   * Reduces latency and speeds up data retrieval.
   * Minimizes repeated expensive operations (e.g., database queries, external API calls).
2. Scalability
   * Offloads frequent read operations from your databases or APIs.
   * Helps maintain consistent performance under high load.
3. Costs
   * Lower usage of third-party APIs or databases can reduce your operational costs.
   * Minimizing network overhead reduces cloud hosting fees in some scenarios.
4. User Experience
   * Decreases [Time to first byte (TTFB)](https://developer.mozilla.org/en-US/docs/Glossary/Time_to_first_byte).
   * Improves perceived performance and user satisfaction.

## Types of Caching

Caching isn’t a one-size-fits-all concept. The appropriate caching mechanism depends on:

* The type of data you’re fetching (static vs. dynamic).
* The frequency of data updates.
* Your infrastructure (CDN, serverless functions, etc.).
* Your operational constraints (cost, complexity, etc.).

We also have multiple different types of caching, each with it's own strategies:

* Web & Network Caching (Browser, HTTP, DNS).
* Application-Level Caching (In-Memory, Page).
* Database Caching (Query, Index, Row / Record).
* Hardware & System Caching (CPU, Disk, I/O).
* Distributed & Cloud Caching (Edge, Redis).
* Specialized Caching (Streaming, Compiler, Games).

Below are the major caching strategies you’ll likely encounter.

### Browser Caching

Browser caching is the simplest form of caching:

* 📌 Primarily controlled via `Cache-Control`, `ETag`, and [other HTTP headers](https://dev.to/andreasbergstrom/understanding-cache-control-and-etag-for-efficient-web-caching-2nf5) sent by the server.
* 📌 Helps reduce repeated downloads of static assets (images, CSS, JavaScript).
* ✅ Straightforward, no server involvement once the client has the resource.
* ⚠️ Can’t effectively manage dynamic content changes (short of forcibly invalidating the cache).

### HTTP Caching (CDNs & Proxies)

Content Delivery Networks (e.g. [AWS Cloudfront](https://aws.amazon.com/cloudfront/)) and reverse proxies (e.g. [Nginx](https://nginx.org/en/)) intercept requests and serve cached content:

* 📌 Highly efficient for static assets, also for certain dynamic content if configured well.
* 📌 Next.js on Vercel automatically integrates with edge caching for pages (especially in static generation scenarios).
* ✅ Gives massive performance gains for global users.
* ✅ Offloads traffic from your origin server.
* ⚠️ Requires advanced setup for dynamic, user-specific data can be complex.
* ⚠️ Might introduce cache invalidation complexities (stale data, etc.).

### Next.js In-Memory Caching

When fetching data in React Server Components, Next.js offers a built-in caching mechanism. You can leverage this by using the built-in `fetch` function’s caching options.

* ✅ Fully integrated with Next.js’ revalidation flow (*ISR*).
* ✅ Minimal configuration for typical use cases.
* ⚠️ Not suitable for large amounts of data or data with immediate real-time requirements.
* ⚠️ For advanced scenarios, you may need a dedicated caching solution (like [Redis](https://redis.io/)).

### Edge Caching

Edge caching is caching content at the network’s edge, physically closer to the user:

* 📌 Reduces [round-trip time (RTT)](https://aws.amazon.com/what-is/rtt-in-networking/).
* 📌 Allows for [geographical load balancing](https://www.vmware.com/topics/load-balancing#:~:text=Geographic%20%E2%80%94%20Geographic%20load%20balancing%20redistributes,data%20centers%20in%20many%20locations.).
* 📌 On Vercel, *Static Site Generation* pages are cached globally at edge locations. When using *Incremental Static Regeneration*, outdated pages are invalidated and [regenerated in the background](https://vercel.com/docs/incremental-static-regeneration/quickstart#background-revalidation).
* ✅ Speed improvements for global users.
* ✅ Integrates seamlessly with Next.js.
* ⚠️ Some advanced dynamic use cases might require custom logic for invalidating or bypassing the cache.

## Data Fetching

When calling `fetch`, you can specify caching and revalidation behavior:

* **cache**: `force-cache` | `no-store` | `only-if-cached`
* **next.revalidate**: number of seconds to revalidate the data

```
const data = await fetch("https://api.example.com/data", {
  cache: "force-cache", // or "no-store"
  next: { revalidate: 60 }, // revalidate after 60s
});
```

* **force-cache**: Enforces caching at the Next.js layer.
* **no-store**: Bypasses caching and always fetches fresh data.
* **revalidate**: Tells Next.js how long to keep the cached version before revalidating in the background.

## Revalidation

Revalidation is the process of invalidating stale content and serving an up-to-date version without incurring the full cost of re-rendering on every request. Next.js provides multiple ways to handle revalidation, each suited to different scenarios.

### Incremental Static Regeneration (ISR)

*ISR* allows you to generate static pages at build time and then revalidate them incrementally when they’re requested again. Here’s a brief flow:

1. User requests a page that is statically generated and cached at build time.
2. The page is served from the cache until the revalidate time window passes.
3. After the revalidate period, the next incoming request triggers Next.js to regenerate the page in the background.

In the App Router, you can configure ISR using `fetch` options or [metadata](#route-segment-config) in your route.

```
export default async function ProductsPage() {
  const products = await fetch("https://api.example.com/products", {
    cache: "force-cache",
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  }).then((res) => res.json());

  return (
    <div>
      <h1>Our Products</h1>
      <ul>
        {products.map((product: { id: number; name: string }) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### Route Segment Config

In Next.js, route metadata provides configuration options that can define how routes behave, including caching, revalidation, and rendering strategies. These options are defined as static exports within a route file, and they enable developers to easily configure settings like `revalidate`, `dynamic`, and more.

You can learn more about different segment options in the [Next.js docs](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config).

### On-Demand Revalidation

*On-Demand Revalidation* lets you invalidate cached pages or data immediately rather than waiting for the revalidate window. This is usually triggered by some external event - for example when a CMS entry is updated. You can have multiple route handlers in `app/api/...` directories, and each handler can handle revalidation for different paths or even multiple paths. For example, you can:

* Revalidate an entire blog section: `/blog/\*`.
* Revalidate a specific path: `/blog/my-post`.
* Revalidate by tags (useful if you have logical groupings of pages).

This way of revalidation ensures your site can be kept up-to-date in near real time, and is great for content-heavy sites with frequent updates.

You can find an example of API handler that revalidates tags sent in search params [later in this article](#complex-revalidation-route-handler-with-secret-key-guard), with some modifications this can be used as a Webhook triggered by CMS when the content changes.

### Fallback Strategies

When regenerating pages, Next.js will serve either:

* The stale page until the new version is ready ([stale-while-revalidate](https://nextjs.org/docs/app/building-your-application/caching#time-based-revalidation:~:text=This%20is%20similar%20to%20stale%2Dwhile%2Drevalidate%20behavior.)).
* A loading or error state if the data is crucial to load.

You can control these states via the new `loading.tsx` and `error.tsx` files in the App Router. This approach keeps the user experience seamless, even if the data is being re-fetched in the background.

## Recipes & Code Snippets

### Caching with `fetch`

Fetch data from an external API and cache it for 5 minutes, then revalidate.

```
// app/(dashboard)/page.tsx
type DashboardData = {
  totalUsers: number;
  onlineUsers: number;
};

export default async function Dashboard() {
  // Revalidate after 5 minutes (300 seconds)
  const data = await fetch("https://api.example.com/dashboard", {
    next: { revalidate: 300 },
  }).then((res) => res.json() as Promise<DashboardData>);

  return (
    <section>
      <h1>Dashboard</h1>
      <p>Total Users: {data.totalUsers}</p>
      <p>Online Users: {data.onlineUsers}</p>
    </section>
  );
}
```

### Mutate then Revalidate

Show an up-to-date list of items after a user adds a new item.

**Step 1: Create the Page**

```
// app/items/page.tsx
import { ItemsList } from "./_components/ItemsList";
import { ItemsForm } from "./_components/ItemsForm";

export default function ItemsPage() {
  return (
    <div>
      <ItemsForm />
      <ItemsList />
    </div>
  );
}

export const dynamic = "force-dynamic";
```

`ItemsPage` is a Server Component that brings together:

* `ItemsForm`, a Client Component for adding items.
* `ItemsList`, a Server Component for displaying the items.

**Step 2: Create a Server Action**

```
// app/actions/revalidateItems.ts
"use server";

import { revalidatePath } from "next/cache";

// Revalidates the `/items` route to ensure fresh data
export async function revalidateItems() {
  revalidatePath("/items");
}
```

This Server Action manually triggers revalidation of the `/items` page. It forces Next.js to refresh all data for that route, including the `ItemsList` component.

**Step 3: Create the ItemsList component**

```
// app/items/_components/ItemsList.tsx
export async function ItemsList() {
  const data = await fetch("http://localhost:3000/api/items", {
    next: {
      revalidate: 60,
    },
  }).then((res) => res.json());

  return (
    <ul>
      {data.items.map((item: string, idx: number) => (
        <li key={idx}>{item}</li>
      ))}
    </ul>
  );
}
```

This component fetches data from the API and displays the list of items.

`next.revalidate: 60` caches the response for 60 seconds. This means:

* If a user refreshes the page within 60 seconds, they see the cached data.
* After 60 seconds, the cache expires, and a new request is made.
* When `revalidateItems()` is called, it forces the `/items` route to refresh its data immediately, bypassing the cache.

**Step 4: Create the ItemsForm component**

```
// app/items/_components/ItemsForm.tsx
"use client";

import { useState, useTransition } from "react";
import { revalidateItems } from "../../actions/revalidateItems"; // Import revalidateItems action

export function ItemsForm() {
  const [newItem, setNewItem] = useState("");
  const [shouldRevalidate, setShouldRevalidate] = useState(true);
  const [isPending, startTransition] = useTransition();

  const handleAddItem = async () => {
    await fetch("/api/items", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ newItem }),
    });

    setNewItem(""); // Clear the input field

    // Conditionally trigger revalidation and refresh
    if (shouldRevalidate) {
      startTransition(() => {
        revalidateItems(); // Manually revalidate the `/items` route
      });
    }
  };

  return (
    <div>
      <input
        type="text"
        value={newItem}
        onChange={(e) => setNewItem(e.target.value)}
      />
      <button onClick={handleAddItem} disabled={isPending}>
        {isPending ? "Adding..." : "Add Item"}
      </button>

      <div>
        <label>
          <input
            type="checkbox"
            checked={shouldRevalidate}
            onChange={(e) => setShouldRevalidate(e.target.checked)}
          />
          Revalidate items after adding
        </label>
      </div>
    </div>
  );
}
```

* This component handles state (`useState`) and asynchronous actions (`useTransition`).
* The checkbox (`shouldRevalidate`) determines whether the `/items` route should be revalidated after a new item is added.
* If the checkbox is checked, `revalidateItems()` is called to refresh the `/items` route immediately.
* `startTransition` allows React to prioritize UI updates (like clearing the input field) while handling revalidation in the background without blocking.

**Step 5: Create `/api/items/` route handler**

```
// app/api/items.ts
import { NextResponse } from "next/server";

const items: string[] = ["Item A", "Item B"]; // Temporary in-memory storage

export async function GET() {
  return NextResponse.json({ items });
}

// Revalidate inside API route after modifying data
export async function POST(request: Request) {
  const { newItem } = await request.json();
  items.push(newItem);

  return NextResponse.json({ success: true });
}
```

**Step 6: Testing Cache Behavior**

1. Open Two Browser Windows
   * Open the ItemsPage in two separate browser windows or tabs.
2. Add a New Item in the First Window
   * Enter a new item in the input field and click "Add Item."
   * If the checkbox is checked:
     * The revalidateItems action is triggered, clearing the cache for /items.
     * startTransition ensures that ItemsList re-renders automatically in the first window to show fresh data.
   * If the checkbox is unchecked:
     * The API route updates the data, but no revalidation occurs. Both windows continue showing cached data until the cache expires (e.g., 60 seconds).
3. Check the Second Window
   * After adding an item in the first window:
     * The second window does not automatically update because it does not share the same React tree as the first window.
     * Manually refreshing the second window fetches fresh data because the cache has been invalidated.
4. Wait for Cache Expiration
   * If no revalidation is triggered, both windows will continue showing stale data until the cache expires based on next.revalidate.

**How It All Works Together**

In this setup, the `ItemsForm` component handles adding new items and optionally revalidating the `/items` route. Here's why this approach functions effectively:

1. Server Action for Revalidation:
   * The `revalidateItems()` function is a server action that calls `revalidatePath("/items")`, invalidating the cache for the `/items` page.
2. Client-Side Interaction:
   * When a new item is added via the form, the `handleAddItem()` function sends a `POST` request to the `/api/items` endpoint to add the item.
   * If `shouldRevalidate` is true, `startTransition()` is used to call `revalidateItems()`.
3. Automatic React Tree Refresh:
   * Calling `revalidateItems()` within `startTransition()` informs React that a low-priority update is occurring, prompting it to re-render components in current React tree.
   * This mechanism ensures that the `ItemsList` component fetches the updated data without requiring an explicit call to `router.refresh()`.
   * The second window does not automatically update because it does not share the same React tree as the first window.

**Why this is just a demonstration**

1. Revalidating in the Server Action is not the best practice:
   * In production, revalidating in the API route (e.g. in the `POST` handler) is the preferred approach. This centralizes cache invalidation with the data mutation.
   * If revalidation is part of the API logic, any client (not just React components) calling the API benefits from consistent behavior.
2. Server Actions Are Useful for Tight Integration:
   * Revalidating in a Server Action (`revalidateItems()`) works well when the mutation and the affected React tree are tightly coupled, as demonstrated here.

Also, keep in mind that `startTransition()` will not update the `ItemsList` component if the `revalidatePath()` is called inside `POST` handler, this happens because:

1. When a Server Action is invoked, React treats it as part of the app's state. This tight integration allows React to re-render affected Server Components automatically when the action is used inside `startTransition()`.
2. API Route Revalidation Isn’t Tied to React:
   * When revalidation is triggered in an API route, React has no direct way to know that the cache was invalidated.
   * This is why an explicit mechanism like `router.refresh()` is required when relying solely on API route-based revalidation.

### Using `use` Hook in Server Components

Simplify data fetching in server components with the built-in `use` hook.

```

// app/profile/page.tsx
import { use } from "react";

async function getProfileData(userId: string) {
const res = await fetch(`https://api.example.com/user/${userId}`);
return res.json();
}

export default function ProfilePage({ userId }: { userId: string }) {
const data = use(getProfileData(userId));

return ( <div> <h1>{data.name}</h1> <p>{data.bio}</p> </div>
);
}

```

The `use()` hook is also capable of reading React Context, you can read more about it in the [React Docs](https://react.dev/reference/react/use).

### Optimistic UI & Reactive Updates

While not strictly a server-side caching strategy, Optimistic UI is a method to instantaneously show updated data while the server operation is still pending. This is typically done client-side, but you can combine it with Next.js:

1. Send an update request (e.g., mutate a resource).
2. Locally update the UI to reflect the change before the server confirms it.
3. Revalidate or refetch to ensure your local UI matches the server’s state.

As an example, you can use the code from [Mutate then Revalidate recipe](#mutate-then-revalidate) and change the `handleAddItem()` method in `ItemsForm` component:

```
 const handleAddItem = async () => {
    const optimisticItem = newItem;

    // Update UI Optimistically
    setItems((prev) => [...prev, optimisticItem]);
    setNewItem(""); // Clear input

    try {
      await fetch("/api/items", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ newItem }),
      });

      // Optional: Refetch the data to ensure consistency
    } catch (error) {
      console.error("Failed to add item:", error);
      setItems((prev) => prev.filter((item) => item !== optimisticItem)); // Revert optimistic update
    }
  };
```

### Using 3rd-Party Caching Layers (e.g. Redis)

For high-traffic or data-intensive applications, you may need an external caching service like *Redis* - it's often used for ephemeral caches, session storage, and real-time data

### Complex revalidation route handler with secret key guard

```
import { revalidateTag } from 'next/cache';
import { NextRequest, NextResponse } from 'next/server';

export enum RequestTag {
	GetItems = 'GetItems',
	GetUserProfile = 'GetUserProfile',
}

const withRevalidateCacheRouteGuard =
	(handler: (request: NextRequest) => Promise<NextResponse>) => async (req: NextRequest) => {
		if (req.method !== 'GET') {
			return new NextResponse('Method not allowed', {
				status: 405,
			});
		}

		const requestSecret = req.headers.get('x-api-secret');

		if (!requestSecret || requestSecret !== process.env.REVALIDATE_CACHE_SECRET) {
			return new NextResponse('Forbidden: Invalid secret', {
				status: 403,
			});
		}

		return handler(req);
	};

const validateTags = (tagsToValidate: string[]) => {
	const allTags = Object.values(RequestTag);

	tagsToValidate.forEach((tag) => {
		if (!allTags.includes(tag as RequestTag)) {
			throw new Error(`Invalid tag provided for revalidation: ${tag}`);
		}
	});
};

export const GET = withRevalidateCacheRouteGuard(async (req: NextRequest) => {
	try {
		const tagsSearchParam = req.nextUrl.searchParams.get('tags');

		// Revalidate all if there are no tags provided
		if (!tagsSearchParam) {
			const allTags = Object.values(RequestTag);

			for (const tag of allTags) {
				revalidateTag(tag);
			}

			return new NextResponse(
				JSON.stringify({
					success: true,
					message: `Cache revalidated successfully for all tags: ${allTags.join(', ')}`,
				}),
				{
					status: 200,
					headers: { 'Content-Type': 'application/json' },
				}
			);
		} else {
			// Revalidate tags provided
			const tags = tagsSearchParam.split(',');

			validateTags(tags);

			for (const tag of tags) {
				revalidateTag(tag);
			}

			return new NextResponse(
				JSON.stringify({ success: true, message: `Cache revalidated successfully for tags: ${tags.join(', ')}` }),
				{
					status: 200,
					headers: { 'Content-Type': 'application/json' },
				}
			);
		}
	} catch (error: unknown) {
		const errorMessage = (error as Error).message || 'Internal server error revalidating cache.';

		return new NextResponse(JSON.stringify({ success: false, message: `Revalidation cache failed: ${errorMessage}` }), {
			status: 400,
			headers: { 'Content-Type': 'application/json' },
		});
	}
});
```

> In a typical project, you’d place `RequestTag` in a dedicated utility file or an `_enums/` directory to make it reusable across the application, rather than keeping it directly inside a single route. This keeps your code organized and makes tags accessible to any component or function that needs them for revalidation logic.

This route handler demonstrates a secure, structured approach for revalidating Next.js caches based on user-defined tags. It ensures that only valid requests carrying the correct API secret can initiate cache revalidation, enabling fine-grained control over which cached resources get updated. This can be especially useful in scenarios where you need to trigger cache revalidation on demand. For example, you may want to quickly revalidate certain tags when the content changed in database, ensuring that any outdated data is refreshed (if you don't have Webhook based on-demand revalidation). In such cases, you can send a `GET` request (via *cURL*, *Postman*, or a *CI script*) to this endpoint, including the required secret header.

1. Purpose & Flow
   * Defines a secure endpoint that only accepts `GET` requests and requires a secret key to revalidate cached data.
   * If the request’s secret key is invalid or missing, the handler rejects the request.
2. Tag Revalidation
   * Uses a custom enum to define valid cache tags.
   * Ensures only recognized tags can be revalidated, throwing an error for any invalid ones.
3. Handling Different Inputs
   * When no specific tags are sent, all known tags are revalidated.
   * When specific tags are sent (comma-separated), each one is validated and then revalidated.
4. Error Handling
   * Employs a try/catch structure.
   * Responds with detailed error messages in a consistent JSON format.
5. Extensibility
   * Straightforward to add more tags in the enum or adapt for additional logic.
   * Wraps the core logic in a higher-order function to keep the request checks (method, secret) separate from revalidation logic.

To use the `RequestTag` enum for on-demand or scheduled revalidation, you can pass it in the fetch options as shown below. When Next.js receives a subsequent request to this URL, it knows to cache (and later revalidate) based on the tag:

```
await fetch('/api/items', {
  next: {
    revalidate: 60 * 60, // 60 minutes
    tags: [RequestTag.GetItems],
  },
});
```

## Best Practices & Common Pitfalls

1. Choose the Right Revalidate Interval
   * A short interval ensures fresher data but may cause more frequent regeneration.
   * A long interval reduces server load but risks serving stale content.
2. Leverage Edge Functions (if on *Vercel*)
   * Serve cached content closer to the user for faster performance, especially for global audiences.
   * Combine with revalidate to ensure data is kept fresh.
3. Use `no-store` for Highly Dynamic Content
   * If your data changes very frequently (e.g., stock prices), skip caching to prevent stale data.
   * Alternatively, consider partial caching strategies or websockets.
4. Be Mindful of Memory Footprint
   * Next.js in-memory caching is stored in the server’s memory, so large volumes of data can be expensive.
   * Offload large data sets to external caches (e.g., *Redis*) where needed.
5. Handle Errors Gracefully
   * If a fetch call fails, ensure your UI can handle it.
   * Use error.tsx or try-catch blocks around fetch to display fallback UI.
6. Check for overfetching
   * In server components, if multiple components fetch the same data, consider consolidating into a shared data-fetching layer or using cache() to reduce redundant calls.
7. Security & Authorization
   * Ensure no sensitive data is inadvertently cached or served to unauthorized users.
   * Use tokens or session-based approaches carefully, especially in SSR contexts.
8. Monitor & Test
   * Use performance monitoring tools to see real-world impacts of your caching strategy.
   * A/B test or canary test changes to your caching or revalidation to ensure no regressions.

### Memory Leak in Node.js

In some Node.js versions (e.g. `20.16.0`), we identified a memory leak in Next.js' `fetch` API. Each request using `fetch()` adds an entry to the in-memory cache but never clears it, potentially leading to an out-of-memory exception after multiple requests.

The simplest workaround is to downgrade Node.js to `20.15.1`. If you need to use a newer version than `20.16.x`, monitor memory usage to ensure stability.

For more details, refer to [this GitHub Discussion](https://github.com/vercel/next.js/discussions/68636).

### Changing cache key on every request

In Next.js, when caching fetch API calls, the cache key is generated based on the entire request object, including headers. This means that if your requests have unique headers per user—such as authorization tokens or cookies—the cache will store separate entries for each unique request. Consequently, with a large number of users and requests, this can lead to a significant number of cache entries, potentially causing an out-of-memory exception.

For instance, if you have 100 authorized users, each making 10 requests with unique headers, the cache could accumulate 1,000 distinct entries, even if the requested content is identical across users.

To mitigate this issue, consider implementing a custom caching strategy that normalizes cache keys for identical content, regardless of user-specific headers. This approach can help reduce redundant cache entries and manage memory usage more effectively.

For a deeper understanding of the cache key generation, you can review the [Next.js source code](https://github.com/vercel/next.js/blob/e02397d26473ac41a3d6dd6a396769b584dd0e49/packages/next/src/server/lib/patch-fetch.ts#L296) related to this functionality.

### Middleware affecting cache behavior

This is similar to the previous pitfall - middleware can modify requests and responses dynamically, which may lead to unexpected cache behavior. If a middleware alters headers, query parameters, or cookies before reaching a cached page, it can create different cache variations than expected.

For example, if middleware adds a `Set-Cookie` header dynamically, Next.js might treat every response as unique, leading to cache fragmentation.

To prevent this, ensure middleware does not modify request attributes that contribute to cache keys unless intentional.

### Misunderstaing Cache Revalidation

A common misconception in Next.js caching is that triggering cache revalidation (e.g., using `revalidatePath()` or `revalidateTag()`) immediately removes old cached data from memory or disk. Many developers assume that revalidation "clears" the previous cache, freeing up memory or storage.

However, cache revalidation does not delete the old data. Instead:

* Revalidation invalidates the cache entry, marking it as "stale."
* The old data remains in memory or disk until the next request to the revalidated route is made.
* Only when a new request is processed does Next.js fetch fresh data, update the cache, and remove the stale entry.

This behavior can lead to unexpected memory or storage issues, especially in scenarios with frequent revalidations:

* Old cache entries remain in memory, consuming valuable resources.
* If many paths are invalidated but not requested, the server's memory usage can grow significantly over time.

### Over-Caching dynamic data

By default, Next.js caches `fetch` calls indefinitely when used in static site generation. This means that if you're fetching dynamic data without specifying a caching strategy, you may end up serving stale content until the next rebuild.

For example, if your application fetches live stock prices but caches the response at build time, users will see outdated values until you trigger a new deployment.

To prevent this, set `cache: 'no-store'` on `fetch()` for truly dynamic data. If partial caching is needed, use `next: { revalidate: X }` to refresh the data periodically while still benefiting from caching.

### `revalidate: 0` Misconception

Setting `revalidate: 0` does not enable *ISR* - it actually forces server-side rendering on every request. This means every page load triggers a fresh request to the data source, which can significantly impact performance.

If your intent is to cache data but always refresh immediately, consider using a very low revalidate value instead (e.g., `revalidate: 1` for near real-time updates with caching).

### Stale Data in Incremental Static Regeneration

*ISR* allows pages to be updated in the background, but users might still see stale data until a revalidation occurs. This is especially problematic for time-sensitive content like breaking news or live sports scores.

For example, if a blog post is updated but the revalidate interval is set to 10 minutes, users could still see old content for up to 10 minutes before Next.js regenerates the page.

To balance freshness and performance, choose an appropriate revalidate interval based on how often the content changes. For near real-time updates, consider switching to server-side rendering or client-side fetching instead of relying on ISR.

### Development vs Production caching

In development mode, Next.js does not cache `fetch` requests. This means that while your application might appear to fetch fresh data on every request during development, caching will behave differently in production.

For example, if you're testing an API response that should be cached for 5 minutes, you won't see this behavior locally. Instead, run the production build in Docker or deploy the app to a staging environment to test real-world caching behavior.

## Further Reading & Resources

* [Next.js Documentation on Data Fetching](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching)
* [React Server Components](https://react.dev/reference/rsc/server-components)
* [Caching on Vercel's Edge Network](https://vercel.com/docs/edge-network/caching)
