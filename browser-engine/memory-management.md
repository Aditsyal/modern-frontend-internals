# Memory Leak Detection

A memory leak is a slow-motion car crash. It doesn't kill your app immediately; instead, it slowly drains the system's resources until the UI stutters, the browser tab grows bloated, and eventually, the whole thing collapses.

### The Anatomy of a Leak

In JavaScript, memory management is handled by a Garbage Collector (GC). The GC is smart, but it's not a mind reader. If you leave a reference to an object, the GC assumes you still need it. A leak happens when you "forget" to clean up these references.

#### The Three Horsemen of Memory Leaks:

1. **Detached DOM Nodes:** You remove an element from the page (`document.body.removeChild(el)`), but you still have a reference to it in a JavaScript variable. The element is gone from the screen, but its entire internal tree stays in memory.
2. **Zombie Event Listeners:** Attaching a listener to `window` or `document` inside a component and failing to remove it on unmount. The listener keeps the component's scope (and all its variables) alive forever.
3. **Forgotten Timers:** A `setInterval` that keeps ticking in the background, holding onto every variable in its closure.

### The FinalizationRegistry: Advanced Tracking

While you can't force GC, the `FinalizationRegistry` lets you register a callback that runs after an object is garbage-collected. This is invaluable for debugging internal state leaks.

```javascript
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Object "${heldValue}" was reclaimed.`);
});

registry.register(someObject, "Metadata Record");
```

### The "Three Snapshot" Diagnosis

To catch a leak, you need to think like a detective. Use the **Memory** tab in Chrome DevTools and follow the "Three Snapshot" rule:

1. **Baseline:** Take a snapshot when the app is idle.
2. **Stress:** Perform a suspicious action (like opening a heavy modal) and then close it. Repeat this 5-10 times.
3. **Validation:** Take a second snapshot.
4. **Comparison:** Look for objects that were created but never destroyed. In the "Comparison" view, sort by **Retained Size**. This shows you the total memory that would be freed if that specific object was deleted.

### Engineering for Purity

Don't just fix leaks -- architect against them.

- Use **WeakMap** and **WeakSet** for metadata. They don't prevent the GC from reclaiming objects.
- Always return a cleanup function in your `useEffect` hooks.
- Avoid global variables like the plague. If it's on `window`, it's permanent until the page is refreshed.

## Pragmatic Evaluation

> **The Philosophy:** Accepting that the Garbage Collector is a partner, not a servant. Engineering for reachability.

### When to use this

- **Long-lived Applications:** Dashboards or SPAs that users keep open for days without refreshing.
- **Heavy Data Processing:** Applications handling large JSON blobs, images, or real-time streams.
- **Component Libraries:** Ensuring shared components don't pollute the consumer's memory heap.

### When to avoid

- **Short Sessions:** Static pages or sites where the user stays for less than a minute.
- **Script Overhead:** Don't over-engineer `WeakMap` usage for trivial, short-lived variables.

---

Related [Detached DOM Nodes](./detached-dom-nodes.md), [Garbage Collection Timing](./advanced-apis.md), [Event Loop Performance](./event-loop.md)
