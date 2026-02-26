# Paint vs. Composite vs. Layout

Not all CSS properties are created equal. To the browser, changing a `width` is a massive architectural project, while changing a `transform` is just moving a picture across a wall. Understanding this hierarchy is the difference between a stuttering UI and a buttery-smooth experience.

### 1. Layout (The Heavy Lifter)

When you change properties like `width`, `margin`, or `flex`, the browser has to calculate the geometry of the entire page. It's an expensive CPU operation because changing one element often ripples through the entire document (Reflow).
**Verdict:** Avoid animating these at all costs.

### 2. Paint (The Artist)

Once the browser knows where things are, it has to draw them. This involves filling in pixels for text, colors, shadows, and borders. While cheaper than Layout, it still requires the CPU to generate bitmaps and upload them to the GPU.
**Verdict:** Better than layout, but still adds significant overhead to every frame.

### 3. Composite (The Specialist)

This is the "cheat code" of web performance. The browser takes pre-painted "layers" and hands them to the GPU. The GPU is incredibly fast at moving, scaling, and fading these layers.
**Verdict:** This is where you want to live.

### The Waterfall Effect

The browser rendering pipeline is a one-way street: **Layout to Paint to Composite**.

- If you trigger **Layout**, you force the browser to **Paint** and **Composite** again.
- If you trigger **Paint**, you force a **Composite**.
- If you trigger **Composite** (using `transform` or `opacity`), you skip the first two stages entirely.

### High-Performance Animation Strategy

If you want 60fps, your animations must be "Composite-only." Instead of animating `top: 0` to `top: 100px` (which triggers Layout), use `transform: translateY(100px)`. To the user, it looks the same; to the browser, it's the difference between recalculating a city's traffic flow and just sliding a map across a table.

### The Hidden Cost: Runtime CSS-in-JS

Modern performance engineering favors **Zero-Runtime CSS** (Tailwind, StyleX, CSS Modules). Traditional runtime CSS-in-JS libraries (like older versions of styled-components or Emotion) must generate and inject `<style>` tags into the DOM _during_ execution. This happens on the main thread, often during critical rendering windows or animations, leading to "style calculation" spikes that can cause frame drops and hurt INP.

---

Related [Critical Rendering Path](./critical-rendering-path.md), [Browser Compositing Layers](./compositing-layers.md), [Layout Thrashing](./layout-thrashing.md)
