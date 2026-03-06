# Edge Rendering

Edge Rendering moves the "brains" of your application from a central warehouse in Virginia or Ireland to a convenience store at the end of the user's street. It is the ultimate weapon against the speed of light.

## 1. The Geography of Latency

Traditional SSR is hampered by physical distance. A user in Tokyo requesting a site hosted in London must wait for signals to traverse the globe and back. This "round trip time" (RTT) creates a hard floor for how fast your site can feel.

Edge Rendering intercepts the request at the nearest CDN node. By executing code on these "Edge" servers, you provide HTML in milliseconds, regardless of where the origin server lives.

## 2. The Edge Runtime: V8 Without the Baggage

Edge functions don't use the full Node.js environment. They utilize lightweight V8 Isolates -- the same technology that powers browser tabs.

- **Cold Starts are Dead:** Unlike traditional AWS Lambda, which might take seconds to "spin up" a container, V8 Isolates start in under 5 milliseconds.
- **The Tradeoff:** You lose access to the local filesystem (`fs`) and legacy Node.js modules. You are restricted to Web Standard APIs like `fetch`, `Request`, and `Response`. This forces you to write cleaner, more portable code.

## 3. Strategic Use Cases

### Dynamic Personalization

Stop using client-side "flicker" for A/B tests. The Edge can inspect a user's cookie, geographic location, or device type and rewrite the HTML stream on the fly. The user receives a perfectly tailored page in the very first byte.

### Distributed Authentication

Don't let unauthorized requests even touch your expensive origin server. Verify JWTs or session tokens at the Edge and issue redirects immediately. This hardens your security and saves origin resources.

## 4. The Data Gravity Problem

Edge rendering is fast, but it can't fix a slow database. If your data is trapped in a single region, your Edge function still has to wait for that data to travel across the world.

### Regional vs. Global Data Fetching

1.  **Regional:** The server is at the edge, but the DB is in `us-east-1`. The RTT for the data fetch kills the benefit of edge rendering.
2.  **Global:** Pair edge rendering with distributed databases like Turso (LibSQL), DynamoDB Global Tables, or Upstash (Redis). This ensures that the data is as close to the user as the code is.

## 5. Pragmatic Evaluation

> **The Philosophy:** Shift rendering to the network's periphery to minimize RTT while accepting a restricted execution environment.

### When to use this

- **Geographically Distributed Users:** When your audience is global and you want a consistent experience across regions.
- **Performance-Sensitive Routes:** Landing pages, marketing sites, and high-traffic entry points.
- **Micro-frontends:** Using the Edge to "stitch" together multiple micro-apps before the user receives the final HTML.

### When to avoid

- **Heavy Computation:** Edge functions are often billed by execution time and have memory limits (e.g., 128MB).
- **Deep Node.js Dependency:** If your app relies on native Node modules or large libraries that aren't compatible with V8 Isolates.
- **Centralized Data Sources:** If your database is strictly pinned to one region and cannot be replicated or accessed with low latency from the edge.

---

Related [Server Components](../rendering-patterns/server-components.md)
