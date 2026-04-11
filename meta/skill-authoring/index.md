# Skill Authoring Best Practices

Guidelines for writing effective AI skill files — both the drop-in CLAUDE.md-style
instruction files in this repo and official Claude Code Skills (`SKILL.md` with
frontmatter). This skill is split into focused sub-files — read only what you need.

## What is a "skill" in this repo?

A skill here is a reusable Markdown file (or directory of files) that an AI coding
assistant loads as project context. Two formats are supported:

- **Single file** — one `.md` under ~400 lines for tightly-focused topics
- **Skill graph** — a directory with `index.md` pointing at focused sub-files; the AI
  reads the index and loads only the sub-files relevant to the current task

The skill graph pattern is preferred for anything larger than a single screen of
content. It keeps the AI's context window small by deferring details until needed.

## Sub-Files

| File | When to read |
|------|-------------|
| [structure.md](structure.md) | Deciding single-file vs graph, directory layout, where skills live in this repo |
| [organization.md](organization.md) | Writing the `index.md`, splitting sub-files, progressive disclosure |
| [writing.md](writing.md) | Conciseness, terminology, templates, degrees of freedom |
| [anti-patterns.md](anti-patterns.md) | Mistakes to avoid when writing skills |
| [official-skills.md](official-skills.md) | Converting to the Claude Code `SKILL.md` format with YAML frontmatter |

## Key Principles

1. **Context is a public good** — every token competes with conversation history and
   other context. Write as little as the AI needs, nothing more.
2. **Assume the AI already knows the basics** — skip explanations of common concepts,
   languages, and tools. Only add what the AI wouldn't know from general training.
3. **Progressive disclosure** — the entry point is a table of contents; details live
   in sub-files that load on demand.
4. **One level deep** — sub-files should be referenced directly from the index, not
   from other sub-files, or the AI may only partially read them.
5. **Concrete over abstract** — show real code, real commands, real filenames.
6. **Consistent terminology** — pick one word for a concept and use it throughout.
7. **Iterate from real usage** — observe where the AI struggles or misfires, then
   revise. Don't over-engineer up front.

## Sources

- [Anthropic: Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Claude Code: Extend Claude with skills](https://code.claude.com/docs/en/skills)
- [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
