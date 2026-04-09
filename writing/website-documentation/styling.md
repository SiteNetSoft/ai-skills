# Custom Styling

## Brand Colors (extra.css)

Override Material's CSS custom properties for branding:

```css
:root {
  --md-primary-fg-color: #2c3840;
  --md-primary-fg-color--light: #3d4f5a;
  --md-primary-fg-color--dark: #1e2a30;
  --md-accent-fg-color: #26a69a;
}
```

## Landing Page Components

Use CSS classes for hero sections, feature grids, and quick-link buttons:

```css
/* Hero section */
.md-typeset .hero {
  text-align: center;
  padding: 2rem 1rem;
}

/* Feature grid — auto-wrap, responsive */
.md-typeset .features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin: 2rem 0;
}

/* Quick-link buttons */
.md-typeset .links {
  display: flex;
  gap: 1rem;
  justify-content: center;
  flex-wrap: wrap;
  margin: 2rem 0;
}

.md-typeset .links a.primary {
  background: var(--md-primary-fg-color);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: 0.25rem;
  font-weight: 600;
  text-decoration: none;
}

.md-typeset .links a.secondary {
  border: 2px solid var(--md-primary-fg-color);
  color: var(--md-primary-fg-color);
  padding: 0.75rem 1.5rem;
  border-radius: 0.25rem;
  font-weight: 600;
  text-decoration: none;
}

/* Dark mode override */
[data-md-color-scheme="slate"] .md-typeset .links a.secondary {
  border-color: var(--md-accent-fg-color);
  color: var(--md-accent-fg-color);
}
```

## Template Overrides

For minimal customization, create `docs/overrides/main.html`:

```html
{% extends "base.html" %}
```

Only add blocks when you need to inject custom content (analytics, banners, etc.).
