# Scheduler Priorities and React Lanes

The React Scheduler is the traffic controller of the UI. While the `scheduler` package manages a general-purpose priority queue using numeric levels, React internally maps these to a more granular system called **Lanes**.

## The Priority Hierarchy (Scheduler)

The Scheduler defines five levels, each with a specific timeout (expiration time). If a task stays in the queue too long, it "expires" and is forced to run immediately.

1.  **Immediate Priority:** Synchronous tasks (e.g., input focus, discrete events).
2.  **User-Blocking Priority:** UI interactions (e.g., clicks, typing). Expiration: 250ms.
3.  **Normal Priority:** The default (e.g., data fetching, background updates). Expiration: 5000ms.
4.  **Low Priority:** Non-urgent background work (e.g., logging, off-screen rendering).
5.  **Idle Priority:** Tasks that run only when the browser has nothing else to do.

## The Expert Nuance: React Lanes (Bitmasks)

While the Scheduler uses numeric levels, React 17+ uses **Lanes** (31-bit integers) to track updates. This shift from "priority levels" to "bitmasks" allows for sophisticated update management.

### The Bitmask Logic

Lanes are represented as bits in a 32-bit integer. This allows React to perform ultra-fast bitwise operations to check, batch, or exclude specific updates.

```typescript
// Simplified Lane definitions
const NoLanes = 0b0000000000000000000000000000000;
const SyncLane = 0b0000000000000000000000000000001;
const InputContinuousLane = 0b0000000000000000000000000000100;
const DefaultLane = 0b0000000000000000000000000010000;
const TransitionLanes = 0b0000000011111111111111111100000;
```

**Why Bitmasks?**

- **Batching:** `renderLanes | updateLane` instantly merges two updates.
- **Filtering:** `renderLanes & transitionLanes` checks if a render includes transitions.
- **Exclusion:** `renderLanes & ~SyncLane` allows React to work on background tasks while explicitly ignoring synchronous ones.

### Lane Entanglement

Lanes allow for **entanglement**, where two independent updates are forced to finish together. For example, if a high-priority "Focus" event occurs during a low-priority "Transition," React can entangle them to ensure the UI doesn't show a "partially focused" state while the transition is loading.

## Time Slicing and the Run Loop

The Scheduler yields to the browser every **5ms** to ensure the main thread remains responsive.

```javascript
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  advanceTimers(currentTime);
  currentTask = peek(taskQueue);

  while (currentTask !== null) {
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYieldToHost())
    ) {
      // Yield to the browser
      break;
    }
    // Perform Fiber work...
    const continuationCallback = callback(didUserCallbackTimeout);
    // ...
  }
}
```

### The Yielding Mechanism

React uses a `MessageChannel` (macrotask) to yield control back to the browser's event loop. This is more efficient than `setTimeout(0)` because it avoids the minimum 4ms delay in nested timers and executes immediately after the browser has had a chance to paint.

## Pragmatic Evaluation

> **The Philosophy:** Lanes decouple "what" is being updated from "when" it must finish, allowing React to be a multi-threaded engine on a single-threaded runtime.

### When to use this knowledge

- **Performance Debugging:** Use React DevTools "Timeline" to see if tasks are being interrupted or if "Lanes" are clashing.
- **Custom Hook Design:** If building complex state managers, understand that `useTransition` creates a "TransitionLane" that can be interrupted.

### When to avoid

- **Standard App Development:** Most developers should never interact with Lanes directly. Rely on high-level APIs like `useTransition` and `useDeferredValue`.

---

Related [Time Slicing](../browser-engine/time-slicing.md), [Concurrent Rendering](./concurrent-rendering.md), [Fiber Architecture](./fiber.md)
