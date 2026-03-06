# Streaming SSR: Breaking the All-or-Nothing Barrier

Traditional SSR is a sequential bottleneck. The server must fetch all data, render the entire HTML string, and send the full response before the browser can show a single pixel. Streaming SSR breaks this sequence by sending HTML in chunks as they become ready.

## The HTTP Pipeline

Streaming leverages the `Transfer-Encoding: chunked` header in HTTP/1.1 or the native streaming capabilities of HTTP/2 and HTTP/3. This allows the server to keep the connection open and "push" new HTML segments as background data promises resolve.

## Technical Workflow in React 18/19

React uses `renderToPipeableStream` (Node.js) or `renderToReadableStream` (Edge/Web Streams) to orchestrate this. The process follows a strict hierarchy:

1.  **The Shell:** The server immediately sends the document `<head>` and the layout shell. The user sees the header and navigation instantly.
2.  **Suspense Fallbacks:** Any component wrapped in `<Suspense>` is sent as a placeholder (e.g., a skeleton or spinner).
3.  **The Payload:** As data fetches resolve in the background, React renders the final HTML for that specific component.
4.  **The Swap:** React sends the final HTML chunk followed by a tiny `<script>` tag. This script performs a DOM injection, swapping the fallback placeholder with the real content.

```ts
// Example using renderToPipeableStream (Node.js)
import { renderToPipeableStream } from 'react-dom/server';

function handleRequest(req, res) {
  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ['/main.js'],
    onShellReady() {
      res.setHeader('Content-Type', 'text/html');
      pipe(res);
    },
    onAllReady() {
      // Useful for crawlers
    }
  });
}
```

## Why This Beats Traditional SSR

- **TTFB (Time to First Byte):** Is near-instant because the server doesn't wait for the database.
- **Concurrent Parsing:** The browser starts parsing CSS and downloading JS while the server is still rendering the body content.
- **User Perceived Performance:** The page feels alive immediately, rather than showing a white screen for 2 seconds while a slow API call finishes.

## Next.js Implementation

In the Next.js App Router, every page is a stream by default. By using `loading.js` files or manual `<Suspense>` boundaries, you define the "granularity" of your stream. A well-architected streaming page ensures that the "Above the Fold" content is in the first chunk, while heavy data tables are streamed in later.

## Pragmatic Evaluation

> **The Philosophy:** Progressive enhancement on the rendering layer. Provide the "shell" as fast as possible, and let the remaining parts of the page "fill in" as they resolve.

### When to use this

- **High-Latency Data Sources:** When your database or API is slow, but you want to keep the user engaged.
- **Complex Page Structures:** When some parts of the page are computationally expensive to render but others are static.
- **SEO-Critical Pages:** Streaming allows search engines to start indexing the page content as soon as the first byte is received.

### When to avoid

- **Simplistic Pages:** If your data-fetching is near-instant and the page is lightweight, the overhead of streaming might not be necessary.
- **Legacy Browser Support:** While widely supported, extremely old environments might struggle with chunked transfer or DOM injection scripts.

---

Related [Suspense Boundaries](../react-core/suspense.md), [Server Components](../rendering-patterns/server-components.md)
