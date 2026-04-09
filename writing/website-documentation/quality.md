# Quality Checklist

Before merging documentation changes:

1. `mkdocs build --strict` passes with no warnings
2. All internal links resolve (strict mode catches this)
3. Every new page is added to `nav:` in `mkdocs.yml`
4. Code blocks have language tags
5. Images have alt text
6. No orphan pages (pages not reachable from nav)
7. Frontmatter `description` is set on every page
8. Local dev server (`mkdocs serve`) renders correctly

## Sources

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunnel.github.io/mkdocs-material/)
- [PyMdown Extensions](https://facelessuser.github.io/pymdown-extensions/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
