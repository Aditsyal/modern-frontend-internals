# Code Splitting Strategies

Shipping a 2MB JavaScript bundle is like trying to force a user to read the entire library before they can enter the building. Code splitting is the art of delivering only the "pages" the user needs right now.

## Route-Based Splitting: The Bare Minimum

If your "Admin Dashboard" code is being downloaded by a user on the "Login" page, you are failing. Route-level splitting should be your default state.

- Use `React.lazy` or equivalent framework features to wrap every major route.
- This ensures that the initial payload only contains the framework core and the specific code for the landing page.

## The "Above the Fold" Component Split

Don't stop at routes. Modals, complex data visualizations (D3, Three.js), and "heavy" components should be split out.

- **The Rule:** If a component isn't visible on the initial render (e.g., a "Settings" modal), it shouldn't be in the main bundle.
- Use dynamic `import()` inside an event handler to fetch the code only when the user clicks the button.

```ts
// Lazy-load a heavy component on user interaction
function OpenChartButton() {
  const [Chart, setChart] = useState<any>(null);

  const handleClick = async () => {
    // Dynamic import kicks off the network request
    const { HeavyChart } = await import('./HeavyChart');
    setChart(() => HeavyChart);
  };

  return (
    <>
      <button onClick={handleClick}>Load Data Viz</button>
      {Chart && <Chart data={data} />}
    </>
  );
}
```

## Vendor Splitting & Long-Term Caching

Third-party libraries (`lodash`, `moment`, `react-dom`) change far less often than your business logic.

- By splitting `node_modules` into a separate `vendor.js` chunk, you allow the user to keep that heavy file in their browser cache across multiple deployments.
- When you update a single line of your code, the user only has to download a tiny `app.js` update, not the entire 500KB library set.

## Speculative Loading: Prefetch vs. Preload

Code splitting can introduce a "delay" when the user clicks a button and has to wait for the new chunk.

1.  **Preload:** Use for resources needed _now_. If the user is definitely going to see a chart, preload it.
2.  **Prefetch:** Use for resources needed _soon_. Tell the browser to download the "Dashboard" chunk while the user is still idling on the "Home" page. The browser will fetch it with low priority, ensuring it's ready the moment the user navigates.

## Pragmatic Evaluation

> **The Philosophy:** Defer the cost of downloading code until it is absolutely necessary for the current view or likely to be used soon.

### When to use this

- **SaaS Platforms:** Dashboards with many modals and complex features that are not always needed.
- **Large Component Libraries:** When your UI kit includes dozens of components that are only used sparingly.
- **Improving LCP (Largest Contentful Paint):** By reducing the main bundle size, you ensure the browser can reach the painting phase faster.

### When to avoid

- **Micro-apps:** If your entire app is under 50KB, splitting it into multiple files can actually hurt performance due to additional HTTP request overhead.
- **Server-Side Only Logic:** If you are using RSCs, the "code splitting" is handled at the network boundary -- the JS isn't sent at all.

---

Related [LCP](../browser-engine/lcp.md)
