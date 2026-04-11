# Organization & Progressive Disclosure

How to write the `index.md` entry point and split content into sub-files that
load only when needed.

## The `index.md` entry point

Treat `index.md` as a table of contents, not a home for content. Its job is to
help the AI decide what to load next, and to carry only the principles that
apply across all sub-files.

**Required sections:**

1. **Title + one-paragraph scope** — what this skill covers, what it doesn't
2. **Sub-files table** — two columns: file name, "When to read"
3. **Key Principles** — 3–8 short bullets that apply across the whole skill
4. **Sources** *(optional but recommended)* — external references the skill is
   based on, so readers can verify or go deeper

Keep `index.md` short. If it grows past ~100 lines, something that should be in
a sub-file has leaked into the index.

### Example sub-files table

```markdown
## Sub-Files

| File | When to read |
|------|-------------|
| [structure.md](structure.md) | Setting up a new project, directory layout |
| [config.md](config.md) | Configuring the build tool |
| [testing.md](testing.md) | Writing and running tests |
```

The "When to read" column is load-bearing. Write it from the reader's
perspective — what task are they doing? — not from the content's perspective.

- **Good:** "Debugging a failing test"
- **Bad:** "Contains assertion helpers and fixtures"

## Splitting content into sub-files

Split by **concern**, not by chronology or file type. A good sub-file answers
one question the user might have in isolation.

**Good splits (website-documentation/):**
- `structure.md` — directory layout
- `config.md` — `mkdocs.yml` settings
- `styling.md` — CSS and themes
- `cicd.md` — deployment workflow

Each file is independent: reading `styling.md` doesn't require reading
`structure.md` first.

**Bad splits:**
- `part-1.md`, `part-2.md`, `part-3.md` — chronological, not topical
- `setup.md`, `more-setup.md` — artificial split on the same concern
- `overview.md` — duplicates the index

## The one-level-deep rule

Sub-files should link back to `index.md` or to other siblings, but **the AI
should never need to follow a chain of references**. If `index.md` → `a.md` →
`b.md` is required to answer a question, the AI may only partially read `b.md`
and miss content.

**Fix:** Link `b.md` directly from `index.md` with a clear "When to read"
description, even if it logically belongs "under" `a.md`.

## Sub-file length and table of contents

Sub-files longer than ~100 lines should include a short table of contents at
the top. The AI sometimes reads files partially (e.g. `head -100`), and a ToC
ensures it sees the full scope even in a preview.

```markdown
# API Reference

## Contents
- Authentication
- Core methods
- Error handling
- Examples

## Authentication
...
```

## Organizing by domain

For skills that span multiple independent domains (e.g. a BigQuery skill with
finance, sales, and product datasets), put each domain in its own sub-file.
The AI loads only the relevant domain.

```text
bigquery-skill/
├── index.md          # Overview, pointers to domain files
├── finance.md        # Revenue, billing, ARR
├── sales.md          # Opportunities, pipeline
└── product.md        # API usage, features
```

This is the same pattern as splitting by concern, applied when the concerns are
data domains rather than task types.
