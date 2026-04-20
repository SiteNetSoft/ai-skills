# Packaging & Project Structure

## Project Layout (src layout)

```
my-project/
├── pyproject.toml
├── README.md
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── core.py
└── tests/
    └── test_core.py
```

- Use the **src layout** — prevents accidental imports of the package from the project root
- `__init__.py` marks a directory as a package; keep it minimal (re-exports only, if any)

## pyproject.toml (PEP 621)

Single config file for all tools. Minimal example:

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "my-project"
version = "0.1.0"
description = "Short description"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    "httpx>=0.27",
]

[project.optional-dependencies]
dev = [
    "pytest>=8",
    "pytest-cov",
    "mypy>=1.10",
    "ruff>=0.4",
]

[project.scripts]
my-cli = "mypackage.cli:main"
```

- Do not use `setup.py` or `setup.cfg` for new projects
- Pin minimum versions with `>=`; avoid exact pins (`==`) in libraries
- Use `[project.optional-dependencies]` for dev/test/docs extras

## Virtual Environments

Always develop inside a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate      # Linux / macOS
.venv\Scripts\activate         # Windows

pip install -e ".[dev]"        # editable install with dev extras
```

- `.venv` is the conventional name; add it to `.gitignore`
- Editable install (`-e`) means changes to `src/` are reflected immediately without reinstalling
- Never install packages into the system Python

## Dependency Management

| Tool | Use case |
|------|---------|
| `pip` + `pyproject.toml` | Simple projects, libraries |
| `pip-tools` (`pip-compile`) | Reproducible lockfiles from `requirements.in` |
| `uv` | Fast resolver + installer, drop-in pip replacement |
| `hatch` | Full project management (envs, versioning, build) |
| `poetry` | Integrated resolver + publishing |

Recommended: **uv** for speed in CI; **hatch** or **poetry** when you want an opinionated full workflow.

Lock files for applications:
```bash
uv pip compile pyproject.toml -o requirements.lock
```

## Build Backends

| Backend | When to use |
|---------|------------|
| `hatchling` | General purpose; good default |
| `flit-core` | Pure-Python packages, minimal config |
| `setuptools` | Legacy codebases, C extensions |
| `maturin` | Python + Rust (PyO3) |

## Entry Points

CLI entry points in `pyproject.toml`:
```toml
[project.scripts]
my-tool = "mypackage.cli:main"

[project.gui-scripts]
my-gui = "mypackage.app:run"
```

Plugin/extension points:
```toml
[project.entry-points."mypackage.plugins"]
csv = "mypackage.plugins.csv:CsvPlugin"
```

## Versioning

- Single source of truth: version in `pyproject.toml` under `[project] version`
- Or use dynamic versioning from git tags:
  ```toml
  [tool.hatch.version]
  source = "vcs"
  ```
- Follow [Semantic Versioning](https://semver.org/) for public APIs
- `__version__` in `__init__.py` is optional; use `importlib.metadata` instead:
  ```python
  from importlib.metadata import version
  __version__ = version("my-project")
  ```
