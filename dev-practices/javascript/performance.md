# Performance

## requestAnimationFrame

Use `requestAnimationFrame` (rAF) for all visual updates — never `setTimeout` or `setInterval` for animation:

```js
function animate() {
  // update DOM here — runs before the next repaint
  updatePositions();
  requestAnimationFrame(animate);
}

const rafId = requestAnimationFrame(animate);

// cancel when done
cancelAnimationFrame(rafId);
```

- rAF is throttled when the tab is hidden — saves battery and CPU
- Keep rAF callbacks fast (< 16 ms for 60 fps) — move computation to a Web Worker

## Avoid Layout Thrashing

Reading layout properties (e.g., `offsetHeight`, `getBoundingClientRect`) after writing forces the browser to recalculate layout synchronously. Batch reads and writes:

```js
// bad — alternating read/write triggers forced reflow each time
boxes.forEach(box => {
  const h = box.offsetHeight;
  box.style.height = h * 2 + 'px';
});

// good — all reads, then all writes
const heights = boxes.map(box => box.offsetHeight);
boxes.forEach((box, i) => {
  box.style.height = heights[i] * 2 + 'px';
});
```

Properties that trigger layout: `offsetWidth/Height`, `clientWidth/Height`, `getBoundingClientRect`, `scrollTop`, `scrollLeft`, `getComputedStyle`.

## IntersectionObserver

Detect when elements enter the viewport without scroll event listeners:

```js
const observer = new IntersectionObserver((entries) => {
  for (const entry of entries) {
    if (entry.isIntersecting) {
      loadImage(entry.target);
      observer.unobserve(entry.target);   // stop watching once loaded
    }
  }
}, { rootMargin: '200px' });   // pre-load 200px before visible

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img));
```

- Use for lazy loading images, infinite scroll, and scroll-driven animations
- `rootMargin` offsets the intersection boundary — use positive values for pre-loading

## Debounce and Throttle

Limit how often expensive callbacks fire in response to high-frequency events:

```js
// debounce — fires AFTER the burst ends (e.g., search-as-you-type)
function debounce(fn, wait) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), wait);
  };
}

// throttle — fires AT MOST once per interval (e.g., resize, scroll)
function throttle(fn, interval) {
  let last = 0;
  return (...args) => {
    const now = Date.now();
    if (now - last >= interval) {
      last = now;
      fn(...args);
    }
  };
}

input.addEventListener('input', debounce(search, 300));
window.addEventListener('resize', throttle(recalcLayout, 100));
```

- Debounce: `resize` on complex layouts, `input` for live search, form validation
- Throttle: `scroll` for progress bars, `mousemove` for drag interactions

## Memory Leaks

Common causes and fixes:

| Cause | Fix |
|-------|-----|
| Event listeners never removed | Use `AbortController` or remove explicitly on teardown |
| Timers never cleared | Call `clearTimeout`/`clearInterval` on teardown |
| Closures holding large data | Set references to `null` when done |
| Detached DOM nodes referenced in JS | Release the reference when removing from DOM |
| MutationObserver/IntersectionObserver not disconnected | Call `.disconnect()` on teardown |

```js
// pattern: component with cleanup
function createWidget(container) {
  const controller = new AbortController();
  const rafId = requestAnimationFrame(tick);

  function destroy() {
    controller.abort();
    cancelAnimationFrame(rafId);
  }

  return { destroy };
}
```

## Other Quick Wins

- Use CSS `transform` and `opacity` for animation — they do not trigger layout or paint
- Add `will-change: transform` sparingly on elements that animate — promotes to GPU layer
- Use `<img loading="lazy">` and `fetchpriority="high"` in HTML before writing JS lazy loading
- Prefer `textContent` over `innerHTML` — no HTML parsing overhead
- Avoid synchronous `localStorage` reads in hot paths — cache the value in a variable
