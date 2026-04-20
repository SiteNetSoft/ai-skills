# Accessibility

## First Rule of ARIA

Before reaching for ARIA, use a native HTML element. Native elements have built-in
roles, states, keyboard behavior, and browser/AT support — ARIA only patches the gap.

> "No ARIA is better than bad ARIA." — WAI-ARIA Authoring Practices

```html
<!-- Bad: ARIA div button -->
<div role="button" tabindex="0" onclick="...">Click me</div>

<!-- Good: native button -->
<button type="button">Click me</button>
```

## ARIA Roles

Use `role` only when there is no native HTML equivalent:

| When to use | Example |
|------------|---------|
| Custom widget with no HTML equivalent | `role="tablist"`, `role="tab"`, `role="tabpanel"` |
| Landmark override needed | `role="search"` on a `<form>` |
| Non-semantic container needs region | `role="region"` + `aria-label` on a `<section>` that lacks a heading |

Do **not** use `role` to override an existing semantic element:
```html
<!-- Wrong — adds noise, does nothing useful -->
<button role="button">Save</button>
<nav role="navigation">...</nav>
```

## Essential ARIA Attributes

| Attribute | Purpose |
|-----------|---------|
| `aria-label` | Overrides accessible name (use when visible text is absent) |
| `aria-labelledby` | Points to element(s) that provide the accessible name |
| `aria-describedby` | Points to supplementary description (hints, errors) |
| `aria-hidden="true"` | Removes element from accessibility tree (decorative icons, duplicates) |
| `aria-expanded` | State of a toggle/disclosure widget |
| `aria-controls` | References the element controlled by this widget |
| `aria-live` | Announces dynamic content changes (`polite` or `assertive`) |
| `aria-invalid` | Marks invalid form fields |
| `aria-required` | Marks required fields (prefer native `required` attribute) |
| `aria-current` | Identifies the current item in a set (e.g., current page in `<nav>`) |
| `aria-disabled` | Communicates disabled state while keeping element focusable |

## Keyboard Navigation

All interactive elements must be reachable and operable via keyboard:

- Native elements (`<a>`, `<button>`, `<input>`, `<select>`, `<textarea>`) are keyboard-accessible by default — keep them
- `tabindex="0"` — adds a custom element to the tab order at its natural document position
- `tabindex="-1"` — removes from tab order but allows programmatic focus via `.focus()`
- **Never** use `tabindex` values greater than 0 — they break the natural focus order

### Keyboard Patterns for Common Widgets

| Widget | Enter/Space | Arrow keys | Escape |
|--------|------------|-----------|--------|
| Button | Activate | — | — |
| Link | Follow | — | — |
| Checkbox | Toggle | — | — |
| Listbox | Select | Move focus | Close |
| Tabs | Activate tab | Move focus between tabs | — |
| Dialog | Confirm | — | Close |
| Combobox | Open/select | Navigate options | Close |

## Focus Management

- Visible focus indicator is **required** — never `outline: none` without a replacement style
- On modal open: move focus to the first interactive element inside the modal
- On modal close: return focus to the element that opened it
- On route change (SPA): move focus to the new page heading or a skip target

```html
<!-- Skip link — first focusable element on the page -->
<a href="#main-content" class="skip-link">Skip to main content</a>
...
<main id="main-content" tabindex="-1">...</main>
```

```css
/* Show skip link only on focus */
.skip-link {
  position: absolute;
  transform: translateY(-100%);
}
.skip-link:focus {
  transform: translateY(0);
}
```

## Images and Alt Text

```html
<!-- Informative image -->
<img src="chart.png" alt="Bar chart showing 40% increase in sales from Q1 to Q2 2025">

<!-- Decorative image — empty alt, no role -->
<img src="divider.png" alt="">

<!-- Functional image (link/button) — describe the action, not the image -->
<a href="/home"><img src="logo.png" alt="Acme Corp — Home"></a>

<!-- Complex image — short alt + long description nearby -->
<img src="org-chart.png" alt="Organisation chart" aria-describedby="org-desc">
<p id="org-desc">The chart shows three tiers: CEO at top, VPs below, and managers at the base.</p>
```

Rules:
- Every `<img>` needs an `alt` attribute — omitting it forces screen readers to announce the filename
- Empty `alt=""` for decorative images — do not write "decorative image" or "spacer"
- Describe **content and function**, not appearance ("red arrow pointing right")
- Icons inside buttons: hide with `aria-hidden="true"`, provide accessible name on the button

## Color and Contrast

| Ratio | Requirement |
|-------|-------------|
| 4.5:1 | Normal text (WCAG AA) |
| 3:1 | Large text ≥18pt or ≥14pt bold (WCAG AA) |
| 3:1 | UI components and graphical objects (WCAG AA) |
| 7:1 | Normal text (WCAG AAA) |

- Never convey information by color alone — use shape, pattern, or text as well
- Ensure focus indicators meet the 3:1 contrast ratio against adjacent colors

## Screen Reader Patterns

- Use `aria-live="polite"` for non-urgent dynamic updates (search results, filtered lists)
- Use `aria-live="assertive"` only for critical alerts — it interrupts the user
- `role="status"` is a polite live region; `role="alert"` is assertive
- Hidden visually but available to screen readers:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```html
<!-- Icon button with screen-reader text -->
<button type="button" aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

- Set `focusable="false"` on inline SVG to prevent IE/Edge from adding them to the tab order

## Language

- `lang` on `<html>` for the page language
- `lang` on any inline element that switches language:

```html
<p>The French word for hello is <span lang="fr">bonjour</span>.</p>
```
