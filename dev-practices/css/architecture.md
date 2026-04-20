# Architecture

## File Organization

```
styles/
├── base/
│   ├── reset.css          # normalize / modern reset
│   ├── typography.css     # font-face, body type scale
│   └── tokens.css         # design tokens (:root custom properties)
├── layout/
│   ├── grid.css           # page-level grid
│   └── stack.css          # reusable layout primitives
├── components/
│   ├── button.css
│   ├── card.css
│   └── modal.css
├── utilities/
│   └── utils.css          # single-purpose classes (sr-only, truncate)
└── main.css               # imports everything via @import or @layer
```

- Keep one concern per file
- Components are the most numerous — each gets its own file
- Utilities live separately and are almost always low-specificity one-liners

---

## Cascade Layers (@layer)

`@layer` gives explicit control over cascade order, eliminating specificity wars.

```css
/* main.css — declare layer order first */
@layer reset, tokens, base, layout, components, utilities;

@import "base/reset.css"      layer(reset);
@import "base/tokens.css"     layer(tokens);
@import "base/typography.css" layer(base);
@import "layout/grid.css"     layer(layout);
@import "components/card.css" layer(components);
@import "utilities/utils.css" layer(utilities);
```

- Layers declared later win over earlier layers, regardless of specificity
- Unlayered styles beat all layers — third-party CSS without `@layer` can override yours; wrap it:

```css
@import url("vendor/library.css") layer(vendor);
@layer reset, vendor, base, components, utilities;
```

- Override a layer from a higher layer — no `!important` needed:

```css
@layer components {
  .btn { padding: 0.5em 1em; }
}

/* utilities layer is declared after components — wins without !important */
@layer utilities {
  .p-0 { padding: 0; }
}
```

---

## CUBE CSS Methodology

**C**omposition · **U**tility · **B**lock · **E**xception

```css
/* Composition — layout primitives, no cosmetics */
.cluster { display: flex; flex-wrap: wrap; gap: var(--space-sm); }
.stack   { display: grid; gap: var(--space-md); }

/* Utility — single-purpose, design-token-driven */
.color-primary { color: var(--color-primary); }
.font-bold     { font-weight: 700; }

/* Block — component with internal structure */
.card { background: var(--color-surface); border-radius: var(--radius-md); }
.card__title { font-size: var(--text-lg); }

/* Exception — data-attribute variant */
.card[data-variant="featured"] { background: var(--color-primary); color: #fff; }
```

CUBE avoids deep nesting and high specificity by keeping each layer at its correct abstraction.

---

## BEM (Block Element Modifier)

See [selectors.md](selectors.md) for naming rules. BEM pairs well with `@layer components`:

```css
@layer components {
  .card { }
  .card__title { }
  .card--featured { }
}
```

---

## @scope — Style Boundaries (2024)

Limit styles to a subtree without relying on high-specificity selectors.

```css
/* Styles only apply inside .card, not outside */
@scope (.card) {
  h2 { font-size: 1.25rem; }
  p  { color: var(--color-muted); }
}

/* Scope with a lower boundary — stop before .card__footer */
@scope (.card) to (.card__footer) {
  p { margin-block-end: 0.5rem; }
}
```

- `:scope` pseudo-class inside `@scope` refers to the scope root
- Lower boundary excludes matching elements from the scope
- Useful for component isolation without Shadow DOM

---

## Avoiding Specificity Wars

| Problem | Fix |
|---------|-----|
| Overriding a third-party style | Wrap vendor CSS in a `@layer`; your unlayered rules win |
| Component variant overrides | Use data attributes + `@scope`; keep same specificity |
| State styles fighting base | Write state as a modifier class or use `:is()` to merge |
| Utility class overridden by component | Put utilities in a higher layer than components |
| `!important` creep | Limit `!important` to a `@layer utilities` only |

```css
/* Explicit layer order resolves 90% of specificity problems */
@layer reset, base, layout, components, utilities;
```

---

## Practical @import Pattern (with layers)

```css
/* main.css */
@layer reset, tokens, base, layout, components, utilities;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  :where(h1, h2, h3, h4, p, ul, ol) { margin: 0; }
}

@layer tokens {
  :root {
    --color-primary: #2563eb;
    --space-md: 1rem;
  }
}

@layer components {
  @import url("components/button.css");
  @import url("components/card.css");
}

@layer utilities {
  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0 0 0 0);
    white-space: nowrap;
  }
  .truncate {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
}
```
