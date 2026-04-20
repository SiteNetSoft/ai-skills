# Performance

## Critical CSS

Inline above-the-fold styles in `<style>` to eliminate render-blocking requests.

```html
<head>
  <!-- Inlined critical path styles -->
  <style>
    *, *::before, *::after { box-sizing: border-box; }
    body { margin: 0; font-family: system-ui, sans-serif; }
    .hero { min-height: 100dvh; display: flex; align-items: center; }
  </style>

  <!-- Non-critical styles load asynchronously -->
  <link rel="stylesheet" href="/styles/main.css" media="print" onload="this.media='all'">
  <noscript><link rel="stylesheet" href="/styles/main.css"></noscript>
</head>
```

- Tools like [Critical](https://github.com/addyosmani/critical) automate extraction
- Aim for under 14 KB of inlined critical CSS (first TCP congestion window)
- Revisit critical CSS when page structure changes

---

## content-visibility

Skips rendering for off-screen content — large gains for long pages.

```css
/* Sections below the fold skip layout + paint until close to viewport */
.page-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 600px; /* placeholder size to prevent scroll jumps */
}
```

- `content-visibility: auto` — browser decides when to render (recommended)
- `content-visibility: hidden` — always skip rendering (like `display:none` but preserves state)
- Pair with `contain-intrinsic-size` to give a height estimate and prevent layout shifts
- Not suitable for above-the-fold content or elements that must be accessible at all times

---

## CSS Containment (contain)

Tells the browser a subtree is independent — enables layout/paint/style optimizations.

```css
.card {
  contain: layout paint; /* Changes inside do not affect outside layout or paint */
}

/* Shorthand */
.isolated-component {
  contain: strict; /* = layout style paint size — most isolation */
}
```

| Value | Effect |
|-------|--------|
| `layout` | Internal layout changes do not affect outside |
| `paint` | Contents clipped to box; enables paint skipping |
| `style` | Counter scoping (limited use) |
| `size` | Element size not influenced by children |
| `strict` | All of the above |
| `content` | layout + paint + style |

- `contain: layout paint` is the sweet spot for most components
- Do not use `contain: size` unless you explicitly set a size — it collapses the element otherwise

---

## will-change (Revisited)

Promotes element to a GPU composite layer ahead of time.

```css
/* Only apply when animation is imminent */
.menu[data-open="true"] {
  will-change: transform, opacity;
}
```

Rules:
- Apply just before the animation starts; remove it after it ends
- Never blanket-apply to many elements — GPU memory cost compounds
- Check DevTools "Layers" panel to confirm intent and detect regressions
- Do not use as a fix for animation jank without profiling first

---

## Avoiding Expensive Selectors

| Pattern | Problem | Fix |
|---------|---------|-----|
| `* { … }` | Matches everything, re-runs on every DOM change | Scope tightly |
| `div div div span` | Long descendant chains | Use a direct class |
| `[class*="col-"]` | Substring attribute match is slow at scale | Direct class or `[class~="col"]` |
| `:nth-child(n)` on large lists | Recalculated on every mutation | Add a class when possible |
| Deeply nested `:not(:last-child)` | Complex combinators | Flatten with BEM |

- CSS selector matching is fast by default — only optimize when DevTools shows "recalculate style" cost
- The rendering cost of a bad selector is usually much smaller than a bad layout or unnecessary JS

---

## Font Loading Strategies

```html
<!-- Preload critical font files -->
<link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin>
```

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-var.woff2") format("woff2");
  font-weight: 100 900;     /* variable font range */
  font-display: swap;       /* show fallback immediately; swap when loaded */
  font-style: normal;
}
```

| font-display value | Behavior |
|--------------------|----------|
| `auto` | Browser default (usually block) |
| `block` | Short block period, then swap — invisible text initially |
| `swap` | Zero block period — FOUT but no invisible text |
| `fallback` | Short block, short swap — balanced |
| `optional` | Short block, no swap — use only if network is fast |

- Use `swap` for body text (avoid invisible text / FOIT)
- Use `optional` for decorative fonts where fallback is acceptable
- Use variable fonts (`font-weight: 100 900`) to ship one file instead of many
- Subset fonts to reduce file size: [google-webfonts-helper](https://gwfh.mranftl.com/)

---

## Reducing Layout Shifts (CLS)

```css
/* Reserve space for images before they load */
img {
  aspect-ratio: attr(width) / attr(height); /* requires width/height HTML attrs */
}

/* Or explicit ratio */
.hero-image {
  aspect-ratio: 16 / 9;
  width: 100%;
}

/* Avoid inserting content above existing content */
.banner {
  min-height: 56px; /* match expected height */
}
```

- Always set `width` and `height` attributes on `<img>` — browsers use them before load
- Use `aspect-ratio` on embedded media, iframes, and lazy-loaded images
- Avoid dynamically injecting content above the fold

---

## Reducing Paint Area

```css
/* Clip contents to box — browser can skip painting outside */
.scroll-container {
  overflow: hidden;
  contain: paint;
}

/* Avoid expensive effects on frequently animated elements */
.animated-bg {
  /* Use transform instead of changing background-position */
  background-image: url("tile.svg");
  animation: slide 10s linear infinite;
}

@keyframes slide {
  to { transform: translateX(-50%); } /* moves element, not background */
}
```

---

## Resource Hints

```html
<!-- Preconnect to font CDN -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- Prefetch a stylesheet needed on next page -->
<link rel="prefetch" href="/styles/checkout.css" as="style">
```

- `preconnect` — warms up DNS + TLS before the browser needs the resource
- `preload` — high-priority fetch for current page resources
- `prefetch` — low-priority fetch for resources needed soon (next page)
