# Custom Properties (CSS Variables)

## Basics

```css
/* Define on :root for global scope */
:root {
  --color-primary: #2563eb;
  --spacing-md: 1rem;
  --radius-sm: 0.25rem;
}

/* Use anywhere */
.btn {
  background-color: var(--color-primary);
  padding: var(--spacing-md);
  border-radius: var(--radius-sm);
}
```

- Custom properties cascade and inherit like any other property
- They are case-sensitive: `--color` and `--Color` are different
- They can hold any valid CSS value — lengths, colors, strings, even partial values

---

## Fallbacks

```css
/* Fallback if --color-accent is not defined */
color: var(--color-accent, #e11d48);

/* Chain fallbacks */
color: var(--color-brand, var(--color-primary, #2563eb));

/* Fallback to another property */
gap: var(--grid-gap, 1rem);
```

- Use fallbacks for optional overrides that consumers may or may not set
- Do not use fallbacks to mask missing definitions in your own system — define the variable

---

## Design Token System

Organize variables in three layers: primitive → semantic → component.

```css
:root {
  /* 1. Primitive tokens — raw values */
  --blue-500: #3b82f6;
  --blue-600: #2563eb;
  --gray-900: #111827;
  --gray-100: #f3f4f6;
  --space-4: 1rem;
  --space-6: 1.5rem;

  /* 2. Semantic tokens — intent, not value */
  --color-bg:         var(--gray-100);
  --color-text:       var(--gray-900);
  --color-primary:    var(--blue-600);
  --color-primary-hover: var(--blue-500);

  /* 3. Component tokens — scoped defaults */
  --btn-bg:           var(--color-primary);
  --btn-bg-hover:     var(--color-primary-hover);
  --btn-padding:      0.5em 1.25em;
}
```

Benefits: swapping a theme only changes semantic tokens; primitives and components remain stable.

---

## Theming and Dark Mode

### Approach 1 — Media query (automatic)

```css
:root {
  --color-bg:   #ffffff;
  --color-text: #111827;
  --color-surface: #f9fafb;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg:   #0f172a;
    --color-text: #f1f5f9;
    --color-surface: #1e293b;
  }
}
```

### Approach 2 — Data attribute (user toggle)

```css
:root,
[data-theme="light"] {
  --color-bg:   #ffffff;
  --color-text: #111827;
}

[data-theme="dark"] {
  --color-bg:   #0f172a;
  --color-text: #f1f5f9;
}
```

Toggle in JS: `document.documentElement.dataset.theme = 'dark'`

### Approach 3 — light-dark() function (2024)

```css
:root {
  color-scheme: light dark;
  --color-bg:   light-dark(#ffffff, #0f172a);
  --color-text: light-dark(#111827, #f1f5f9);
}
```

- `light-dark()` is the most concise; requires `color-scheme` on the containing element
- `color-mix()` can generate hover/active states without extra tokens:

```css
--color-primary-hover: color-mix(in srgb, var(--color-primary) 85%, black);
```

---

## Scoping Variables

Custom properties inherit down the tree — scope overrides to a subtree:

```css
/* Default card appearance */
.card {
  --card-bg: var(--color-surface);
  --card-border: 1px solid var(--color-border);
  background: var(--card-bg);
  border: var(--card-border);
}

/* Featured variant — redefine tokens locally */
.card--featured {
  --card-bg: var(--color-primary);
  --card-border: none;
  color: #fff;
}
```

---

## Computed / Dynamic Values

```css
/* Spacing scale from a single base */
:root {
  --space-unit: 0.25rem;
  --space-1: calc(var(--space-unit) * 1);  /* 4px */
  --space-2: calc(var(--space-unit) * 2);  /* 8px */
  --space-4: calc(var(--space-unit) * 4);  /* 16px */
  --space-8: calc(var(--space-unit) * 8);  /* 32px */
}

/* Animated value with @property — enables transitions on custom props */
@property --progress {
  syntax: "<number>";
  initial-value: 0;
  inherits: false;
}

.progress-bar {
  --progress: 0;
  width: calc(var(--progress) * 100%);
  transition: --progress 0.4s ease;
}
```

- `@property` registers a typed custom property — enables CSS transitions on custom properties
- Without `@property`, custom properties cannot be transitioned/animated

---

## Common Pitfalls

- An invalid `var()` resolves to the property's initial value, not zero — silence can hide bugs
- Custom properties set to empty string (`--foo: ;`) are valid but may cause unexpected behavior
- Avoid putting vendor-prefixed values in custom properties; use `@supports` instead
- Do not use custom properties for values that must be static at parse time (e.g., media query values — use `env()` or fixed values there)
