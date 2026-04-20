# Linting

Use [Ruff](https://docs.astral.sh/ruff/) as the all-in-one linter and formatter. It replaces
flake8, isort, pyupgrade, pydocstyle, and Black in a single fast tool.

## Ruff Configuration

All configuration lives in `pyproject.toml`:

```toml
[tool.ruff]
target-version = "py312"
line-length = 88

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "B",    # flake8-bugbear
    "C4",   # flake8-comprehensions
    "UP",   # pyupgrade
    "SIM",  # flake8-simplify
    "TID",  # flake8-tidy-imports
    "TCH",  # flake8-type-checking
    "ANN",  # flake8-annotations (public APIs must be typed)
    "S",    # flake8-bandit (security)
    "PT",   # flake8-pytest-style
    "RUF",  # Ruff-native rules
]
ignore = [
    "ANN101",  # missing type for `self` — not needed
    "ANN102",  # missing type for `cls` — not needed
    "S101",    # use of assert — acceptable in tests
]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["ANN", "S"]   # relax annotation and security rules in tests

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
```

## Key Rule Sets

| Code | Plugin | What it catches |
|------|--------|----------------|
| `E`/`W` | pycodestyle | PEP 8 style violations |
| `F` | pyflakes | Undefined names, unused imports |
| `I` | isort | Import ordering |
| `B` | bugbear | Likely bugs and bad practices |
| `UP` | pyupgrade | Upgrade syntax to target Python version |
| `SIM` | simplify | Simplifiable code patterns |
| `TCH` | type-checking | Move pure-type imports under `TYPE_CHECKING` |
| `ANN` | annotations | Enforce type annotations on public APIs |
| `S` | bandit | Common security issues |
| `PT` | pytest-style | Consistent pytest usage |
| `RUF` | Ruff native | Python anti-patterns, ambiguous characters |

## Running Ruff

```bash
ruff check .              # lint
ruff check --fix .        # lint + auto-fix safe fixes
ruff format .             # format (replaces Black)
ruff format --check .     # format check only (CI)
```

## mypy

```toml
[tool.mypy]
python_version = "3.12"
strict = true
warn_return_any = true
warn_unused_ignores = true
no_implicit_reexport = true
```

Run:
```bash
mypy src/
```

- Install stubs for third-party packages: `pip install types-requests types-PyYAML`
- Suppress a single line with `# type: ignore[error-code]` — include the code, never bare `# type: ignore`

## Pre-commit Integration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.4.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.10.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]
```

## CI Example (GitHub Actions)

```yaml
- name: Lint
  run: |
    ruff check .
    ruff format --check .
    mypy src/
```

## Rule Exceptions

- Disable a rule for a single line: `# noqa: E501`
- Include the rule code — bare `# noqa` is flagged by `RUF100`
- Disable a rule project-wide only when it conflicts with an intentional pattern; document why in `pyproject.toml`
