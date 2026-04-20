# CSS Best Practices

Guidelines for writing modern, maintainable, and performant CSS. Focused on native CSS (2024+)
without preprocessor dependency.

## Sub-Files

| File | When to read |
|------|-------------|
| [layout.md](layout.md) | Flexbox, Grid, subgrid, logical properties, layout recipes |
| [selectors.md](selectors.md) | Specificity, BEM naming, nesting, :is(), :where(), :has() |
| [responsive.md](responsive.md) | Media queries, container queries, fluid typography, mobile-first |
| [custom-properties.md](custom-properties.md) | CSS variables, theming, dark mode, design tokens |
| [animations.md](animations.md) | Transitions, keyframes, performance, prefers-reduced-motion |
| [architecture.md](architecture.md) | File organization, CUBE CSS, cascade layers, @scope |
| [performance.md](performance.md) | Critical CSS, content-visibility, contain, font loading |

## Key Principles

1. **Cascade is a feature** — understand and use it; don't fight it with high specificity
2. **Native first** — use modern CSS before reaching for a preprocessor or JS
3. **Mobile-first** — base styles serve small screens; media queries add complexity upward
4. **Custom properties over magic numbers** — every repeated value belongs in a variable
5. **Layout separates from cosmetics** — structure (grid/flex) lives apart from color and type
6. **Accessible by default** — respect `prefers-reduced-motion`, `prefers-color-scheme`, `focus-visible`
7. **Specificity stays low** — prefer class selectors; use `@layer` to manage cascade order

## Sources

- [MDN Web Docs — CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [web.dev — Learn CSS](https://web.dev/learn/css)
- [Every Layout](https://every-layout.dev/)
- [CUBE CSS Methodology](https://cube.fyi/)
- [CSS Tricks — Complete Guides](https://css-tricks.com/guides/)
- [Smashing Magazine — CSS](https://www.smashingmagazine.com/category/css/)
- [Chrome for Developers — CSS](https://developer.chrome.com/docs/css-ui)
