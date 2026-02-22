# Time Slicing

Time Slicing is the technical foundation of "Concurrent" UI frameworks. It solves the "All-or-Nothing" problem of traditional rendering, where the browser is forced to finish drawing an entire component tree before it can do anything else.

## The Synchronous Ceiling

In a standard React or Vue app, once a render starts, the main thread is occupied until it's finished. If you're rendering a list of 5,000 complex items, and the user tries to type in a search box, the browser can't process that keystroke until the list is done. This creates a "heavy" feeling that frustrates users.

## The Interruptible Engine

Time Slicing turns the rendering process into an interruptible stream of small "slices."

1. **Work:** The framework renders a small chunk of the UI (e.g., 5ms of work).
2. **Check:** It asks the browser, "Is there any urgent mail?" (clicks, keypresses).
3. **Yield:** If an event is pending, it pauses the render, saves its spot, and lets the browser handle the event.
4. **Resume:** Once the urgent work is done, it picks up exactly where it left off.

## The Fiber Analogy

Think of traditional rendering as a single long scroll of paper. If you want to change something, you have to rewrite the whole thing. Time Slicing (and React's Fiber architecture) treats the UI as a stack of index cards. You can stop after any card, go do something else, and come back to the stack later.

This directly improves **Interaction to Next Paint (INP)**. By ensuring the main thread is never blocked for more than a few milliseconds, the UI remains responsive even during massive data updates.

## Pragmatic Evaluation

> **The Philosophy:** Interruptibility adds complexity but buys user trust. A UI that responds feels fast, even if the total work takes longer.

### When to use this

- Rendering large lists, tables, or trees in client-side frameworks
- Complex UI updates that could take 50ms+ to complete
- Applications where INP is a critical metric (e-commerce, dashboards)

### When to avoid

- Simple apps with minimal interactive elements
- Static content sites where rendering is a one-time cost
- Projects without framework support for concurrent rendering

---

_Deepen your knowledge:_ [Event Loop](./event-loop.md)
