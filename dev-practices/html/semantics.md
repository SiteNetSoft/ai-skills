# Semantic HTML

## Document Structure

Every page needs a clear top-level structure:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Page Title</title>
  </head>
  <body>
    <header>...</header>
    <nav>...</nav>
    <main>
      <article> or <section> content
    </main>
    <aside>...</aside>
    <footer>...</footer>
  </body>
</html>
```

- `<main>` must appear **once** per page and must not be nested inside `<article>`, `<aside>`, `<header>`, `<footer>`, or `<nav>`
- `<!DOCTYPE html>` is mandatory — omitting it triggers quirks mode
- Always set `lang` on `<html>`; use a BCP 47 tag (e.g., `lang="en"`, `lang="fr-CA"`)

## Landmark Elements

| Element | Role | Use for |
|---------|------|---------|
| `<header>` | `banner` (top-level) / `none` (nested) | Site header or section header |
| `<nav>` | `navigation` | Primary or secondary navigation links |
| `<main>` | `main` | The dominant content of the page |
| `<article>` | `article` | Self-contained, independently distributable content |
| `<section>` | `region` (if named) | Thematic grouping of content with a heading |
| `<aside>` | `complementary` | Content tangentially related to the surrounding content |
| `<footer>` | `contentinfo` (top-level) / `none` (nested) | Footer for page or section |

- A `<section>` without a heading (`<h1>`–`<h6>`) provides no meaningful landmark; add one or use `<div>`
- Multiple `<nav>` elements are fine — use `aria-label` to distinguish them: `<nav aria-label="Primary">`

## Heading Hierarchy

- Use `<h1>`–`<h6>` to express document outline, **not** for visual sizing
- One `<h1>` per page (the page title or main topic)
- Do not skip levels: `<h1>` → `<h2>` → `<h3>`, never `<h1>` → `<h3>`
- Headings inside `<article>` restart their own hierarchy

```html
<!-- Good -->
<main>
  <h1>Guide to Composting</h1>
  <section>
    <h2>Getting Started</h2>
    <h3>Choosing a Bin</h3>
  </section>
</main>

<!-- Bad — skips h2 -->
<main>
  <h1>Guide to Composting</h1>
  <h3>Choosing a Bin</h3>
</main>
```

## Inline Semantic Elements

| Element | Meaning | Not for |
|---------|---------|---------|
| `<strong>` | Strong importance | Visual bold only — use CSS for that |
| `<em>` | Stress emphasis | Visual italic only |
| `<time datetime="2025-01-15">` | Machine-readable date/time | — |
| `<abbr title="...">` | Abbreviation | — |
| `<code>` | Inline code | Blocks of code (use `<pre><code>`) |
| `<mark>` | Highlighted/relevant text | Decorative highlighting |
| `<cite>` | Title of a creative work | Author names |
| `<q>` | Short inline quotation | Long quotes (use `<blockquote>`) |
| `<del>` / `<ins>` | Removed / inserted content | Strikethrough styling |

## Lists

- `<ul>` — unordered items (order does not matter)
- `<ol>` — ordered/sequential items (steps, rankings)
- `<dl>` / `<dt>` / `<dd>` — term/description pairs (glossaries, metadata)
- Never use `<ul>` or `<ol>` purely for layout; that is CSS's job

## When to Use `<div>` and `<span>`

Use `<div>` and `<span>` **only** when no semantic element fits:

- Layout containers that carry no meaning
- JS hooks where no semantic element applies
- Styling wrappers required by a CSS technique (e.g., a scroll container)

```html
<!-- Bad — div soup -->
<div class="header">
  <div class="nav">...</div>
</div>

<!-- Good -->
<header>
  <nav>...</nav>
</header>
```

## Tables

Use `<table>` for **tabular data** only — never for layout.

```html
<table>
  <caption>Monthly Sales by Region</caption>
  <thead>
    <tr>
      <th scope="col">Region</th>
      <th scope="col">Jan</th>
      <th scope="col">Feb</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">North</th>
      <td>120</td>
      <td>95</td>
    </tr>
  </tbody>
</table>
```

- Always include `<caption>` to describe the table
- Use `scope="col"` on column headers, `scope="row"` on row headers
- Use `<thead>`, `<tbody>`, and optionally `<tfoot>` for structure
