# Animations

## Transitions

Use transitions for state changes triggered by user interaction or class toggling.

```css
.btn {
  background-color: var(--color-primary);
  /* Specify properties explicitly — never use `all` */
  transition:
    background-color 150ms ease,
    box-shadow 150ms ease,
    transform 150ms ease;
}

.btn:hover {
  background-color: var(--color-primary-hover);
  box-shadow: 0 4px 12px hsl(0 0% 0% / 0.15);
}
```

- Avoid `transition: all` — it catches unexpected properties and hurts performance
- Keep durations short for interactions: 100–200ms for hover, 200–350ms for open/close
- Use `ease` or `ease-out` for elements entering; `ease-in` for elements leaving

---

## Keyframe Animations

```css
@keyframes fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}

@keyframes slide-up {
  from { transform: translateY(1rem); opacity: 0; }
  to   { transform: translateY(0);    opacity: 1; }
}

.modal {
  animation: slide-up 250ms ease-out both;
}
```

### animation shorthand order
```
name duration timing-function delay iteration-count direction fill-mode play-state
```

```css
.spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

## Performance — Animate Only Cheap Properties

The browser has three rendering stages: **layout → paint → composite**. Only composite-layer properties skip layout and paint entirely.

| Property | Cost | Notes |
|----------|------|-------|
| `transform` | Cheap (composite) | Translate, scale, rotate, skew |
| `opacity` | Cheap (composite) | |
| `filter` | Medium (paint) | Acceptable for simple blur |
| `clip-path` | Medium (paint) | |
| `background-color` | Medium (paint) | |
| `width`, `height` | Expensive (layout) | Use `transform: scale()` instead |
| `top`, `left`, `margin` | Expensive (layout) | Use `transform: translate()` instead |

```css
/* Prefer transform over positional properties */
.tooltip {
  transform: translateY(-0.5rem); /* NOT top: -8px */
}

/* Prefer opacity for fades */
.overlay {
  opacity: 0;
  transition: opacity 200ms;
}
```

---

## will-change

Signals to the browser to promote an element to its own composite layer ahead of time.

```css
/* Only add immediately before animation; remove after */
.animating {
  will-change: transform, opacity;
}
```

- Use sparingly — every promoted layer consumes GPU memory
- Never apply to large numbers of elements (e.g., all list items)
- Remove `will-change` when animation ends via JS if applied dynamically
- Prefer `transform: translateZ(0)` as a last resort hack; `will-change` is the correct tool

---

## Prefers-Reduced-Motion

Always provide a reduced-motion alternative. Vestibular disorders and epilepsy make motion harmful.

### Pattern 1 — media query wrapper
```css
@keyframes slide-in {
  from { transform: translateX(-100%); }
  to   { transform: translateX(0); }
}

.drawer {
  animation: slide-in 300ms ease-out;
}

@media (prefers-reduced-motion: reduce) {
  .drawer {
    animation: none;
    /* Provide a non-motion alternative if needed */
    transition: opacity 200ms;
  }
}
```

### Pattern 2 — opt-in motion (recommended)
```css
/* Default: no motion */
.card {
  opacity: 1;
}

/* Motion only for users who have not requested reduction */
@media (prefers-reduced-motion: no-preference) {
  .card {
    animation: fade-in 300ms ease-out both;
  }
}
```

Pattern 2 is safer — motion requires explicit opt-in rather than opt-out.

---

## Scroll-Driven Animations (2024)

Animate elements based on scroll position without JavaScript.

```css
/* Animate based on overall page scroll */
@keyframes grow-progress {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

.progress-bar {
  transform-origin: left;
  animation: grow-progress linear;
  animation-timeline: scroll(root);
}

/* Animate when element enters viewport */
@keyframes fade-up {
  from { opacity: 0; translate: 0 2rem; }
  to   { opacity: 1; translate: 0 0; }
}

.reveal {
  animation: fade-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 40%;
}
```

- `scroll()` — ties animation to a scrolling container
- `view()` — ties animation to an element entering/leaving the viewport
- Always pair with `prefers-reduced-motion: no-preference` guard

---

## View Transitions API (2024)

Animate between page states or DOM changes with the browser's built-in cross-fade.

```css
/* Default cross-fade between navigations (MPA) */
@view-transition {
  navigation: auto;
}

/* Custom per-element transition */
.hero-image {
  view-transition-name: hero;
}

::view-transition-old(hero),
::view-transition-new(hero) {
  animation-duration: 400ms;
}
```

For SPA-style transitions, trigger via JS: `document.startViewTransition(() => updateDOM())`.

---

## Common Animation Utilities

```css
@keyframes bounce-in {
  0%   { transform: scale(0.3); opacity: 0; }
  50%  { transform: scale(1.05); }
  70%  { transform: scale(0.9); }
  100% { transform: scale(1); opacity: 1; }
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.5; }
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-6px); }
  40%, 80% { transform: translateX(6px); }
}
```
