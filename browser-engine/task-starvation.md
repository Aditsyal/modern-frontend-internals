# Task Starvation

Task Starvation is a state where the browser's JavaScript Event Loop gets stuck in a "Microtask Black Hole." It's a deceptive performance killer because the CPU might not even be at 100%, yet the UI is completely frozen and unresponsive to user input.

## The Microtask Trap

To understand starvation, you have to understand the Event Loop's obsessive-compulsive nature. After every Macrotask (like a timer), the browser _must_ empty the entire Microtask queue before it can move on to rendering or the next Macrotask.

If a Microtask adds _another_ Microtask, the browser stays in that loop indefinitely.

```js
// The Infinite Starvation Loop
function starve() {
  Promise.resolve().then(starve); // Adds another microtask immediately
}
```

In this scenario, the browser never reaches the "Render" phase. The screen stops updating, and clicks are ignored, because the browser is too busy clearing an ever-growing pile of Promises.

### Real-World Starvation

This often happens in complex applications with recursive state updates or massive Promise chains. If you're processing a 10,000-item dataset using `.then()` chains, you might be starving the main thread of the time it needs to paint the next frame.

### The Solution: Yielding

The cure for starvation is **Yielding to the Main Thread**. By turning a Microtask into a Macrotask (using `setTimeout(fn, 0)` or the modern `scheduler.yield()`), you force the Event Loop to finish its current cycle. This gives the browser a chance to handle user input and render a frame before returning to your work.

Don't let your code become a Microtask bully. If you have work that takes more than 16ms, yield control back to the browser.

## Pragmatic Evaluation

> **The Philosophy:** Yielding trades raw throughput for responsiveness. A slightly slower operation that keeps the UI interactive is better than a fast one that freezes everything.

### When to use this

- Processing large datasets or arrays with chained Promises
- Complex state updates that trigger cascading re-renders
- Any async operation that could recursively enqueue more work

### When to avoid

- Simple one-shot operations that complete in under 16ms
- Critical path rendering where yield delays would hurt perceived performance

---

Related [Event Loop](./event-loop.md), [Long Tasks API](./long-tasks.md), [Time Slicing](./time-slicing.md)
