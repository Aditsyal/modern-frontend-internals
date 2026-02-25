# Cumulative Layout Shift (CLS)

CLS is the "Bait-and-Switch" metric. It measures how much the page's content moves around while the user is trying to read or interact with it. A high CLS isn't just a performance issue; it's a usability failure that leads to accidental clicks and user frustration.

## The Math of Instability

Google doesn't just count shifts; it weighs them.
**Layout Shift Score = Impact Fraction × Distance Fraction**

- **Impact Fraction:** How much of the viewport changed? (e.g., a banner pushing down 50% of the screen).
- **Distance Fraction:** How far did the element move relative to the viewport's largest dimension?

A tiny movement of a huge element can be just as damaging as a huge movement of a tiny element.

## The "Silent" Killers

### 1. The Aspect Ratio Problem

If you don't define `width` and `height` on an image, the browser assumes it's 0x0 until the first few bytes of the image arrive. The moment the dimensions are known, the text underneath "jumps" down.
**The Fix:** Modern browsers use the `width` and `height` attributes to calculate the aspect ratio _before_ the image downloads. Always include them.

### 2. Late-Loading Injections

Ads and "Newsletter Signups" are notorious for this. They pop in at the top of the page 2 seconds after load.
**The Fix:** Pre-allocate the space. If an ad is usually 250px tall, give its container a `min-height: 250px`. It's better to have a blank gap that fills in than a jumpy UI.

### 3. Web Font "Size Jumps" (FOUT)

When a custom font loads, its character widths often differ from the system fallback font. This causes lines to wrap differently, shifting every paragraph below it.

**The Fix:** Use `font-display: swap` combined with the `size-adjust` property in `@font-face`. This allows you to scale the fallback font to match the custom font's "footprint" exactly.

```css
@font-face {
  font-family: "FallbackFont";
  src: local("Arial");
  size-adjust: 95%; /* Adjusts the size of the fallback to match the web font */
}
```

## The 500ms Grace Period

The browser ignores layout shifts that happen within 500ms of a user input (like a click). This is because some shifts are intentional (e.g., expanding a "See More" section). CLS only penalizes **unexpected** shifts.

## Pragmatic Evaluation

> **The Philosophy:** Stability is a form of performance. A fast page that jumps is a broken page.

### When to use this

- Designing content-heavy blogs or documentation sites.
- Implementing dynamic advertising or "newsletter" overlays.

### When to avoid

- Highly interactive game loops where the viewport is constantly changing by design.
- Full-screen animations where the entire layout is in motion intentionally.

---

Related [Largest Contentful Paint (LCP)](./lcp.md), [Critical Rendering Path](./critical-rendering-path.md), [Layout Thrashing](./layout-thrashing.md)
