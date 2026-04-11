# ai-skills

A collection of AI skill files (`CLAUDE.md` and similar) for improving AI-assisted software development workflows.

## Categories

- **`writing/`** — Skills for technical documentation, commit messages, PR descriptions, and other written content
- **`dev-practices/`** — Skills for code quality, testing, CI/CD, security, and engineering best practices
- **`meta/`** — Skills about authoring skills themselves (see [`meta/skill-authoring/`](meta/skill-authoring/index.md))

## Skill Format

Skills come in two formats:

- **Single file** — One `.md` file for focused topics (e.g., `dev-practices/golang.md`). Best for skills under ~400 lines.
- **Skill graph** — A directory with an `index.md` entry point and focused sub-files (e.g., `writing/website-documentation/`). The AI reads the index and only loads sub-files relevant to the current task. Best for larger skills that cover multiple concerns.

## Usage

Copy or symlink the relevant skill file (or directory) into the root of your project directory. AI coding assistants like [Claude Code](https://claude.com/claude-code) will automatically pick it up.

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
