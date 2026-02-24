# CSS Containment

The browser's layout engine is historically "global." If you change the height of a `span` in the footer, the browser theoretically has to check if that change affected the header. In a complex app, this "Global Recalculation" is a performance nightmare.

## Isolation as a Strategy

The `contain` property is a way to draw a boundary around a DOM subtree and tell the browser: "Nothing inside this box affects anything outside this box."

1. **`contain: layout`:** Changes inside this element won't trigger a re-layout of the parent. The element becomes a "Silo."
2. **`contain: paint`:** Ensures children never draw outside the parent's bounds. If the parent is off-screen, the browser skips painting the entire subtree.
3. **`contain: size`:** Tells the browser the parent's size doesn't depend on its children. This is the "ultimate" optimization, but it requires you to set a fixed `height` and `width`.
4. **`contain: inline-size`:** A more flexible variant that only ignores child size in the inline direction (usually width). This is useful for container queries where the width is fixed but the height is dynamic.

## Content-Visibility: Lazy Rendering for the DOM

`content-visibility: auto` is one of the most powerful CSS properties ever released. It applies `contain: content` automatically.

- If the element is off-screen, the browser simply **does not render it**. No layout, no paint.
- Unlike `display: none`, the content is still searchable and accessible.
- As the user scrolls, the browser renders the "cards" on the fly. This can turn a page with 10,000 complex items from a laggy mess into a buttery smooth experience.

## Intrinsic Size: The Scrollbar Fix

When using `content-visibility`, the browser doesn't know how tall the off-screen elements are, which can break the scrollbar. Use `contain-intrinsic-size` to provide a "placeholder" height. This keeps the scrollbar consistent and prevents "jumps" as new content is rendered.

## Pragmatic Evaluation

> **The Philosophy:** Trading global layout flexibility for predictable, localized rendering boundaries.

### When to use this

- **Complex UIs:** Applications with massive DOM trees or frequent updates in specific regions.
- **Infinite Scrolls:** Use `content-visibility` to keep memory footprint low while maintaining accessibility.
- **Third-party Widgets:** Isolate external content to prevent it from affecting your app's layout performance.

### When to avoid

- **Small Pages:** Over-engineering simple layouts adds unnecessary complexity.
- **Dynamic Sizing:** Avoid `contain: size` if elements must strictly respond to their dynamic content height.
- **Layout Jumps:** If `contain-intrinsic-size` isn't used correctly, it can lead to frustrating scrollbar instability.

---

Related [Critical Rendering Path](./critical-rendering-path.md), [Layout Thrashing](./layout-thrashing.md), [Browser Compositing Layers](./compositing-layers.md)
