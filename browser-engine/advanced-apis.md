# Advanced Performance APIs

Stop guessing why your application feels sluggish. Modern browsers provide low-level instrumentation that moves performance tuning from "vibe-based" fixes to data-driven engineering.

## The Observer Pattern: Stop Polling

Older methods like `performance.getEntries()` are expensive because they force you to poll the browser for data, often missing transient events. The `PerformanceObserver` API flips the script by using a push-based model. It only wakes up your code when a specific event occurs, keeping the main thread clear.

```javascript
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    // Precise timing for layout shifts, LCP elements, or long tasks
    console.log(`${entry.name}: ${entry.startTime}ms`);
  }
});

// "buffered: true" captures events that happened before the observer started
observer.observe({ type: "longtask", buffered: true });
```

## Long Tasks: The 50ms Boundary

The browser tries to maintain 60 frames per second, which gives you roughly 16.6ms per frame. However, for responsiveness, the "Long Task" threshold is 50ms. Anything exceeding this blocks the main thread long enough for a user to perceive lag.

If your observer reports long tasks, you have two choices:

1. **Offload:** Move the logic to a Web Worker.
2. **Fragment:** Use `scheduler.yield()` (or `setTimeout(0)` as a fallback) to break the task into chunks, allowing the browser to breathe (and paint) between segments.

```javascript
async function heavyWork() {
  for (const item of bigData) {
    process(item);
    // Yield control back to the browser every 50ms
    if (performance.now() - lastYield > 50) {
      await scheduler.yield();
      lastYield = performance.now();
    }
  }
}
```

## High-Precision Observers

`IntersectionObserver` and `ResizeObserver` aren't just for lazy loading; they are integrated directly into the browser's layout cycle.

- **IntersectionObserver:** Use this to pause heavy animations or video playbacks the moment they leave the viewport.
- **ResizeObserver:** Avoid the "infinite loop" trap. If a callback modifies the dimensions of the element it's watching, the browser throws an error. This is a safety rail for the layout engine, preventing cyclic reflows that would otherwise crash the tab.

## The Memory Shadow: Garbage Collection

You cannot force the browser to run Garbage Collection (GC), but you can force it to work harder. In high-frequency loops -- like scroll listeners or `requestAnimationFrame` -- creating objects (`{}`) or arrays (`[]`) triggers "GC Pressure."

Modern engines use generational GC, which is fast for short-lived objects but eventually hits a "Stop-the-World" pause to clean up the heap. To avoid "jank" (stuttering), reuse objects or use TypedArrays when processing large data sets. If the Memory tab shows a "sawtooth" pattern, you are creating too much temporary garbage.

## Pragmatic Evaluation

> **The Philosophy:** Moving from reactive debugging to proactive observability using native browser telemetry.

### When to use this

- **SLA Monitoring:** Use `PerformanceObserver` to track field data (RUM) for Core Web Vitals.
- **Computational Heavy Tasks:** Use `scheduler.yield()` when you must perform heavy JS work on the main thread without freezing the UI.
- **Resource Optimization:** Use `IntersectionObserver` to aggressively prune off-screen resources.

### When to avoid

- **Premature Optimization:** Don't add observers for trivial tasks where the overhead of the observer itself might be larger than the benefit.
- **Synchronous Dependencies:** If a task _must_ complete before the next user interaction, fragmentation via `yield` might introduce unwanted latency.

---

Related [Long Tasks API](./long-tasks.md), [Memory Leak Detection](./memory-management.md)
