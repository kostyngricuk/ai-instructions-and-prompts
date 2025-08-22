---
applyTo: '**/*.tsx, **/*.ts, **/*.jsx, **/*.js, **/*.css, **/*.html'
description: 'Comprehensive frontend performance optimization instructions covering React, Vue, Angular, DOM manipulation, asset optimization, and web performance best practices.'
---

# Frontend Performance Optimization Best Practices

## Introduction

Frontend performance is the difference between a delightful user experience and frustrated users who abandon your app. This guide focuses on practical, real-world frontend performance optimizations that make an immediate impact on user experience, Core Web Vitals, and overall application responsiveness.

---

## Frontend Performance Principles

- **Measure First, Optimize Second:** Always profile and measure before optimizing. Use Chrome DevTools, Lighthouse, and Web Vitals to identify real bottlenecks.
  - *Pro Tip:* Focus on Core Web Vitals (LCP, FID, CLS) for user-centric metrics that impact SEO and UX.
- **Optimize for the Critical Rendering Path:** Prioritize above-the-fold content and defer non-critical resources.
- **Minimize Resource Usage:** Reduce bundle sizes, optimize images, and eliminate unused code through tree-shaking.
- **Prefer Native Web APIs:** Modern browsers provide efficient APIs that often outperform heavy libraries.
- **Document Performance Assumptions:** Comment on performance-critical code sections and optimization decisions.
- **Automate Performance Testing:** Integrate Lighthouse CI and performance budgets into your build process.
- **Test on Real Devices:** Mobile performance often differs significantly from desktop, especially on slower networks.

---

## Frontend Performance

### Rendering and DOM
- **Minimize DOM Manipulations:** Batch updates where possible. Frequent DOM changes are expensive.
  - *Anti-pattern:* Updating the DOM in a loop. Instead, build a document fragment and append it once.
- **Virtual DOM Frameworks:** Use React, Vue, or similar efficiently—avoid unnecessary re-renders.
  - *React Example:* Use `React.memo`, `useMemo`, and `useCallback` to prevent unnecessary renders.
- **Keys in Lists:** Always use stable keys in lists to help virtual DOM diffing. Avoid using array indices as keys unless the list is static.
- **Avoid Inline Styles:** Inline styles can trigger layout thrashing. Prefer CSS classes.
- **CSS Animations:** Use CSS transitions/animations over JavaScript for smoother, GPU-accelerated effects.
- **Defer Non-Critical Rendering:** Use `requestIdleCallback` or similar to defer work until the browser is idle.

### Asset Optimization
- **Image Compression:** Use tools like ImageOptim, Squoosh, or TinyPNG. Prefer modern formats (WebP, AVIF) for web delivery.
- **SVGs for Icons:** SVGs scale well and are often smaller than PNGs for simple graphics.
- **Minification and Bundling:** Use Webpack, Rollup, or esbuild to bundle and minify JS/CSS. Enable tree-shaking to remove dead code.
- **Cache Headers:** Set long-lived cache headers for static assets. Use cache busting for updates.
- **Lazy Loading:** Use `loading="lazy"` for images, and dynamic imports for JS modules/components.
- **Font Optimization:** Use only the character sets you need. Subset fonts and use `font-display: swap`.

### Network Optimization
- **Reduce HTTP Requests:** Combine files, use image sprites, and inline critical CSS.
- **HTTP/2 and HTTP/3:** Enable these protocols for multiplexing and lower latency.
- **Client-Side Caching:** Use Service Workers, IndexedDB, and localStorage for offline and repeat visits.
- **CDNs:** Serve static assets from a CDN close to your users. Use multiple CDNs for redundancy.
- **Defer/Async Scripts:** Use `defer` or `async` for non-critical JS to avoid blocking rendering.
- **Preload and Prefetch:** Use `<link rel="preload">` and `<link rel="prefetch">` for critical resources.

### JavaScript Performance
- **Avoid Blocking the Main Thread:** Offload heavy computation to Web Workers.
- **Debounce/Throttle Events:** For scroll, resize, and input events, use debounce/throttle to limit handler frequency.
- **Memory Leaks:** Clean up event listeners, intervals, and DOM references. Use browser dev tools to check for detached nodes.
- **Efficient Data Structures:** Use Maps/Sets for lookups, TypedArrays for numeric data.
- **Avoid Global Variables:** Globals can cause memory leaks and unpredictable performance.
- **Avoid Deep Object Cloning:** Use shallow copies or libraries like lodash's `cloneDeep` only when necessary.

### Accessibility and Performance
- **Accessible Components:** Ensure ARIA updates are not excessive. Use semantic HTML for both accessibility and performance.
- **Screen Reader Performance:** Avoid rapid DOM updates that can overwhelm assistive tech.

### Framework-Specific Tips
#### React
- Use `React.memo`, `useMemo`, and `useCallback` to avoid unnecessary renders.
- Split large components and use code-splitting (`React.lazy`, `Suspense`).
- Avoid anonymous functions in render; they create new references on every render.
- Use `ErrorBoundary` to catch and handle errors gracefully.
- Profile with React DevTools Profiler.

#### Angular
- Use OnPush change detection for components that don't need frequent updates.
- Avoid complex expressions in templates; move logic to the component class.
- Use `trackBy` in `ngFor` for efficient list rendering.
- Lazy load modules and components with the Angular Router.
- Profile with Angular DevTools.

#### Vue
- Use computed properties over methods in templates for caching.
- Use `v-show` vs `v-if` appropriately (`v-show` is better for toggling visibility frequently).
- Lazy load components and routes with Vue Router.
- Profile with Vue Devtools.

#### React Native
- Use `React.memo`, `useMemo`, and `useCallback` to optimize component re-renders.
- Use FlatList or SectionList for efficient rendering of large lists.
- Specify image dimensions and enable caching to optimize image performance.
- Leverage native animation drivers and avoid heavy computation on the JS thread.
- Profile performance with React Native Debugger and Hermes.

### Common Frontend Pitfalls
- Loading large JS bundles on initial page load.
- Not compressing images or using outdated formats.
- Failing to clean up event listeners, causing memory leaks.
- Overusing third-party libraries for simple tasks.
- Ignoring mobile performance (test on real devices!).

### Frontend Troubleshooting
- Use Chrome DevTools' Performance tab to record and analyze slow frames.
- Use Lighthouse to audit performance and get actionable suggestions.
- Use WebPageTest for real-world load testing.
- Monitor Core Web Vitals (LCP, FID, CLS) for user-centric metrics.

## Practical Examples

### Example 1: Debouncing User Input in JavaScript
```javascript
// BAD: Triggers API call on every keystroke
input.addEventListener('input', (e) => {
  fetch(`/search?q=${e.target.value}`);
});

// GOOD: Debounce API calls
let timeout;
input.addEventListener('input', (e) => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    fetch(`/search?q=${e.target.value}`);
  }, 300);
});
```

### Example 2: Lazy Loading Images in HTML
```html
<!-- BAD: Loads all images immediately -->
<img src="large-image.jpg" />

<!-- GOOD: Lazy loads images -->
<img src="large-image.jpg" loading="lazy" />
```

### Example 3: Efficient React Component Optimization
```jsx
// BAD: Recreates function on every render
const Component = ({ items }) => {
  return (
    <div>
      {items.map((item, index) => (
        <div key={index} onClick={() => handleClick(item)}>
          {item.name}
        </div>
      ))}
    </div>
  );
};

// GOOD: Memoized component with stable keys
const Component = React.memo(({ items }) => {
  const handleClick = useCallback((item) => {
    // handle click
  }, []);

  return (
    <div>
      {items.map((item) => (
        <div key={item.id} onClick={() => handleClick(item)}>
          {item.name}
        </div>
      ))}
    </div>
  );
});
```

### Example 4: CSS Performance Optimization
```css
/* BAD: Expensive selectors and reflows */
.container * {
  position: relative;
}

div:nth-child(odd) p:first-child {
  color: red;
}

/* GOOD: Efficient selectors and GPU acceleration */
.container-item {
  contain: layout style paint;
}

.highlight {
  color: red;
  transform: translateZ(0); /* GPU layer */
}
```

---

## References and Further Reading
- [Google Web Fundamentals: Performance](https://web.dev/performance/)
- [MDN Web Docs: Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [WebPageTest](https://www.webpagetest.org/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [React Performance](https://react.dev/learn/render-and-commit)
- [Vue Performance](https://vuejs.org/guide/best-practices/performance.html)
- [Angular Performance](https://angular.io/guide/performance-checklist)
- [Web.dev Performance](https://web.dev/learn/performance/)
- [Chrome DevTools Performance](https://developer.chrome.com/docs/devtools/performance/)

---

## Conclusion

Frontend performance optimization is an ongoing process. Always measure, profile, and iterate. Use these best practices, checklists, and troubleshooting tips to guide your development and code reviews for high-performance, scalable, and efficient web applications. Focus on Core Web Vitals and real user experience metrics to make the biggest impact.

---

<!-- End of Performance Optimization Instructions --> 