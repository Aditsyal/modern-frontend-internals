# Detached DOM Nodes

A detached DOM node is a "Zombie." It's been removed from the visible world (the document), but it still haunts the memory because a piece of JavaScript is still holding its hand.

## How Zombies are Created

JavaScript uses "Reachability" to decide what to delete.

1. You create an element: `const btn = document.createElement('button')`.
2. You append it: `document.body.appendChild(btn)`.
3. You remove it: `btn.remove()`.
4. **The Leak:** Because the variable `btn` still exists in your code, the browser cannot reclaim the memory. If that button was the parent of a complex 1,000-row table, that entire table is now a "Detached" leak.

## The "Retainer" Path

Finding leaks requires looking at the **Memory Heap Snapshot** in DevTools.

- Filter by "Detached".
- Look at the **Retainers** section. It will show you the chain of variables keeping the node alive. Often, it's a global array like `window.activePopups` or a closure inside a `setInterval` that was never cleared.

## Prevention: The Cleanup Ritual

- **Nullification:** The moment you `remove()` an element, set the variable to `null`.
- **WeakMaps:** If you need to store data about an element, use a `WeakMap`. Unlike a standard `Map`, a `WeakMap` does not prevent the browser from garbage-collecting the element if it's no longer in the DOM.
- **Event Listeners:** Always pair `addEventListener` with `removeEventListener` (or use the `once: true` option). If the listener's callback references the element, it can create a cycle that the GC can't break.

### WeakMaps for Metadata

If you need to attach metadata to an element without leaking it, use a `WeakMap`.

```javascript
const elementMetadata = new WeakMap();
const btn = document.querySelector("#action-btn");

// Storing state associated with the element
elementMetadata.set(btn, { clickedCount: 0 });

// When btn is removed and unreferenced, its entry in elementMetadata is automatically eligible for GC
```

## Pragmatic Evaluation

> **The Philosophy:** Understanding that DOM reachability in JS is the number one cause of web memory bloat.

### When to use this

- **Dynamic UI Frameworks:** When building or using custom rendering logic that manually manipulates the DOM.
- **Complex Tables/Lists:** When removing large chunks of content from the view.

### When to avoid

- **Declarative Frameworks:** Modern frameworks (React, Vue) usually handle DOM cleanup automatically, though leaks can still occur in custom hooks or refs.

---

Related [Browser Memory Leak Detection](./memory-management.md), [Layout Thrashing](./layout-thrashing.md), [Browser Compositing Layers](./compositing-layers.md)
