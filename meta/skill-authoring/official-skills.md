# Official Claude Code Skills (`SKILL.md`)

This repo's skills are plain Markdown files designed to be dropped into a
project root as CLAUDE.md-style context. Claude Code also supports a
first-class **Skill** format with YAML frontmatter, discoverability, and
slash-command invocation. Both formats coexist. This page explains how to
convert between them.

## Format differences

|                         | This repo's ai-skills             | Claude Code `SKILL.md`                      |
|-------------------------|-----------------------------------|---------------------------------------------|
| Entry point             | `index.md`                        | `SKILL.md`                                  |
| YAML frontmatter        | No                                | Required (at minimum `description`)         |
| Discovery               | Manual (symlink / copy into repo) | Automatic by `name` and `description`       |
| Invocation              | Always loaded as context          | On-demand by the AI, or `/skill-name`       |
| Supporting files        | Sibling `.md` sub-files           | Sibling files + `scripts/` + any assets     |
| Install location        | Anywhere in the project           | `~/.claude/skills/<name>/` or `.claude/skills/<name>/` |

The writing principles (conciseness, progressive disclosure, one level
deep, consistent terminology) are identical. Only the packaging differs.

## Directory layout for `SKILL.md`

```text
my-skill/
├── SKILL.md              # Required — frontmatter + instructions
├── reference.md          # Optional — loaded on demand
├── examples.md           # Optional — loaded on demand
└── scripts/
    └── helper.py         # Executed, not loaded into context
```

Scripts in `scripts/` are called via bash. Their source does not consume
tokens — only their output does.

## Frontmatter reference

```yaml
---
name: processing-pdfs
description: Extract text and tables from PDFs, fill forms, merge documents. Use when working with PDFs, forms, or document extraction.
---
```

### Required / recommended fields

| Field         | Rules                                                                              |
|---------------|------------------------------------------------------------------------------------|
| `name`        | Lowercase letters, numbers, hyphens only. Max 64 chars. No "anthropic" / "claude". |
| `description` | Third person. Max 1024 chars. Includes **what** it does and **when** to use it.    |

### Optional fields

| Field                      | Purpose                                                                 |
|----------------------------|-------------------------------------------------------------------------|
| `disable-model-invocation` | `true` hides it from auto-loading; user must invoke via `/name`         |
| `user-invocable`           | `false` hides from `/` menu — AI-only reference material                |
| `allowed-tools`            | Tools pre-approved while the skill is active (e.g. `Bash(git *)`)       |
| `argument-hint`            | Autocomplete hint, e.g. `[issue-number]`                                |
| `context: fork`            | Run the skill in a forked subagent context                              |
| `agent`                    | Which subagent type to use with `context: fork` (e.g. `Explore`)        |
| `model` / `effort`         | Override model / effort level while the skill is active                 |
| `paths`                    | Glob patterns that auto-activate the skill for matching files           |

## Naming conventions

Prefer **gerund form** (verb + -ing) for skill names — it reads as a
capability:

- `processing-pdfs`
- `analyzing-spreadsheets`
- `writing-commit-messages`
- `migrating-databases`

Noun phrases (`pdf-processing`) and imperatives (`process-pdfs`) are
acceptable alternatives. Avoid vague names (`helper`, `utils`, `tools`) and
reserved words (`anthropic-*`, `claude-*`).

## Writing the description

The description is how the AI decides whether to load this skill out of
potentially 100+ candidates. It must include **what** and **when**.

**Good:**

> Extract text and tables from PDF files, fill forms, merge documents. Use
> when working with PDF files or when the user mentions PDFs, forms, or
> document extraction.

> Generate descriptive commit messages by analyzing git diffs. Use when the
> user asks for help writing commit messages or reviewing staged changes.

Always third person — first-person or second-person breaks discovery:

- Bad: "I can help you process Excel files"
- Bad: "You can use this to process Excel files"
- Good: "Processes Excel files and generates reports"

Descriptions longer than 250 characters get truncated in the skill listing,
so **front-load the key use case**.

## Install locations

| Scope      | Path                                                |
|------------|-----------------------------------------------------|
| Personal   | `~/.claude/skills/<skill-name>/SKILL.md`            |
| Project    | `.claude/skills/<skill-name>/SKILL.md`              |
| Plugin     | `<plugin>/skills/<skill-name>/SKILL.md`             |
| Enterprise | Managed via settings                                |

Higher priority wins on name conflicts: enterprise > personal > project.

## Converting a this-repo skill to `SKILL.md`

1. Create `<skill-name>/SKILL.md` at the target location
2. Copy `index.md`'s body into `SKILL.md` and add frontmatter:
   ```yaml
   ---
   name: writing-documentation
   description: <what + when, third person, front-loaded>
   ---
   ```
3. Rename any relative links from `[structure.md](structure.md)` to match
   the new file names if needed (usually unchanged)
4. Verify `SKILL.md` body stays under ~500 lines — split further if not
5. Test by asking the AI a task that should trigger the skill, and verify
   it loads automatically

## When to use which format

- **Use this repo's format** when the skill is meant to be dropped into a
  specific project as always-on context (language best practices, project
  conventions, style guides)
- **Use `SKILL.md` format** when the skill is a reusable capability the AI
  should load on demand (processing a file type, running a workflow,
  triggering a scripted action)

The formats aren't exclusive — a single skill directory can ship both, and
the `index.md` / `SKILL.md` files can even share content via symlink.
