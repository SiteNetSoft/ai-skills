# Writing Content

## Page Frontmatter

Every page should have a `description` for search and SEO:

```yaml
---
description: Brief description of what this page covers.
---
```

## Homepage (index.md)

Structure the homepage to orient new visitors:

1. **Hero** — project name, one-line tagline, logo
2. **What it is** — 2-3 sentence explanation
3. **Key differentiator** — what makes this different (use an admonition)
4. **Analogy or quick start** — make the concept accessible
5. **Overview table** — categories, counts, links to sections
6. **Quick links** — "Get Started" and "View on GitHub" buttons

## Content Pages

- Start with a level-1 heading matching the nav label
- Use level-2 headings for major sections, level-3 for subsections
- Keep paragraphs short — 2-4 sentences
- Use tables for reference data (fields, commands, options)
- Use admonitions for callouts:
  ```markdown
  !!! info "Title"
      Content goes here.

  !!! warning
      Important caveat.

  !!! tip
      Helpful suggestion.
  ```
- Use code blocks with language tags for all code:
  ````markdown
  ```yaml
  key: value
  ```
  ````
- Use tabs for multi-language or multi-platform examples:
  ```markdown
  === "Linux"
      ```bash
      sudo apt install package
      ```

  === "macOS"
      ```bash
      brew install package
      ```
  ```

## Cross-References

- Link to other pages using relative paths: `[Topic Name](section/topic.md)`
- Link to sections with anchors: `[Error Handling](tools/hery.md#error-handling)`
- Avoid absolute URLs for internal links — they break in local dev

## File Naming

- Lowercase, hyphen-separated: `api-gateway.md`, `getting-started.md`
- Match the conceptual hierarchy: `application-cache-redis.md` for nested topics
- Keep names short but descriptive
