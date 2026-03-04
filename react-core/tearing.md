# Tearing: The Consistency Challenge

Tearing is a visual bug unique to Concurrent Rendering. It occurs when different parts of the UI display different values for the same piece of external state during a single "render transaction".

## Why Tearing Happens

In synchronous rendering, the main thread is blocked. Nothing can change the state until the render finishes. In concurrent rendering, React **yields**.

1.  React renders Component A with `count = 1`.
2.  React yields to the browser.
3.  A WebSocket event updates the global store to `count = 2`.
4.  React resumes and renders Component B with `count = 2`.
5.  **The Tear:** Component A shows "1", Component B shows "2". The UI is inconsistent.

## The `useSyncExternalStore` Fix

To prevent tearing, external state managers (Redux, Zustand) must use the `useSyncExternalStore` hook. This hook gives React a way to:

- **Subscribe:** Listen for changes in the store.
- **Get Snapshot:** Read the current value.

If a change occurs in the store while React is in the middle of a concurrent render, React will detect the mismatch via the subscription. It will then **discard** the inconsistent Work-In-Progress tree and restart the render from the top, ensuring every component sees the same "snapshot" of state.

### How it Works Internally

When a store update occurs, `useSyncExternalStore` triggers a **synchronous re-render** for those specific components. This effectively "opts out" of the concurrent model for that specific slice of state, ensuring visual consistency across all components that depend on that store.

## Rule of Thumb for Developers

If your state lives inside React (`useState`, `useReducer`), React handles consistency for you. If your state lives **outside** React (global variables, browser APIs, custom stores), you **must** use `useSyncExternalStore` to participate in the concurrent model safely.

## Pragmatic Evaluation

> **The Philosophy:** External state is a "black box" to React. By using `useSyncExternalStore`, you provide React with the necessary hooks to treat that external state as a reliable, consistent source of truth.

### When to use this

- **Global State Managers:** Redux, Zustand, and Recoil already use this under the hood.
- **Browser APIs:** If you're subscribing to `window.innerWidth`, `navigator.onLine`, or `matchMedia`, use `useSyncExternalStore` for a flicker-free experience.

### When to avoid

- **Internal React State:** Stick to `useState` for most components. `useSyncExternalStore` adds overhead that is unnecessary for simple, local state.
- **Mutable Global Variables:** Avoid global variables altogether. If you must use them, you MUST wrap them in a store that provides a subscription mechanism.

---

Related [Concurrent Rendering](./concurrent-rendering.md), [Scheduler Priorities](./scheduler.md)
