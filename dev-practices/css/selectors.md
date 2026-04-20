# Selectors

## Specificity Rules

Specificity is calculated as `(id, class, element)` — higher wins; same specificity = last declared wins.

| Selector type | Specificity |
|---------------|-------------|
| `#id` | (1, 0, 0) |
| `.class`, `[attr]`, `:pseudo-class` | (0, 1, 0) |
| `element`, `::pseudo-element` | (0, 0, 1) |
| `:is()`, `:not()`, `:has()` | Takes highest specificity of argument |
| `:where()` | (0, 0, 0) — always zero |
| Inline style | (1, 0, 0, 0) — beats everything |
| `!important` | Overrides all; avoid |

- Aim to keep most selectors at (0, 1, 0) — one class
- Avoid ID selectors in stylesheets; use classes instead
- Never use `!important` in component styles; it belongs only in utility overrides or `@layer` reset

---

## BEM Naming Convention

```
Block__Element--Modifier
```

```css
/* Block */
.card { }

/* Element — part of the block */
.card__title { }
.card__body  { }
.card__image { }

/* Modifier — variant of block or element */
.card--featured   { }
.card__title--large { }
```

Rules:
- Elements are never nested deeper than one level in the name (`card__body__text` is wrong — use `card__text`)
- Modifiers add visual/state variation; they are never used alone
- JavaScript hooks use a `js-` prefix so they stay separate from styling: `js-modal-trigger`

---

## Native CSS Nesting

Available natively (no preprocessor required) since 2023.

```css
.card {
  padding: 1rem;
  border-radius: 0.5rem;

  /* Element */
  .card__title {
    font-size: 1.25rem;
  }

  /* Pseudo-class on the block itself — use & */
  &:hover {
    box-shadow: 0 4px 12px hsl(0 0% 0% / 0.1);
  }

  /* State modifier */
  &.card--featured {
    border: 2px solid var(--color-accent);
  }

  /* Media query inside rule */
  @media (width >= 768px) {
    padding: 1.5rem;
  }
}
```

- `&` refers to the parent selector — required when combining with a pseudo-class or modifier
- Avoid deep nesting (more than 2–3 levels); it reproduces specificity problems
- Nesting `@media` / `@container` inside a rule is supported and keeps related styles together

---

## :is() — Grouping Without Repetition

Matches if the element matches any selector in the list. Specificity = highest in list.

```css
/* Instead of: h1 a, h2 a, h3 a */
:is(h1, h2, h3) a {
  color: var(--color-link);
}

/* Useful for state variants */
:is(.btn:hover, .btn:focus-visible) {
  background-color: var(--color-btn-hover);
}
```

---

## :where() — Zero-Specificity Grouping

Identical to `:is()` but always contributes **zero** specificity — safe for resets and base layers.

```css
/* Reset that is trivially overridable */
:where(h1, h2, h3, h4) {
  margin-block: 0;
  line-height: 1.2;
}
```

---

## :has() — Parent / Relational Selector

Selects an element if it contains (or is followed by) a matching descendant.

```css
/* Card with an image gets extra padding */
.card:has(img) {
  padding-block-start: 0;
}

/* Form field invalid state — style label, not just input */
.field:has(input:invalid) .field__label {
  color: var(--color-error);
}

/* Navigation item that is current */
.nav__item:has(> [aria-current="page"]) {
  border-inline-start: 3px solid var(--color-accent);
}

/* Two-column layout only when sidebar is present */
.layout:has(.sidebar) {
  grid-template-columns: 240px 1fr;
}
```

---

## :not() — Exclusion

```css
/* All list items except the last */
.list__item:not(:last-child) {
  border-block-end: 1px solid var(--color-border);
}

/* Buttons that are not disabled */
.btn:not(:disabled):hover {
  background-color: var(--color-btn-hover);
}

/* Multiple exclusions (selector list accepted) */
p:not(.intro, .footnote) {
  text-indent: 1.5em;
}
```

---

## Attribute Selectors

```css
[href^="https"] { /* starts with */  }
[href$=".pdf"]  { /* ends with */    }
[href*="cdn"]   { /* contains */     }
[lang|="en"]    { /* en or en-US */  }
[class~="icon"] { /* space-sep list contains */ }
```

- Attribute selectors have class-level (0, 1, 0) specificity
- Useful for styling external links, file-type icons without extra classes

---

## Selector Performance Tips

- Avoid universal selector (`*`) in deeply nested contexts
- Keep selectors short — the browser reads right to left
- Expensive: `[class*="col-"]`, `:nth-child()` on large lists, `*` descendant combinator
- Prefer direct child `>` over descendant ` ` when possible
