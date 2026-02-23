# GPU Acceleration in CSS

Hardware acceleration isn't a "make fast" button; it's a redirection of labor. You are taking work off the scholarly, sequential **CPU** and handing it to the massive, parallelized army of the **GPU**.

## Parallel Power

The CPU calculates the position of an element by solving math equations one by one. The GPU calculates the position of an element by telling 1,000 "cores" to each handle a small patch of pixels simultaneously.
For operations like **translation (moving)**, **scaling (resizing)**, and **opacity (fading)**, the GPU is orders of magnitude more efficient.

## The `will-change` Warning

In the old days, we used `translateZ(0)` to "trick" the browser into using the GPU. Today, we have `will-change`.

- **The nuance:** `will-change` should be treated like a high-performance sports car -- don't leave it idling in the driveway.
- When you set `will-change: transform`, the browser allocates a dedicated buffer in VRAM immediately. If you put this on every element, you will exhaust the device's memory, leading to blurry text, flickering, or crashes.

## Sub-pixel Rendering

Because the GPU operates on a coordinate system that is independent of the DOM's layout, it can render elements at "half-pixel" positions. This is why a `transform: translateX` animation looks perfectly smooth, while a `left` animation looks "steppy." The GPU is interpolating the pixels to create the illusion of fluid motion.

---

Related [Browser Compositing Layers](./compositing-layers.md), [Critical Rendering Path](./critical-rendering-path.md)
