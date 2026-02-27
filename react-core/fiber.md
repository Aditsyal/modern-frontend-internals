# Fiber Architecture: The Engine of Concurrency

Fiber is the internal engine of React, introduced in version 16 to replace the old "Stack Reconciler." While the old reconciler used the JavaScript call stack (making it synchronous and recursive), Fiber uses a virtual stack implemented as a **linked-list of work units**.

## The Fiber Object

Each React element has a corresponding "Fiber" object. This object acts as a unit of work and maintains:

- **`stateNode`:** Reference to the actual component instance or DOM node.
- **`child`, `sibling`, `return`:** Pointers that form the tree structure as a linked list.
- **`alternate`:** A reference to the "other" fiber (Double Buffering).
- **`lanes`:** Bitmasks representing the priority of the work (Concurrent Features).
- **`memoizedState`:** A linked list of hooks (`useState`, `useEffect`).

### The Linked List Structure

Unlike traditional trees with `children[]` arrays, Fiber nodes point to their first child, their next sibling, and their parent (return).

```
  Parent (return)
    |
  Child 1 (child)  to  Child 2 (sibling)  to  Child 3
```

## The Two-Phase Model: Render & Commit

Fiber splits the rendering process into two distinct phases to enable interruptibility and priority-based scheduling.

### 1. The Render Phase (Asynchronous)

React traverses the Fiber tree and determines what needs to change.

- **Double Buffering:** React maintains two trees: `current` (the one currently on screen) and `workInProgress`. During an update, React clones the `current` fiber into a `workInProgress` fiber using the `alternate` pointer.
- **Interruptible Work Loop:**
  ```js
  function workLoopConcurrent() {
    // Perform work until the tree is complete OR the browser needs to paint
    while (workInProgress !== null && !shouldYield()) {
      performUnitOfWork(workInProgress);
    }
  }
  ```
- **Pure:** This phase has no observable side effects. It can be thrown away and restarted if a higher-priority update comes in.

### 2. The Commit Phase (Synchronous)

Once the `workInProgress` tree is complete, React swaps the pointers (making `workInProgress` the new `current`) and applies changes to the DOM.

- **Atomic:** To prevent "UI tearing" (inconsistent visual state), this phase is synchronous and uninterruptible.
- **Side Effects:** DOM mutations, `useLayoutEffect`, and scheduling `useEffect` occur here.

## The Linked List Advantage: Time Slicing

Because Fiber uses pointers instead of a recursive stack, React can stop mid-traversal, save the `workInProgress` pointer, and resume later. This is the foundation of **Time Slicing**, allowing React to keep the main thread responsive even during heavy rendering.

## Pragmatic Evaluation

> **The Philosophy:** Trading memory (cloning trees) and architectural complexity for responsiveness and scheduling control.

### When to use this

- **High-Interactivity Apps:** When UI responsiveness (e.g., typing in an input) must not be blocked by heavy background rendering.
- **Complex UI Updates:** When you need to prioritize critical updates (animations) over non-critical ones (data fetching).

### When to avoid

- **Micro-benchmarks:** Fiber adds significant object allocation overhead. In extremely simple apps, a direct-to-DOM or build-time approach is objectively faster.
- **Memory-Constrained Environments:** Maintaining two full fiber trees increases memory pressure compared to simple template-based systems.

---

Related [Reconciliation Algorithm](../react-core/reconciliation.md)
