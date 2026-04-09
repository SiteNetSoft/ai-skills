# Configuration (mkdocs.yml)

## Minimal Starter

```yaml
site_name: Project Name
site_url: https://example.com
site_description: One-line description
site_author: Organization

repo_url: https://github.com/org/repo
repo_name: org/repo

copyright: Copyright &copy; 2026 Organization

theme:
  name: material
  custom_dir: docs/overrides
  logo: assets/logo.svg
  favicon: assets/logo.svg
  palette:
    - media: "(prefers-color-scheme: light)"
      scheme: default
      primary: custom
      accent: teal
      toggle:
        icon: material/brightness-7
        name: Switch to dark mode
    - media: "(prefers-color-scheme: dark)"
      scheme: slate
      primary: custom
      accent: teal
      toggle:
        icon: material/brightness-4
        name: Switch to light mode
  features:
    - content.code.copy
    - content.code.annotate
    - navigation.tabs
    - navigation.sections
    - navigation.top
    - navigation.footer
    - search.suggest
    - search.highlight
    - toc.follow

extra_css:
  - assets/extra.css

plugins:
  - search
  - glightbox
  - meta

markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
  - pymdownx.highlight:
      anchor_linenums: true
      line_spans: __span
      pygments_lang_class: true
  - pymdownx.inlinehilite
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.snippets
  - tables
  - toc:
      permalink: true
  - attr_list
  - md_in_html
  - def_list
  - footnotes
```

## Navigation

Define navigation explicitly in `mkdocs.yml` — don't rely on auto-discovery:

```yaml
nav:
  - Home: index.md
  - Section Name:
      - Overview: section/overview.md
      - Topic A: section/topic-a.md
      - Topic B: section/topic-b.md
```

- Every section starts with an overview page
- Order pages logically (general to specific, not alphabetically)
- Use short, descriptive labels — users scan nav, not read it
- Nest at most 2 levels deep in the nav tree

## Theme Features to Enable

| Feature | Purpose |
|---------|---------|
| `content.code.copy` | Copy button on code blocks |
| `content.code.annotate` | Numbered annotations in code |
| `navigation.tabs` | Horizontal top-level nav |
| `navigation.sections` | Collapsible sidebar sections |
| `navigation.top` | "Back to top" button |
| `navigation.footer` | Previous/next page links |
| `search.suggest` | Search autocomplete |
| `search.highlight` | Highlight matches in results |
| `toc.follow` | Scroll-aware table of contents |

## Markdown Extensions to Enable

| Extension | Purpose |
|-----------|---------|
| `admonition` | Info/warning/note/tip boxes |
| `pymdownx.details` | Collapsible sections |
| `pymdownx.superfences` | Fenced code blocks with language tags |
| `pymdownx.highlight` | Syntax highlighting with Pygments |
| `pymdownx.tabbed` | Tab groups for multi-language examples |
| `pymdownx.snippets` | Include content from other files |
| `tables` | Markdown tables |
| `toc` | Table of contents with permalink anchors |
| `footnotes` | Footnote references |
| `def_list` | Definition lists |
| `attr_list` | HTML attributes on Markdown elements |
