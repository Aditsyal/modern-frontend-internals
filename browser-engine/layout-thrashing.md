# Layout Thrashing

Layout Thrashing is the silent killer of 60fps performance. It occurs when you force the browser to calculate the geometry of elements multiple times in a single frame, usually because your code is playing "ping-pong" between reading and writing DOM properties.

### The Mechanism of Failure

The browser is fundamentally lazy -- and that's a good thing. When you modify a DOM element's style, the browser doesn't immediately recalculate the entire page's layout. It marks the layout as "dirty" and schedules a reflow for the end of the current task.

Thrashing happens when you interrupt this lazy cycle. If you write a style change (dirtying the layout) and then immediately read a layout-dependent property (like `offsetHeight`), you force a **Synchronous Reflow**. The browser has no choice but to stop everything, calculate the new layout to give you an accurate number, and then continue. If you do this inside a loop, you've created a "Layout Junkie" that destroys your frame budget.

### The Sequence of Doom

```javascript
// BAD: This is a performance catastrophe
for (let i = 0; i < items.length; i++) {
  const width = items[i].offsetWidth; // READ (Forces Reflow)
  items[i].style.height = width + "px"; // WRITE (Dirties Layout)
}
```

In this loop, every single iteration triggers a full layout calculation. On a page with hundreds of elements, this can turn a 16ms frame into a 200ms nightmare.

### Engineering Around Thrashing

The solution is simple in theory but requires discipline in practice: **Batch your work.** Think of the browser as a delivery driver. You don't send them out for one package, wait for them to return, and then send them out for the next. You give them the whole list at once.

1. **Read everything first:** Gather all the metrics you need from the DOM.
2. **Write everything second:** Apply all your style changes in one go.

```javascript
// GOOD: Batched for efficiency
const widths = items.map((item) => item.offsetWidth); // One layout pass (if any)

items.forEach((item, i) => {
  item.style.height = `${widths[i]}px`; // Batch writes at the end
});
```

### The Invisible Triggers

Many developers don't realize how many properties trigger this behavior. Beyond the obvious `offsetWidth` and `offsetHeight`, calling `getComputedStyle()`, accessing `scrollTop`, or even checking `offsetTop` will force a reflow if the layout is dirty.

For complex animations, use `requestAnimationFrame`. It ensures your writes happen at the start of the next frame, giving the browser the best chance to optimize the rendering pipeline. Use tools like the **Performance** tab in DevTools to hunt for those telling purple bars labeled "Forced Reflow" -- that's where your frames are going to die.

---

Related [Critical Rendering Path](./critical-rendering-path.md), [Browser Compositing Layers](./compositing-layers.md)
