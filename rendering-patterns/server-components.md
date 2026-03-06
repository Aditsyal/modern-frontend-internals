# React Server Components (RSC)

React Server Components (RSC) are not "SSR 2.0". They are a fundamental re-engineering of the network boundary. While SSR focuses on turning a component tree into HTML for the initial load, RSCs focus on what parts of the tree *stay* on the server even after the page has loaded. They work together: SSR provides the fast initial HTML, and RSC provides the efficient, JS-free updates for the components that don't need interactivity.

## The RSC Model: Data is the Component

In the RSC model, **the component is the data fetcher.** In traditional React, you send a JS bundle to the browser, which then sends a request back to an API, which then queries a database. RSC cuts the middleman. The component runs on the server, queries the database directly, and streams the result. The browser receives 0 bytes of JavaScript for that component -- only the final UI instructions.

## The Server vs. Client Divide


| Server Components (Rendering)             | Client Components ("use client")           |
| ----------------------------------------- | ------------------------------------------ |
| **0 JS sent to the browser.**             | Bundled and shipped to the user.           |
| Directly access DB, File System, Secrets. | Access `window`, `localStorage`, Sensors.  |
| Use `async/await` for data fetching.      | Use `useEffect` or `useQuery` hooks.       |
| Cannot use state or interactivity.        | The home of `useState` and event handlers. |


## Server Actions: The Mutation Side

If RSCs are for **rendering**, Server Actions are for **mutating**. By marking a function with `"use server"`, you create a secure, RPC-like entry point that the client can call. React handles the `POST` request, the serialization of arguments, and (crucially) the **automatic re-validation** of the current page data after the mutation completes.

## The Serialization Boundary: The Bridge

When data moves from the Server to the Client, it crosses a "Serialization Boundary". Think of this like shipping furniture: you can't ship a fully assembled, custom-built sofa (a Class instance or a Function); you have to ship the instructions and the raw parts (JSON-compatible data) for the Client to assemble.

### What passes the boundary?

1. **JSON-compatible primitives:** Strings, numbers, booleans, and nulls.
2. **Plain Objects and Arrays:** If they contain serializable values.
3. **Promises:** You can pass a promise from the server to the client (for streaming data).
4. **Server Actions:** References to functions marked with `"use server"`.

### What fails?

1. **Functions:** Client components cannot receive non-action functions from the server.
2. **Class Instances:** Only the data properties survive; methods are lost.
3. **Cyclic Data Structures:** Objects that reference themselves will cause serialization errors.

## Killing the Data Waterfall

Waterfalls happen when a parent component waits for data, renders a child, and only *then* does the child start its own fetch. On the server, RSCs can kick off multiple database queries in parallel. Since the server is physically closer to the data source (microseconds of latency instead of milliseconds), the entire tree resolves near-instantly.

```ts
// Parallel execution on the server backbone
async function Dashboard({ userId }: { userId: string }) {
  // Both queries start immediately
  const userPromise = db.users.find(userId);
  const postsPromise = db.posts.find(userId);

  const [user, posts] = await Promise.all([userPromise, postsPromise]);

  return (
    <section>
      <Profile user={user} />
      <Suspense fallback={<Loader />}>
        <Feed posts={posts} />
      </Suspense>
    </section>
  );
}
```

## The RSC Payload (Wire Format)

RSC does not return HTML. It returns a specialized **RSC Payload** -- a serialized representation of the component tree. This payload includes:

1. The rendered output of Server Components.
2. Placeholders where Client Components should be inserted.
3. The props passed from Server to Client.

This is why you can navigate between pages in an RSC app and keep your scroll position, input focus, and local state -- the browser isn't reloading the page; it's "reconciling" the payload into the existing DOM.

## Pragmatic Evaluation

> **The Philosophy:** Shift complexity and data-fetching costs to the server to maximize client-side performance and minimize bundle size.

### When to use this

- **Data-Heavy Applications:** Dashboards, e-commerce listings, and content-rich pages where SEO and performance are paramount.
- **Security-Sensitive Logic:** When interacting with secrets or direct database access that shouldn't be exposed to the client.
- **Reducing JS Fatigue:** When you want to minimize the amount of JavaScript the user's browser has to parse and execute.

### When to avoid

- **Highly Interactive UIs:** Simple widgets, calculators, or purely client-side states where the overhead of a network round-trip for every state change is unacceptable.
- **Offline-First Requirements:** RSC requires a live server connection to render and update the tree.

---

Related [Selective Hydration](../react-core/suspense.md)