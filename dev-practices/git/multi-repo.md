# Git Multi-Repo Workspaces

Patterns for working with directories that contain multiple independent git repositories
side by side (monorepo-style workspaces where each subdirectory is its own repo).

## Detecting Multi-Repo Workspaces

When the working directory is **not** itself a git repo but contains subdirectories
that are, treat it as a multi-repo workspace. Detect repos by checking for `.git`
directories in immediate children.

## Status Across All Repos

When asked for `git status` in a multi-repo workspace, scan every child repo and
report only those with changes. Use short format for density.

```bash
for dir in */; do
  [ -d "$dir/.git" ] || continue
  out=$(cd "$dir" && git status --short 2>/dev/null)
  [ -n "$out" ] && echo "=== ${dir%/} ===" && echo "$out" && echo
done
```

Present results grouped by repo. If a repo has no changes, omit it entirely — only
surface repos with uncommitted work.

## Commit Workflow

When the user asks to commit changes in a specific repo:

1. `cd` into that repo (or use `git -C <repo>`)
2. Check `git log --oneline -5` to match the repo's commit message style
3. Review `git diff` (staged + unstaged) to understand what changed
4. Stage only the relevant files — not `git add .`
5. Commit with a message that follows the repo's conventions
6. Report the branch and short SHA when done

When the user asks to commit across multiple repos, handle each repo independently
and in sequence. Each repo gets its own commit message tailored to its changes.

## Excluding Files from a Commit

When the user says to commit a repo but exclude specific files, stage only the files
that should be included. Do not stage the excluded files and do not mention them in
the commit message.

## Drafting Commit Messages

When the user asks for a commit message (without committing yet), show the message
as a code block. Wait for approval before committing. If the user asks for messages
across multiple repos, show them all at once so they can review in batch.

## Common Multi-Repo Operations

| User says | What to do |
|-----------|-----------|
| "git status" | Scan all child repos, show only those with changes |
| "commit X" | Commit in repo X only |
| "commit everything" | Commit each dirty repo independently, one commit per repo |
| "push" | Confirm which repos and branches before pushing — multi-repo push has high blast radius |
| "what repos have changes?" | Same as status — list repos with uncommitted work |
| "diff for X" | Show `git diff` in repo X only |

## Things to Avoid

- **Do not run `git add .` or `git add -A`** in multi-repo workspaces from the parent
  directory — it does nothing useful and may confuse submodule setups.
- **Do not assume repos share branches.** Each repo may be on a different branch.
- **Do not batch-push without confirmation.** Pushing is visible to others; confirm
  repos and branches first.
- **Do not use `-uall` with `git status`** — it can cause performance issues on large
  repos with many untracked files.
