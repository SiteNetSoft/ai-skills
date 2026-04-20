# Responsive Design

## Mobile-First Approach

- Write base styles for the smallest viewport first
- Use `min-width` media queries to progressively add complexity for larger screens
- Never override mobile styles back in desktop — that is desktop-first and creates extra work

```css
/* Base — mobile */
.card {
  padding: 1rem;
  flex-direction: column;
}

/* Tablet and up */
@media (width >= 640px) {
  .card {
    flex-direction: row;
    padding: 1.5rem;
  }
}

/* Desktop and up */
@media (width >= 1024px) {
  .card {
    padding: 2rem;
  }
}
```

---

## Media Query Syntax (Modern)

Prefer range syntax over `min-width`/`max-width` with `and`:

```css
/* Old */
@media (min-width: 768px) and (max-width: 1023px) { }

/* New range syntax */
@media (768px <= width < 1024px) { }
@media (width >= 768px) { }
```

### Common breakpoints (em-based for zoom resilience)

| Name | em value | px equivalent |
|------|----------|---------------|
| sm | 40em | 640px |
| md | 48em | 768px |
| lg | 64em | 1024px |
| xl | 80em | 1280px |
| 2xl | 96em | 1536px |

- Use `em` in media queries so they scale with browser font-size preferences, not root `font-size`
- Avoid pixel-perfect breakpoints tied to specific device models — base on content, not device

---

## Container Queries

Style a component based on its container's size, not the viewport. Enables truly reusable components.

```css
/* 1. Establish a containment context */
.card-wrapper {
  container-type: inline-size;
  container-name: card; /* optional — for named queries */
}

/* 2. Query the container */
@container card (width >= 480px) {
  .card {
    flex-direction: row;
  }
}

/* Short form without a name */
@container (width >= 480px) {
  .card { flex-direction: row; }
}
```

- `container-type: inline-size` — tracks width only (most common)
- `container-type: size` — tracks width and height (rare; has layout implications)
- Container units: `cqi` (1% container inline size), `cqb` (block), `cqw`, `cqh`, `cqmin`, `cqmax`

```css
/* Fluid font inside a container */
.card__title {
  font-size: clamp(1rem, 4cqi, 1.5rem);
}
```

---

## Fluid Typography with clamp()

`clamp(min, preferred, max)` — scales linearly between two viewport sizes.

```css
/* Formula: preferred = slope * 100vw + intercept */
/* From 16px at 320px viewport to 24px at 1280px viewport */
:root {
  --text-base: clamp(1rem, 0.5rem + 1.5625vw, 1.5rem);
  --text-lg:   clamp(1.125rem, 0.75rem + 1.5625vw, 1.75rem);
  --text-xl:   clamp(1.25rem, 0.875rem + 2vw, 2rem);
  --text-2xl:  clamp(1.5rem, 1rem + 3vw, 2.5rem);
}
```

- Never set a `clamp()` minimum below `1rem` for body text (accessibility)
- Use a [fluid type scale tool](https://utopia.fyi/) to generate precise values

---

## Responsive Units

| Unit | Based on | Use for |
|------|----------|---------|
| `rem` | Root font size | Font sizes, spacing scale |
| `em` | Parent font size | Component-relative spacing, media queries |
| `%` | Parent dimension | Widths, positioning |
| `vw` | Viewport width | Fluid widths, type scaling |
| `vh` | Viewport height | Hero sections (but has mobile URL bar issues) |
| `dvh` | Dynamic viewport height | Mobile full-screen layouts (URL bar aware) |
| `svh` | Small viewport height | Conservative mobile full-screen |
| `lvh` | Large viewport height | Desktop full-screen |
| `cqi` | Container inline size | Type inside container queries |

```css
/* Full-screen section that respects mobile chrome */
.hero {
  min-block-size: 100dvh;
}

/* Font size using rem for accessibility */
body {
  font-size: 1rem; /* 16px default, respects user preferences */
}
```

---

## Prefers-Color-Scheme (Accessibility)

```css
:root {
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #121212;
    --color-text: #e8e8e8;
  }
}
```

Prefer custom properties over duplicating rules; see [custom-properties.md](custom-properties.md) for full theming pattern.

---

## Prefers-Reduced-Motion

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

See [animations.md](animations.md) for per-component approach.

---

## Print Styles

```css
@media print {
  .no-print, nav, aside { display: none; }

  body {
    font-size: 12pt;
    color: #000;
    background: #fff;
  }

  a[href]::after {
    content: " (" attr(href) ")";
  }

  @page {
    margin: 2cm;
  }
}
```
