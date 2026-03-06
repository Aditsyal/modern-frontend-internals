# Partial Hydration: The Pay-for-Play Model

Partial hydration is an optimization strategy that eliminates "hydration overhead" for static content. In a "Full Hydration" model (like Next.js 12 or standard Vite apps), every component on the page is processed by the framework's runtime, even if it has no state or effects. Partial hydration ensures we only pay the JavaScript tax for parts of the UI that actually need it.

## The Cost of "Static" JS

When you ship a static `<footer>` inside a React app, you aren't just shipping HTML. You are shipping:

1. The component's source code in the JS bundle.
2. The serialized props (JSON) to hydrate it.
3. The CPU cycles required for the Virtual DOM to "verify" that the footer hasn't changed.

Partial hydration identifies these static subtrees during the build process and excludes them from the client-side bundle entirely.

## Selective vs. Partial Hydration

It is easy to confuse these two, but their goals differ:

- **Partial Hydration (Architectural):** Decides _what_ code is shipped. It physically removes JS for static components from the bundle.
- **Selective Hydration (Orchestrational):** Decides _when_ code is executed. It ships everything but uses Suspense to prioritize which parts hydrate first.

## Real-World Nuance

Astro is currently the gold standard for partial hydration. By default, every component is static. You must explicitly opt-in to interactivity using "client directives." This forces a "performance-first" mindset: if you don't add `client:load`, you don't ship JS.

## Pragmatic Evaluation

> **The Philosophy:** Partial hydration is the bridge between static and dynamic. It allows for modern DX without the JS tax, making it essential for content-heavy sites.

### When to use this

- **Content-Heavy Sites:** Blogs, e-commerce, and documentation where SEO and FCP are critical.
- **Documentation Sites:** Where 90% of the content is static.
- **Landing Pages:** To achieve 100/100 Lighthouse scores by eliminating the JS tax of static sections.

### When to avoid

- **Complex Dashboards:** Where components need to share deep state across the whole page.
- **Highly Interactive SPAs:** Where every element is dynamic; standard hydration might be more efficient for frequent state updates.

---

Related [Islands Architecture](./islands.md), [Selective Hydration](../react-core/suspense.md)
