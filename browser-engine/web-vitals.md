# Core Web Vitals Guide

Lighthouse scores are "Lab Data", they are a simulation in a clean room. "Field Data" (CrUX) is the only thing that matters. It's the record of how your site performed for a guy on a 3-year-old Android phone over a spotty 3G connection in a subway.

## The 75th Percentile Rule

Google doesn't care about your average speed. They care about the **75th percentile (p75)**. If 74% of your users have a fast site but the bottom 25% are suffering, you fail. Optimization is a fight to bring the "tail" of your performance distribution inward.

## LCP: The "Perceived" Load

Largest Contentful Paint is the moment the user thinks, "Okay, the page is here."

- **Common Failure:** Putting your hero image in a CSS `background-image` or lazy-loading it. The browser's scanner can't find these until the CSS/JS is parsed.
- **The Fix:** Use a standard `<img>` tag with `fetchpriority="high"`. This tells the browser to put this asset at the front of the network queue.

## INP: The "Feel" of the App

Interaction to Next Paint (INP) is now the primary metric for responsiveness, having officially replaced First Input Delay (FID) in 2024. FID only measured the **first** interaction and only the delay, making it a "vanity metric" that many slow sites could pass easily. INP monitors **every** click, tap, and keypress across the entire session, measuring the total time from input to visual feedback.

- If your app feels "heavy" or "janky" after it's loaded, your INP will suffer.
- This is almost always caused by **Long Tasks** blocking the main thread. If you can't reduce the JS, you must yield the thread.

## Diagnostic Workflow: Root Cause Analysis

When a metric drops, don't just start minifying images.

1. **Slow TTFB?** It's a server/CDN issue.
2. **Slow LCP?** It's a resource prioritization issue.
3. **Slow INP?** It's a main-thread execution issue.
4. **High CLS?** It's a layout/CSS issue.

## Pragmatic Evaluation

> **The Philosophy:** Field data (CrUX) is the only source of truth for user experience; lab data is a development-only proxy.

### When to use this

- Assessing real-world user experience across diverse device distributions.
- Investigating the impact of network conditions on business KPIs.

### When to avoid

- During local development where network/CPU is stable (use Lighthouse/Performance panel instead).
- For internal-only applications where the environment is fully controlled.

---

Related [Critical Rendering Path](./critical-rendering-path.md), [Long Tasks API](./long-tasks.md), [Priority Hints](../web-platform-apis/resource-prioritization.md), [Speculative Prerendering](../web-platform-apis/speculative-optimization.md)
