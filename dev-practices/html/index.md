# HTML Best Practices

Guidelines for writing modern, semantic, accessible, and performant HTML. Based on
the HTML Living Standard, WCAG 2.1, and MDN Web Docs.

## Sub-Files

| File | When to read |
|------|-------------|
| [semantics.md](semantics.md) | Semantic elements, document outline, divs vs semantic elements |
| [forms.md](forms.md) | Form structure, input types, labels, validation, fieldset/legend |
| [accessibility.md](accessibility.md) | ARIA, keyboard navigation, focus management, screen readers, alt text |
| [media.md](media.md) | Responsive images, lazy loading, video/audio, modern formats |
| [performance.md](performance.md) | Resource loading, critical rendering path, Core Web Vitals, minimal markup |
| [meta.md](meta.md) | Meta tags, Open Graph, JSON-LD, favicon, viewport, SEO |

## Key Principles

1. **Semantic first** — choose the element whose meaning matches the content; reach for `<div>` only when no semantic element fits
2. **Accessibility by default** — write for keyboard, screen reader, and pointer users equally; WCAG 2.1 AA is the floor
3. **Progressive enhancement** — start with valid, meaningful HTML; layer CSS and JS on top
4. **Minimal markup** — every element must earn its place; avoid wrapper sprawl
5. **Separate concerns** — structure in HTML, presentation in CSS, behavior in JS; no inline styles or event attributes
6. **Performance matters** — loading order, image formats, and resource hints have direct user impact
7. **Valid HTML** — run pages through the W3C validator; invalid markup causes unpredictable parser behavior

## Sources

- [HTML Living Standard](https://html.spec.whatwg.org/)
- [MDN HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [WCAG 2.1 Guidelines](https://www.w3.org/TR/WCAG21/)
- [W3C Validator](https://validator.w3.org/)
- [web.dev Learn HTML](https://web.dev/learn/html)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
