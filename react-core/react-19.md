# React 19: The New Standards

React 19 marks a fundamental shift from "Manual UI Management" to "Framework-Managed Intent." By automating the lifecycle of asynchronous operations, React 19 reduces boilerplate and ensures that the UI remains responsive by default.

## 1. Actions and Managed Transitions

The "Action" is the primary primitive for mutations in React 19. It leverages **Transitions** to handle the pending state, error boundaries, and optimistic updates automatically.

### useActionState: The Async Successor

`useActionState` (formerly `useFormState`) encapsulates the pattern of calling an async function and tracking its pending status.

```ts
const [state, submitAction, isPending] = useActionState(
  async (prevState, formData) => {
    const result = await updateItem(formData.get("id"), formData);
    return result; // Becomes the new 'state'
  },
  initialState,
);
```

**Internal Coordination:** When `submitAction` is called, React wraps it in a transition. This means the UI doesn't freeze while waiting for the promise; it remains interactive, and any state changes returned by the action are applied atomically.

### useOptimistic: Immediate Feedback

`useOptimistic` allows you to render a "provisionary" state while an action is in flight.

```ts
const [optimisticState, setOptimisticState] = useOptimistic(
  currentState,
  (state, newValue) => ({ ...state, ...newValue, isOptimistic: true }),
);

// usage in a transition
startTransition(async () => {
  setOptimisticState({ name: "New Name" });
  await updateNameAction("New Name");
});
```

React automatically reverts the `optimisticState` once the transition (or action) completes, whether it succeeds or fails.

## 2. The `use` API: Conditional Resource Consumption

The `use` primitive is a unique addition that allows reading resources (Promises or Context) directly in the render path.

- **Non-Hook constraints:** Unlike standard hooks, `use` can be called inside loops and `if` statements.
- **Suspense Integration:** If a promise is passed to `use(promise)`, React suspends the component until it resolves, delegating the loading state to the nearest `<Suspense>` boundary.

```ts
function Item({ detailsPromise }) {
  // Suspend until details arrive
  const details = use(detailsPromise);
  return <div>{details.name}</div>;
}
```

## 3. Native Platform Integration

React 19 removes the friction between the framework and the underlying document structure.

### Metadata Hoisting

React now natively understands `<title>`, `<meta>`, and `<link>` tags anywhere in the tree. It automatically "hoists" them to the `<head>`, ensuring SEO and social sharing tags work across SSR and Client environments without external libraries.

### Resource Loading APIs

New `react-dom` APIs provide explicit control over browser prioritization:

- `preload(href, options)`: Hints the browser to download a resource early.
- `preinit(href, options)`: Fetches and evaluates a script or stylesheet immediately.
- `prefetchDNS(href)` and `preconnect(href)`: Optimizes connection overhead for external domains.

## 4. Simplified Ref Handling

`forwardRef` is deprecated. Refs are now standard props, simplifying component composition.

```ts
function MyInput({ ref, label, ...props }) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
}
```

## 5. Server Actions: The RPC Bridge

By using the `"use server"` directive, functions become secure RPC (Remote Procedure Call) endpoints. React handles the underlying network transit and serialization.

- **Composability:** Server Actions can be passed as props to Client Components.
- **Progressive Enhancement:** In form contexts, Server Actions can function even before JavaScript has hydrated the page.

---

## Pragmatic Evaluation

> **The Philosophy:** Moving from imperative state management to declarative intent. The framework takes responsibility for the "Uncanny Valley" (the time between user action and server response).

### When to use this

- **Data-Heavy Applications:** Where managing loading/error states for dozens of mutations becomes a maintenance burden.
- **SEO-Critical Sites:** Utilizing native metadata hoisting and streaming SSR support.
- **Optimistic UIs:** When immediate feedback is required for a high-quality UX.

### When to avoid

- **Legacy Migration:** Wholesale conversion to Actions may not be necessary if existing state management (like TanStack Query) is already handling these concerns.
- **Simple Static Sites:** The overhead of the Action/Transition queue might be unnecessary for apps with minimal interactivity.

---

Related [Concurrent Rendering](../react-core/concurrent-rendering.md), [Suspense Boundaries](../react-core/suspense.md)
