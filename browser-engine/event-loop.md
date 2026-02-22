# Event Loop: Macrotasks vs. Microtasks

The JavaScript Event Loop is a single-threaded juggler. It manages a mountain of work by switching between tasks so fast that it feels simultaneous. But there is a strict hierarchy to how it chooses the next "ball" to catch.

## The Hierarchy of Tasks

1. **Synchronous Code:** The "Now" This always runs until completion.
2. **Microtasks:** The "Immediate Future" This includes `Promises` and `MutationObservers`.
3. **Macrotasks:** The "Scheduled Future" This includes `setTimeout`, `I/O`, and `UI Events`.

## The "Greedy" Microtask Queue

The most important rule: **The Event Loop will not move to a Macrotask or a Repaint until the Microtask queue is empty.**
If a Microtask adds another Microtask, the loop stays in the Microtask phase. This is how you "starve" the browser. If you have an infinite loop of Promises, the browser will never paint, never process a click, and never fire a `setTimeout`. The UI will simply freeze.

## Macrotasks: The Yielding Citizens

Unlike Microtasks, the loop only processes **one** Macrotask per cycle. After one `setTimeout` fires, the browser checks if it needs to paint the screen, then checks for new Microtasks, and only then moves to the next `setTimeout` in the queue.

## Practical Optimization: The "Gap"

- **Batching:** Use Microtasks (Promises) when you want to batch multiple state updates into a single render. This is how Vue and React handle "Next Tick" logic.
- **Responsiveness:** Use Macrotasks (`setTimeout` or `scheduler.yield`) when you have a heavy calculation. By yielding, you give the browser a "gap" to handle user input (like a mouse click) between chunks of work.

## Pragmatic Evaluation

> **The Philosophy:** Batching improves throughput, yielding improves responsiveness. The trade-off is between raw execution speed and user-perceived performance.

### When to use this

- Use Microtasks for state batching and avoiding unnecessary re-renders in frameworks
- Use Macrotasks when processing large datasets or heavy computations that exceed 16ms

### When to avoid

- Avoid Microtask recursion in hot paths without a yield mechanism
- Avoid excessive `setTimeout(fn, 0)` for trivial operations - it adds unnecessary scheduling overhead

---

_Deepen your knowledge:_ [Task Starvation](./task-starvation.md), [Long Tasks](./long-tasks.md)
