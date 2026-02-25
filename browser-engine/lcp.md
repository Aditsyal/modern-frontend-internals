# Largest Contentful Paint (LCP)

LCP isn't just another vanity metric; it is the most accurate proxy we have for 'perceived load speed.' It marks the point where the user feels like the page is actually 'there.' If your LCP is slow, users bail, regardless of how fast your background scripts are running.

## The Thresholds of Success

To provide a good user experience, sites should strive to have Largest Contentful Paint of **2.5 seconds** or less.


| Metric Value | Status                      |
| ------------ | --------------------------- |
| ≤ 2.5s       | **Good** (Target)           |
| ≤ 4.0s       | **Needs Improvement**       |
| > 4.0s       | **Poor** (Critical Failure) |


## The LCP Candidates

The browser only cares about the biggest fish in the pond. It tracks the largest image, video poster, or text block visible in the viewport. As the page loads, the LCP candidate might change; for example, a large headline might be the initial LCP, only to be replaced by a hero image once it finishes downloading.

You can observe these candidates programmatically:

```js
new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    console.log("LCP Candidate:", entry.startTime, entry.element);
  }
}).observe({ type: "largest-contentful-paint", buffered: true });
```

## The Four Pillars of Delay

To fix a bad LCP, you have to identify which of the four stages is dragging you down:

1. **TTFB (Time to First Byte):** If your server is slow, everything else is pushed back. You can't render an image you haven't been told about yet.
2. **Resource Discovery Delay:** This is where most developers fail. If your hero image is discovered via a CSS background-image or a JS-injected tag, the browser's preload scanner can't find it until much later.
3. **Resource Load Duration:** The physical time it takes to pull the pixels over the wire. Large, unoptimized images are the primary culprits here.
4. **Element Render Delay:** Even after the image is downloaded, it might sit in memory while the main thread is blocked by a massive JavaScript bundle or a 'render-blocking' CSS file.

## Aggressive Optimization Tactics

Stop treating all resources as equal. Your LCP image should be the highest priority asset on the page.

- **Fetch Priority:** Use `fetchpriority="high"` on your LCP image. This tells the browser to bump it to the front of the network queue, even ahead of some scripts.
- **Eliminate Discovery Delay:** Never hide your LCP image in an external CSS file or a `useEffect`. Put a `<link rel="preload">` in the `<head>` or use a standard `<img>` tag in the HTML source so the preload scanner hits it immediately.
- **The "LCP-First" CSS Strategy:** Inline the CSS required to layout and style the LCP element directly in the `<head>`. This ensures the browser can paint the element the moment the pixels arrive.

## The Path to 'Good' (≤ 2.5s)

Achieving a consistent 'Good' rating requires a holistic approach across the entire delivery chain:

1. **Zero Discovery Delay:** The LCP resource must be present in the initial HTML. No `useEffect` or `background-image` discovery.
2. **Prioritized Delivery:** Use HTTP/3 and `fetchpriority="high"` to minimize queuing.
3. **Optimized Compression:** Use AVIF for images (20-50% smaller than WebP) and ensure they are appropriately sized for the device.
4. **Server Proximity:** Use an Edge Network/CDN to bring the TTFB below 200ms.
5. **Main-Thread Hygiene:** Ensure the main thread is idle when the LCP resource arrives so it can be painted immediately.

Modern formats like AVIF and WebP aren't optional anymore; they are requirements. If you're serving 500KB JPEGs as hero images, you've already lost the LCP game.

## Pragmatic Evaluation

> **The Philosophy:** LCP is a competition for the browser's preload scanner. If your hero element isn't in the raw HTML, you've already lost.

### When to use this

- Optimizing "above-the-fold" content for landing pages.
- Improving perceived performance on high-latency mobile networks.

### When to avoid

- Single-page applications (SPAs) where the initial view is just a loading spinner.
- Background-only or utility-heavy pages where visual load isn't the primary goal.

---

Related [Cumulative Layout Shift (CLS)](./cls.md), [Interaction to Next Paint (INP)](./inp.md), [Critical Rendering Path](./critical-rendering-path.md)