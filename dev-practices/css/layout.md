# Layout

## Flexbox vs Grid — When to Use Which

| Need | Use |
|------|-----|
| One-dimensional row or column | Flexbox |
| Two-dimensional rows and columns | Grid |
| Content-driven sizing (items control layout) | Flexbox |
| Layout-driven sizing (container controls items) | Grid |
| Centering a single element | Either (Grid is terser) |
| Navigation bar / toolbar | Flexbox |
| Page/section macro layout | Grid |
| Card grids, galleries | Grid |
| Wrapping lists of chips/tags | Flexbox with `flex-wrap` |

---

## Flexbox Patterns

### Core setup
```css
.flex-row {
  display: flex;
  gap: 1rem; /* prefer gap over margins between flex children */
}
```

### Space between with last-item flush
```css
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

### Flex children — grow/shrink rules
```css
/* Item takes all available space */
.flex-grow { flex: 1; }

/* Fixed size, no shrink */
.flex-fixed { flex: 0 0 200px; }

/* Minimum size, can grow */
.flex-min { flex: 1 1 auto; min-width: 0; } /* min-width:0 fixes overflow bug */
```

### Centering
```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

---

## Grid Patterns

### Auto-responsive columns (no media query needed)
```css
.grid-auto {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 280px), 1fr));
  gap: 1.5rem;
}
```

### Named areas — macro page layout
```css
.page {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  grid-template-columns: 240px 1fr;
  grid-template-rows: auto 1fr auto;
  min-height: 100dvh;
}

.page-header  { grid-area: header; }
.page-sidebar { grid-area: sidebar; }
.page-main    { grid-area: main; }
.page-footer  { grid-area: footer; }
```

### Subgrid — align nested items to parent tracks
```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

/* Card rows align across siblings */
.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* inherits parent row tracks */
}
```

### Full-bleed sections inside a constrained layout
```css
.content-grid {
  display: grid;
  grid-template-columns:
    [full-start] minmax(1rem, 1fr)
    [content-start] min(100% - 2rem, 72ch)
    [content-end] minmax(1rem, 1fr)
    [full-end];
}

.content-grid > * { grid-column: content; }
.content-grid > .full-bleed { grid-column: full; }
```

---

## Logical Properties

Use logical properties so layouts adapt to writing direction without extra overrides.

| Physical | Logical |
|----------|---------|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `margin-top` | `margin-block-start` |
| `padding-top / bottom` | `padding-block` |
| `padding-left / right` | `padding-inline` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `border-top` | `border-block-start` |

```css
/* Prefer logical shorthand */
.box {
  margin-block: 1.5rem;
  padding-inline: 1rem;
  border-inline-start: 3px solid var(--color-accent);
}
```

---

## Common Layout Recipes

### Stack (vertical rhythm)
```css
.stack > * + * {
  margin-block-start: var(--stack-gap, 1rem);
}
```

### Sidebar layout (flex, sidebar + fluid main)
```css
.with-sidebar {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.with-sidebar > :first-child { /* sidebar */
  flex-basis: 240px;
  flex-grow: 1;
}

.with-sidebar > :last-child { /* main */
  flex-basis: 0;
  flex-grow: 999;
  min-inline-size: 60%;
}
```

### Cover (full-viewport hero)
```css
.cover {
  display: flex;
  flex-direction: column;
  min-block-size: 100dvh;
  padding: 1rem;
}

.cover > :first-child { margin-block-end: auto; }
.cover > :last-child  { margin-block-start: auto; }
/* Middle element is centered vertically */
```
