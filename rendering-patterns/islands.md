# Islands Architecture: Decoupling Interactivity

Islands architecture treats a web page as a static ocean of HTML containing isolated, self-contained "islands" of interactivity. Unlike traditional Single Page Applications (SPAs) that hydrate from the root down, Islands architecture hydrates from the components up.

## Structural Logic

In a standard React or Next.js app, the entire page is one large JavaScript tree. Even if 90% of the page is static text, the browser must download the logic for that text. Islands architecture (pioneered by Astro and Fresh) flips this:

1.  **The Ocean:** The server generates pure HTML for the layout, text, and images. This content requires zero client-side JavaScript.
2.  **The Islands:** Interactive widgets (carousels, search bars, login forms) are treated as independent entry points.

## Why it Wins on Performance

The primary bottleneck of modern web apps is **Total Blocking Time (TBT)**. By breaking the UI into independent islands, we achieve:

- **Fragmented Bundles:** Instead of one 200KB bundle, the browser loads four 5KB bundles only when needed.
- **Lazy Hydration:** We can delay an island's hydration until it enters the viewport (`client:visible`) or the browser is idle (`client:idle`).
- **Parallelism:** One heavy island cannot block the hydration of another.

## Technical Implementation: The Placeholder Pattern

The server renders the island's HTML and leaves a marker (often a `<script>` or a custom element). This marker contains the serialized props required for that specific component.

```astro
<!-- The Layout and Sidebar are pure HTML -->
<Sidebar />

<main>
  <ArticleContent />
  <!-- The Island: Only this part ships JS to the browser -->
  <CommentSection client:visible />
</main>
```

## Framework Agnosticism

Because islands are independent, they can technically use different frameworks. You can run a high-performance Solid.js search bar next to a complex React data grid on the same page. The "Ocean" doesn't care; it only sees the HTML and the hydration triggers.

## Pragmatic Evaluation

> **The Philosophy:** Islands Architecture is a powerful tool for public-facing, content-heavy sites. Do not use it for complex state-heavy dashboards where cross-component communication is the primary requirement.

### When to use this

- **E-commerce & Content Sites:** Where SEO and First Contentful Paint (FCP) are revenue-critical.
- **Documentation:** Where 90% of the content is static text.
- **Landing Pages:** To achieve 100/100 Lighthouse scores by eliminating the JS tax of static sections.

### When to avoid

- **Complex Admin Panels:** Where components need to share deep state (e.g., a multi-step form spanning the whole page).
- **Social Media Feeds:** Where every element is dynamic and highly interactive; a standard SPA with virtualization is often more efficient.

---

Related [Partial Hydration](./partial-hydration.md), [Selective Hydration](../react-core/suspense.md)
