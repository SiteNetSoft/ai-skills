# Structure & Layout

## Single file vs skill graph

Pick the format based on how much the skill needs to cover.

**Use a single `.md` file when:**

- The topic is tight and coherent (one technology, one workflow)
- Total content is under ~400 lines
- There's no natural way to split into independent concerns
- Example: a commit-message style guide

**Use a skill graph (directory with `index.md`) when:**

- Content exceeds ~400 lines or would if written completely
- The topic has independent concerns a reader might want separately (e.g. testing
  vs. styling vs. CI/CD)
- Different tasks need different sub-sections
- Example: a full "build docs sites with MkDocs" guide

When in doubt, default to a skill graph. It's easier to keep the graph tight than
to split a bloated single file later.

## Directory layout for skill graphs

```text
my-skill/
├── index.md           # Entry point — table of contents + key principles
├── topic-a.md         # Focused sub-file
├── topic-b.md         # Focused sub-file
└── topic-c.md         # Focused sub-file
```

- `index.md` is the required entry point. It's the only file always loaded.
- Sub-files live as siblings — **do not nest** sub-files in further subdirectories
  unless a sub-file is itself a full skill graph.
- Sub-file names are short, lowercase, hyphen-separated: `error-handling.md`, not
  `errorHandling.md` or `ErrorHandling.md`.
- Sub-file names should describe content at a glance: `cicd.md`, not `doc2.md`.

## Where skills live in this repo

| Directory | Scope |
|-----------|-------|
| `dev-practices/<lang-or-framework>/` | Language / framework best practices (Go, Quarkus, etc.) |
| `writing/<topic>/` | Documentation, docs sites, commit messages, PR descriptions |
| `meta/<topic>/` | Skills about skills themselves (this one lives here) |

Create a new top-level category only when a skill genuinely doesn't belong in any
existing one. Categories are cheap to add but fragment the repo if overused.

## Path conventions

- **Always use forward slashes** in file references, even on Windows:
  `reference/guide.md`, not `reference\guide.md`. Unix-style paths work everywhere.
- **Relative links only** — `[structure.md](structure.md)`, not absolute or
  repo-rooted paths. This keeps the skill portable when symlinked into a project.
- **Link to sub-files from `index.md`**. Cross-linking between sub-files is fine
  but should not replace the index-as-table-of-contents role.
