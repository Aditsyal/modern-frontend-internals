# Hydration: The Resurrection of State

Hydration is the process of attaching event listeners and internal state to a static HTML document previously rendered on the server. In an SSR (Server-Side Rendering) or SSG (Static Site Generation) workflow, the server ships a "dead" UI; a string of HTML that looks correct but responds to nothing. Hydration "wakes" this UI up by running the client-side JavaScript, reconciling the DOM, and binding the framework's internal logic to existing nodes.

## The Technical Reality

Hydration is fundamentally a **reconciliation** task. When the client-side bundle executes, the framework re-renders the entire application in memory. It then walks the existing DOM and attempts to "claim" each node. Instead of calling `document.createElement`, it matches its virtual nodes to the physical nodes already present.

This process creates the **Uncanny Valley of Interactivity**: the period where a page is visually complete (LCP is met) but the main thread is locked by hydration logic. If a user clicks a button during this window, nothing happens.

## The Hydration Mismatch

A mismatch occurs when the server's HTML does not perfectly align with the client's first render. This is usually caused by:

- **Non-deterministic data:** Using `Math.random()` or `new Date()` directly in the render path.
- **Environmental differences:** Accessing `window` or `localStorage` which only exist on the client.
- **Third-party browser extensions:** Tools like password managers or ad-blockers that inject nodes into the DOM before hydration completes.

When a mismatch happens in React 18+, the framework will no longer attempt to "patch" individual nodes. Instead, it will throw a hydration error and revert to client-side rendering for the closest `<Suspense>` boundary. This ensures consistency but at a significant performance cost.

## Selective Hydration

React 18 introduced **Selective Hydration**, which allows the browser to hydrate parts of the page independently. By wrapping components in `<Suspense>`, you create boundaries that React can hydrate in isolation. If a user interacts with a component that hasn't hydrated yet (e.g., clicks a button), React will prioritize hydrating that specific subtree to respond to the user action immediately.

## Beyond Hydration: Resumability

While frameworks like React and Vue focus on "Optimizing Hydration," others like **Qwik** are pioneering **Resumability**.

Traditional hydration is "eager": the browser must download and execute JS to rebuild the state. Resumability is "lazy": the server serializes the entire state (including event listeners) into the HTML. When the user clicks a button, the browser downloads *only* the code for that specific event handler and executes it. There is no "Uncanny Valley" because there is no execution cost on page load.

## Pattern: Client-Only Escapes

To avoid mismatches for environment-specific UI, use a "double-render" pattern with `useEffect` or the `suppressHydrationWarning` prop for minor, unavoidable differences (like timestamps).

```ts
import React, { useState, useEffect } from 'react';

interface Props {
  children: React.ReactNode;
}

export function ClientOnly({ children }: Props) {
  const [hasMounted, setHasMounted] = useState(false);

  useEffect(() => {
    setHasMounted(true);
  }, []);

  if (!hasMounted) {
    return null; // Or a server-rendered skeleton
  }

  return <>{children}</>;
}
```

## Pragmatic Evaluation

> **The Philosophy:** Hydration is a necessary evil in current mainstream frameworks. Minimize its impact by using Selective Hydration and avoiding unnecessary client-side state on initial load.

### When to use this

- **SEO-critical pages:** Where fast First Contentful Paint (FCP) is required.
- **React/Next.js Apps:** Standard for providing interactivity to server-rendered content.

### When to avoid

- **Purely Static Sites:** Use SSG without a JS runtime if no interactivity is needed.
- **Low-Bandwidth Scenarios:** Consider Resumability (Qwik) to avoid the "JS tax" entirely.

---

Related [Selective Hydration](../react-core/suspense.md)