# Browser Compositing Layers

The browser's rendering pipeline is a factory line: **JavaScript to Style to Layout to Paint to Composite**. If you want 60 FPS animations, you must bypass the first four stages and go straight to the Compositor.

## The Photoshop Analogy

Think of the page as a stack of transparent films. By default, everything is drawn on one layer. If you move a single element, the browser has to "repaint" the entire film. By promoting an element to its own **Compositing Layer**, you give it its own dedicated piece of memory. Moving it then becomes a simple matter of the GPU sliding one film over the others -- zero repainting required.

## The GPU: Your Parallel Painter

The CPU is a brilliant generalist, but it's slow at painting pixels. The GPU is a specialized beast designed to process thousands of mathematical operations in parallel. When an element is on its own layer, the browser uploads it to the GPU as a "texture."

- **`transform` and `opacity`** are the only properties handled entirely by the Compositor.
- Animating `margin` or `top` forces the CPU to recalculate the **Layout**, which is the slowest operation in the pipeline.

## Layer Promotion Criteria

The browser doesn't promote everything to a layer because layers cost **VRAM (Video RAM)**. It only does so when:

1. It detects 3D transforms (e.g., `translate3d`).
2. You explicitly use `will-change: transform`.
3. Elements like `<video>`, `<canvas>`, or `<iframe>` are present.
4. Overlap occurs: A high z-index element sitting on top of an accelerated layer can force everything else into its own "squashed" layer.

## The VRAM Tax: Layer Explosion

Promotion is a double-edged sword. Every layer requires memory proportional to its width and height (not just its file size).

- **The Trap:** Using `* { will-change: transform; }` is a guaranteed way to crash a mobile browser. You'll run out of VRAM, and the browser will either revert to slow CPU rendering (causing massive lag) or simply kill the tab.
- **The Fix:** Use the "Layers" panel in DevTools. If you see thousands of layers, you are over-optimizing. Aim for a few key layers for heavy animations.

---

Related [Critical Rendering Path](./critical-rendering-path.md), [Layout Thrashing](./layout-thrashing.md).
