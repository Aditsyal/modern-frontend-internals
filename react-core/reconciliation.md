# Reconciliation: The Diffing Engine

Reconciliation is the process by which React calculates the difference between the current UI and the desired state. While a full tree comparison is an O(n^3) problem, React uses a heuristic algorithm to achieve **O(n)** linear complexity.

## The Heuristic Assumptions

React's diffing strategy relies on two "Expert Assumptions":

1.  **Type Stability:** Two elements of different types (e.g., changing a `<div>` to a `<span>`) will produce entirely different trees. React will destroy the old subtree and mount the new one without attempting to find matches deep in the hierarchy.
2.  **Key Stability:** Developers can provide a `key` prop to indicate that an element is stable across renders, even if its position in a list changes.

## The Diffing Algorithm

### 1. Same Type, Different Props

If the element type remains the same, React keeps the underlying DOM node (or component instance) and only updates the attributes that changed (e.g., `style`, `className`, or `id`).

### 2. List Diffing (The Two-Pass Strategy)

When diffing children lists, React uses a two-pass approach to handle common scenarios efficiently.

#### Pass 1: Direct Positional Diff

React iterates through both the old and new children lists simultaneously. If the `key` and `type` match at the same index, it updates the existing fiber. The moment it hits a mismatch (a deletion or insertion), it stops this pass.

#### Pass 2: The Map Strategy

If nodes remain after Pass 1, React creates a `Map` of the remaining old children, keyed by their `key`.

- It then iterates through the remaining new children.
- If a child exists in the map, it is "moved" to the new position.
- If not, it is created.
- Any nodes left in the map after this pass are marked for deletion.

**Visual Mapping:**

- Old: `Key 1` to `Key 2` to `Key 3`
- New: `Key 2` to `Key 1` to `Key 3`
  React uses the map to identify that `Key 1` and `Key 2` simply swapped positions, rather than being deleted and recreated.

## The "Key" Requirement

Without keys, React's diffing is purely positional. Inserting an item at the start of an array causes every subsequent item to be re-rendered because their index-to-element mapping shifted. **Keys provide identity** that persists across renders.

## Fiber Integration: Effect Tags

In the Fiber Reconciler, reconciliation doesn't just calculate differences; it assigns **Effect Tags** (Placement, Update, Deletion) to Fiber nodes. These tags form the instructions for the Commit phase.

## Pragmatic Evaluation

> **The Philosophy:** Optimizing for the 99% case (stable types and keys) while accepting a full rebuild for the 1% (radical structural changes).

### When to use this

- **Dynamic Lists:** Essential for performance when adding, removing, or reordering items in large collections.
- **Stable Hierarchies:** Works best when the component structure mirrors the visual structure.

### When to avoid

- **Random Keys:** Never use `Math.random()` or array indices as keys for dynamic lists. This forces a full rebuild on every render, defeating the reconciler.
- **Extreme Depth:** While O(n), extremely deep trees still incur a "traversal tax". Prefer flatter structures where possible.

---

Related [Fiber Architecture](../react-core/fiber.md)
