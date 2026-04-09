# Website Documentation Best Practices

Guidelines for building and maintaining documentation websites using MkDocs with Material
theme. This skill is split into focused sub-files — read only what you need for the task.

## Stack

- **Generator**: [MkDocs](https://www.mkdocs.org/) (Python-based, Markdown-first)
- **Theme**: [Material for MkDocs](https://squidfunnel.github.io/mkdocs-material/) (responsive, feature-rich)
- **Hosting**: GitHub Pages (via `actions/deploy-pages`)
- **Diagrams**: [PlantUML](https://plantuml.com/) with [C4 model](https://c4model.com/)

## Sub-Files

| File | When to read |
|------|-------------|
| [structure.md](structure.md) | Setting up a new docs project, directory layout, .gitignore |
| [config.md](config.md) | Configuring mkdocs.yml — theme, nav, features, extensions |
| [styling.md](styling.md) | Brand colors, hero sections, feature grids, dark mode CSS |
| [cicd.md](cicd.md) | GitHub Actions workflow, Makefile, requirements.txt |
| [content.md](content.md) | Writing pages — frontmatter, homepage, admonitions, tabs, cross-references |
| [plantuml.md](plantuml.md) | Architecture diagrams — C4 model, .puml examples, rendering |
| [quality.md](quality.md) | Pre-merge checklist and key principles |

## Key Principles

1. **Content-first** — Markdown files are the source of truth, not HTML templates
2. **Explicit navigation** — define nav in `mkdocs.yml`, don't rely on directory auto-discovery
3. **Strict builds** — always use `--strict` to catch issues before deploy
4. **Minimal dependencies** — 3 pip packages; add plugins only when they solve a real problem
5. **Diagram-as-code** — version-controlled diagram source, not opaque image files
6. **One topic per page** — keep pages focused and linkable
7. **Dark mode aware** — test styling in both light and dark schemes
