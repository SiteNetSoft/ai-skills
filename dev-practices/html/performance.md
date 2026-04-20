# Performance

## Resource Loading Attributes

### Scripts

| Attribute | Behavior | When to use |
|-----------|----------|-------------|
| (none) | Blocks HTML parsing; fetches and executes inline | Almost never for external scripts |
| `defer` | Fetches in parallel; executes after HTML parsed, in order | Default choice for most `<script src="...">` |
| `async` | Fetches in parallel; executes as soon as fetched, out of order | Independent analytics/tracking scripts |
| `type="module"` | Deferred by default; strict mode; `import`/`export` | ES module entry points |

```html
<!-- Recommended: defer preserves execution order, does not block parsing -->
<script src="app.js" defer></script>

<!-- Analytics that do not depend on DOM order -->
<script src="analytics.js" async></script>

<!-- ES modules are deferred automatically -->
<script type="module" src="main.js"></script>
```

- Place `<script defer>` in `<head>` — deferred scripts in `<head>` start downloading earlier than scripts at the end of `<body>`
- Never block rendering with `<script>` in `<head>` without `defer` or `async` unless the script must run before first render (e.g., theme detection)

### Resource Hints

```html
<!-- Preload: fetch this resource ASAP, it is needed for current page -->
<link rel="preload" href="hero.jpg" as="image" fetchpriority="high">
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="critical.css" as="style">

<!-- Prefetch: fetch this resource for a likely next navigation (low priority) -->
<link rel="prefetch" href="/about" as="document">

<!-- Preconnect: establish connection early for third-party origins -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- DNS-prefetch: DNS lookup only (fallback for browsers without preconnect) -->
<link rel="dns-prefetch" href="https://cdn.example.com">
```

- `preload` is a strong signal — only use it for resources on the critical path; overusing it wastes bandwidth and can lower priority of truly critical resources
- `crossorigin` is required on font preloads even for same-origin fonts
- `fetchpriority="high"` on the LCP image (`<img>` or `<link rel="preload">`) improves LCP

## Critical Rendering Path

The browser cannot render until it has:
1. The HTML (constructs the DOM)
2. All non-deferred CSS (constructs the CSSOM)
3. Any render-blocking scripts

Reduce time to first render:

- **Inline critical CSS** — styles needed for above-the-fold content can be inlined in `<style>` to avoid a round trip
- **Defer non-critical CSS** — load stylesheets needed only below the fold asynchronously:

  ```html
  <link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">
  <noscript><link rel="stylesheet" href="non-critical.css"></noscript>
  ```

- **Minimize render-blocking scripts** — use `defer`/`async`; move third-party scripts out of the critical path
- **Avoid CSS `@import`** in stylesheets — each `@import` creates an additional request that blocks rendering; use `<link>` tags instead

## Core Web Vitals Impact

| Metric | What it measures | HTML levers |
|--------|-----------------|-------------|
| LCP (Largest Contentful Paint) | Load time of the largest above-fold element | Preload LCP image, `fetchpriority="high"`, avoid lazy-load on LCP image |
| CLS (Cumulative Layout Shift) | Visual stability | `width`+`height` on images/video, `aspect-ratio` on containers, avoid injecting content above existing content |
| INP (Interaction to Next Paint) | Responsiveness | Minimize blocking scripts, avoid large inline scripts |

```html
<!-- CLS: reserve space for images -->
<img src="card.jpg" alt="..." width="400" height="300" loading="lazy">

<!-- LCP: prioritize hero image -->
<link rel="preload" as="image" href="hero.avif" fetchpriority="high">
<img src="hero.avif" alt="..." width="1200" height="630" loading="eager">
```

## Fonts

```html
<!-- 1. Preconnect to font CDN -->
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- 2. Preload the specific font file(s) used above the fold -->
<link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin>
```

```css
/* 3. font-display: swap avoids invisible text while font loads */
@font-face {
  font-family: 'Inter';
  src: url('/fonts/inter-var.woff2') format('woff2');
  font-display: swap;
}
```

- Use `font-display: swap` or `optional` to prevent Flash of Invisible Text (FOIT)
- Prefer variable fonts — one file covers all weights and styles
- Subset fonts to only the characters you need

## Minimal Markup

- Every DOM node has a cost: parsing, style calculation, layout, paint
- Avoid wrapper `<div>` elements that serve no structural or semantic purpose
- Remove comments from production HTML (template engines/bundlers should strip them)
- Do not use HTML to apply spacing — use CSS margin/padding

```html
<!-- Unnecessary nesting -->
<div class="wrapper">
  <div class="container">
    <div class="inner">
      <p>Content</p>
    </div>
  </div>
</div>

<!-- Sufficient -->
<p class="feature-text">Content</p>
```

## `<head>` Order

A well-ordered `<head>` avoids render-blocking surprises:

```html
<head>
  <!-- 1. Charset first — must be within first 1024 bytes -->
  <meta charset="UTF-8">

  <!-- 2. Viewport -->
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

  <!-- 3. Title -->
  <title>Page Title</title>

  <!-- 4. Preconnect / DNS-prefetch for critical origins -->
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

  <!-- 5. Preload critical resources -->
  <link rel="preload" href="hero.avif" as="image" fetchpriority="high">

  <!-- 6. Stylesheets -->
  <link rel="stylesheet" href="styles.css">

  <!-- 7. Deferred scripts -->
  <script src="app.js" defer></script>
</head>
```
