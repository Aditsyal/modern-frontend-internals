# Interaction to Next Paint (INP)

INP is the ultimate "Lag Meter". It measures the gap between a user's action (click, tap, keypress) and the moment they see a visual response on the screen. If LCP is about how fast the site arrives, INP is about how well it works once it's there.

## The Three Phases of Frustration

An INP score of 400ms is broken down into three distinct delays:

1. **Input Delay:** "The browser is busy", The main thread is stuck finishing a Long Task (usually JS execution) and hasn't even noticed your click yet.
2. **Processing Time:** "The code is running", The time your event handlers (`onClick`, etc.) take to execute.
3. **Presentation Delay:** "The browser is drawing", The time it takes to recalculate the layout and paint the new frame after your code finished.

## Why FID Was a Vanity Metric

First Input Delay (FID) only measured the **Input Delay** of the **very first** interaction. It ignored the processing time and completely ignored everything that happened after the first 5 seconds of the page load.
INP is much harder to pass because it looks at the **worst** (or 99th percentile) interaction across the **entire** session. If your app gets slow after 10 minutes of use, INP will catch it.

## The Fix: Yielding is Non-Negotiable

You cannot block the main thread for 200ms and expect a "Good" INP.

- **The "State-First" Pattern:** In your event handler, update the UI immediately (e.g., show a spinner) and then defer the heavy processing.
- **`scheduler.yield()`:** This is the modern, browser-native way to break up long tasks. Unlike `setTimeout(fn, 0)`, which goes to the back of the task queue, `scheduler.yield()` allows the browser to interleave high-priority work (like input handling) while continuing the current task as soon as possible.

```js
// The State-First yielding pattern
async function handleHeavyAction() {
  showSpinner(); // Visual feedback first

  await scheduler.yield(); // Allow input/paint

  // Continue heavy work
  performComplexCalculation();
  hideSpinner();
}
```

- **Concurrent React:** Use `useTransition`. It allows React to work on a heavy re-render in the background while keeping the main thread "interruptible" so it can still respond to new clicks.

## Pragmatic Evaluation

> **The Philosophy:** Interaction is the ultimate test of an application's quality. A site that ignores its users is dead.

### When to use this

- Applications with heavy client-side processing (e.g., data tables, editors).
- Mobile-first web apps where CPU power is limited.

### When to avoid

- Purely static "read-only" sites with no interactive elements.
- WebGL or Canvas-based games where the "Interaction" is handled via a custom engine outside the standard DOM event loop.

---

Related [Web Vitals Overview](./web-vitals.md), [Event Loop](./event-loop.md), [Time Slicing](./time-slicing.md)
