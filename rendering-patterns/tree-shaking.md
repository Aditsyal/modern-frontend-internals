# Tree Shaking Internals

Tree Shaking is the process of dead-code elimination, but the name is a bit of a misnomer. It's more like a "Live-Code Inclusion" system. Bundlers start at your entry point and trace every connection, throwing away anything that isn't explicitly reached.

## The ESM Requirement

Tree shaking _only_ works reliably with **ES Modules** (`import` and `export`). This is because ESM is static. The bundler can analyze the imports without running a single line of code.

Compare this to CommonJS (`require`), where you can do `require(Math.random() > 0.5 ? 'a' : 'b')`. Because the dependency is determined at runtime, the bundler has no choice but to include everything, "just in case."

## The Enemy: Side Effects

The biggest reason your bundles are still bloated is **Side Effects**. If a module does anything when it's loaded (like modifying a global variable or logging to the console), the bundler cannot safely "shake" it away, even if you never use its exports.

## The `sideEffects` Flag

As a library author, you must be explicit. By setting `"sideEffects": false` in your `package.json`, you are giving the bundler a "License to Kill." You're promising that your modules only export values and don't touch anything else. If a file isn't imported, the bundler can delete it without fear of breaking the app.

## Design for Shakeability

To make your code truly shakeable:

1.  **Use Atomic Exports:** Export many small functions instead of one giant "Utils" object.
2.  **Avoid Class Bloat:** Classes are hard to shake because their methods are attached to the prototype, making it difficult for bundlers to prove they aren't being used dynamically.
3.  **Check Your Dist:** Look at your generated code. If you see code from a library you only used one function from, that library is either not using ESM or has "dirty" side effects.

## Bundler Comparison: Shaking Effectiveness

| Bundler     | Shaking Strategy              | Effectiveness                   |
| :---------- | :---------------------------- | :------------------------------ |
| **Rollup**  | Scope Hoisting + ESM analysis | High                            |
| **esbuild** | Native Go implementation      | Very High                       |
| **Webpack** | Plugin-based with Terser      | Medium (requires manual config) |

## Pragmatic Evaluation

> **The Philosophy:** Optimize by inclusion. Only send the code that is strictly used by the application, assuming libraries are designed with side-effect-free ESM.

### When to use this

- **Building Shared Libraries:** Always ensure your library is ESM-first and includes the `sideEffects` flag.
- **Large Component Libraries:** When you are importing single components from a large library like `lucide-react` or `radix-ui`.
- **Standardizing Utilities:** When your project has dozens of utility functions, atomic exports ensure you only pay for what you use.

### When to avoid

- **Legacy Projects:** If you are stuck with CommonJS modules, tree shaking won't give you any meaningful benefit.
- **Micro-apps:** In very small applications, the difference between a few kilobytes might be outweighed by the complexity of configuring advanced bundlers.

---

Related [Code Splitting Strategies](../rendering-patterns/code-splitting.md), [LCP](../browser-engine/lcp.md)
