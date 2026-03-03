# Concurrent Rendering: Understanding Fluidity

Concurrent Rendering is a fundamental shift in how React manages the main thread. Historically, rendering was a single, uninterruptible transaction. Once React started rendering a large tree, it would block the browser until it finished. Concurrent mode makes rendering **interruptible**.

## The HOV Lane Analogy

Think of traditional rendering as a single-lane road. If a heavy data-grid (the truck) is rendering, a user's click (the ambulance) is stuck behind it.

Concurrent rendering turns the main thread into a multi-lane highway with a priority system:

- **Urgent Tasks:** Input events, clicks, and animations.
- **Transition Tasks:** Search results, page navigations, and heavy list filtering.

If a user types while a heavy render is in progress, React will **pause** the heavy render, handle the keystroke to keep the UI responsive, and then resume the render in the background.

## Key APIs: `useTransition` and `useDeferredValue`

These hooks allow developers to categorize updates.

```javascript
const [isPending, startTransition] = useTransition();

const handleSearch = (e) => {
  // 1. High Priority: Update the input field immediately
  setQuery(e.target.value);

  // 2. Low Priority: Start the expensive search filtering
  startTransition(() => {
    setFilteredResults(expensiveFilter(e.target.value));
  });
};
```

While `useTransition` is for **imperative** updates (actions), `useDeferredValue` is for **declarative** props or state that should lag behind the main update.

```javascript
const deferredQuery = useDeferredValue(query);

// This component will only re-render with the new value once
// the main thread is idle, preventing input lag.
return <ExpensiveList query={deferredQuery} />;
```

## Double Buffering & The Alternate Pointer

Concurrent rendering uses a "Double Buffering" technique similar to game engines. React maintains a **Current Fiber Tree** (the one the user sees) and prepares a **Work-In-Progress (WIP) Tree**.

### The Alternate Pointer

Every Fiber node contains an `alternate` pointer. This pointer links the "Current" Fiber to its "Work-In-Progress" counterpart.

- **Phase 1: Render (Interruptible):** React builds the WIP tree in memory. If a higher-priority task arrives, React can discard the WIP tree and restart, or pause and resume.
- **Phase 2: Commit (Synchronous):** Once the WIP tree is complete and all Suspense boundaries are resolved, React "swaps" the trees in a single, synchronous operation (the Commit phase).

```typescript
// The Fiber Node property that enables double-buffering
interface Fiber {
  // ...
  alternate: Fiber | null; // Pointer to the mirror version of this fiber
}
```

Because the WIP tree is rendered in memory and can be interrupted, the user never sees "partial" states. React only "swaps" the trees once the entire WIP tree is consistent.

## Impact on INP (Interaction to Next Paint)

Google's INP metric rewards apps that acknowledge user input within 200ms. Concurrent rendering is the primary tool for passing this metric. It ensures that even during massive state transitions, the main thread yields to the browser every 5ms, allowing the browser to paint and respond to the user.

## Pragmatic Evaluation

> **The Philosophy:** Interruptibility is the key to balancing complex UI states with smooth user interactions on a single-threaded runtime.

### When to use this

- **Heavy UI Tasks:** Use `useTransition` for search results, filtering large lists, or switching between complex tabs.
- **Improving INP:** If your Lighthouse scores show poor "Interaction to Next Paint," concurrent rendering is the solution.

### When to avoid

- **Simple State Updates:** Do not wrap every state change in a transition. This overhead can actually slow down simple interactions.
- **External State Management:** If you use a non-React state manager (e.g., global variables) without proper integration, you risk **Tearing**.

---

Related [Fiber Architecture](./fiber.md), [Time Slicing](../browser-engine/time-slicing.md)
