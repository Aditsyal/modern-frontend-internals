# Suspense Boundaries and Selective Hydration

Suspense is not just a loading spinner; it is a mechanism for **coordinating the delivery of UI**. It allows React to treat "asynchronous readiness" as a first-class citizen in the component tree, orchestrating how and when different parts of the application become visible and interactive.

## 1. Selective Hydration: The Priority Shift

In traditional SSR, hydration is a "waterfall" event: the browser must fetch all JavaScript, and React must hydrate the entire page in a single pass. If a heavy component (like a complex data table) is slow to hydrate, it blocks the interactivity of everything else.

**Selective Hydration** solves this through two main mechanisms:

1.  **Independent Hydration:** React hydrates each `<Suspense>` boundary independently. If the main content is ready but the sidebar is still "suspended" (or just heavy), React can hydrate the main content first.
2.  **Interaction-Driven Prioritization:** If a user clicks a button inside a boundary that hasn't hydrated yet, React will **pause** its current hydration work and jump to that specific boundary. It "fast-tracks" the code necessary to respond to that user's specific intent.

## 2. Streaming Integration

When the server encounters a Suspense boundary, it doesn't wait. It streams the "fallback" UI to the browser immediately and continues rendering the rest of the page. Once the data resolves on the server, React streams the final HTML and a small "swap script."

```html
1. Initial stream
<div id="suspense-1">Loading...</div>

2. Later stream
<template id="suspense-1-content">
  <div>Real Content</div>
</template>
<script>
  // The $RS (or similar internal) script swaps the content
  $RS("suspense-1", "suspense-1-content");
</script>
```

## 3. React 19 and the `use` API

In React 19, Suspense becomes the primary way to handle data fetching via the `use` API. When a component calls `use(promise)`, it "suspends" if the promise is pending. This moves the responsibility of "checking for data" from the component logic to the framework's Fiber reconciler.

- **Unified Orchestration:** You can wrap multiple components in a single Suspense boundary to ensure they "reveal" together, preventing "UI popping."
- **Data Flow:** Because `use` can be conditional, you can fetch data only when needed without breaking the component's structure.

## 4. The "Uncanny Valley" Solution

The combination of Streaming SSR and Selective Hydration eliminates the "frozen page" problem -- where a page looks ready but isn't interactive. The page becomes interactive in chunks, and the most critical chunk (the one the user is actually interacting with) is prioritized by the engine.

---

## Pragmatic Evaluation

> **The Philosophy:** Trading "Initial Page Load" time (which is minimized via fallbacks) for "Time to Interactivity" (which is optimized via selective hydration).

### When to use this

- **Dashboard Interfaces:** Where different widgets load at different speeds and shouldn't block each other.
- **E-commerce Product Pages:** To ensure the "Add to Cart" button is interactive as soon as possible, even if reviews or related products are still loading.
- **Content-Rich Sites:** To provide a perceived performance boost by showing content as it becomes available.

### When to avoid

- **Extremely Low-Bandwidth Environments:** If the "swap scripts" and template tags add too much byte overhead compared to a single static HTML blob.
- **Simple, Fast APIs:** If your data fetching is consistently sub-100ms, the overhead of managing boundaries might outweigh the benefits.

---

Related [Concurrent Rendering](../react-core/concurrent-rendering.md), [React 19 Standards](../react-core/react-19.md)
