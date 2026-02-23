# Long Tasks API

The browser's main thread is a single-lane highway. If a JavaScript function decides to park in the middle of that lane for 200ms, the entire UI comes to a grinding halt. The Long Tasks API is your radar system for catching these offenders.

## The 50ms Rule

The W3C defines a "Long Task" as any execution block that exceeds 50 milliseconds. Why 50ms? Because to maintain a responsive 60fps UI, the browser needs about 16ms per frame. If you take 50ms, you've already missed three frames. More importantly, it leaves enough "breathing room" to handle a user input (like a click) within the 100ms window that humans perceive as "instant."

## The Invisible Jank

When the main thread is occupied by a long task, the browser is effectively deaf. It can't respond to clicks, it can't run animations, and it can't scroll smoothly. This is the root cause of high **Interaction to Next Paint (INP)** scores. A user clicks a button, nothing happens for 300ms, and then suddenly the UI jumps -- that's the "jank" caused by a long task.

## Monitoring in the Wild

Don't rely on your high-end developer machine to catch these. Use a `PerformanceObserver` to track long tasks on real user devices:

```js
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    // entry.duration is the total time spent
    // entry.attribution can tell you which container/script is the culprit
    reportToAnalytics({
      name: "longtask",
      duration: entry.duration,
      startTime: entry.startTime,
    });
  });
});
observer.observe({ type: "longtask", buffered: true });
```

## Strategic Defragmentation

You can't always avoid heavy work, but you can avoid doing it all at once.

- **Time Slicing:** Break large tasks into smaller chunks using `scheduler.yield()` (or `setTimeout` as a fallback). This allows the browser to "interleave" user input between your work chunks.
- **Offloading:** If it doesn't touch the DOM, it doesn't belong on the main thread. Move heavy data processing, image manipulation, or complex calculations to a **Web Worker**.

Treat the main thread as a precious resource. Every millisecond you spend there is a millisecond you're stealing from your user's experience.

## Pragmatic Evaluation

> **The Philosophy:** Detection is the first step to optimization. You cannot fix what you do not measure.

### When to use this

- Production monitoring with Real User Monitoring (RUM) tools
- Performance regression testing in CI/CD pipelines
- Debugging high INP scores in Core Web Vitals audits

### When to avoid

- Development-only environments without real user data
- Over-optimization before establishing a performance baseline
- Low-traffic pages where monitoring overhead outweighs insights

---

Related [Time Slicing](./time-slicing.md), [Event Loop](./event-loop.md)
