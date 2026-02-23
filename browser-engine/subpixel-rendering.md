# Subpixel Rendering

Subpixel rendering is the "dark magic" that makes text look crisp on low-resolution displays and animations feel fluid. It works by exploiting the physical reality that every pixel on your screen is actually composed of three smaller lights: Red, Green, and Blue.

### Digital Deception

A standard pixel is a square, but edges in the real world are often curved or diagonal. Subpixel antialiasing allows the browser to turn on just _parts_ of a pixel (the Red and Green subpixels, for instance) to simulate an edge that falls "between" the pixel grid. This results in significantly sharper typography.

### The GPU Trade-off

Here's the catch: high-quality subpixel antialiasing is hard for GPUs. When an element is moved to its own **Compositing Layer** (to make it animate faster), the browser often switches from subpixel antialiasing to standard "grayscale" antialiasing.
**The Result:** You might notice text looking slightly "thinner" or "blurrier" as soon as it starts to animate. This is the browser choosing performance over perfect typography.

### Fractional Positioning

In animations, subpixel rendering allows for "Subpixel Positioning." If you move an element by `0.5px`, the GPU uses texture sampling to blend the colors across pixels. This is why a slow `transform: translateX` looks smooth, whereas a slow `left: 0.5px` (which forces a repainting of the pixel grid) often looks jittery and "stair-stepped."

To maintain the highest text quality, avoid unnecessary `will-change: transform` or `opacity` changes on text-heavy elements unless they are actually animating.

---

Related [Browser Compositing Layers](./compositing-layers.md), [GPU Acceleration in CSS](./gpu-acceleration.md)
