# Python Best Practices

Guidelines for writing idiomatic, clean, and maintainable Python code. Based on PEP 8,
PEP 621, and the broader Python community standards for modern Python (3.10+).

## Sub-Files

| File | When to read |
|------|-------------|
| [style.md](style.md) | PEP 8 formatting, naming conventions, imports, docstrings, comments |
| [error-handling.md](error-handling.md) | Exception hierarchy, try/except patterns, custom exceptions, EAFP vs LBYL |
| [typing.md](typing.md) | Type hints, generics, Protocol, TypeVar, mypy configuration |
| [testing.md](testing.md) | pytest patterns, fixtures, parametrize, mocking, coverage |
| [packaging.md](packaging.md) | pyproject.toml, virtual environments, dependency management, project structure |
| [async.md](async.md) | asyncio patterns, async/await, tasks, event loop, aiohttp |
| [linting.md](linting.md) | Ruff configuration, mypy, recommended rules |

## Key Principles

1. **Readability counts** — code is read far more than it is written (PEP 20)
2. **Explicit over implicit** — name things clearly; avoid side-effectful imports
3. **Use the standard library first** — prefer built-ins and stdlib over pip packages
4. **Type everything public** — annotate all public functions, methods, and module-level variables
5. **Fail fast and loud** — raise specific exceptions early; never silently swallow errors
6. **Flat is better than nested** — guard-clause early returns; avoid deep nesting
7. **One config file** — consolidate tool config in `pyproject.toml` (PEP 621)

## Sources

- [PEP 8 – Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [PEP 20 – The Zen of Python](https://peps.python.org/pep-0020/)
- [PEP 621 – Storing project metadata in pyproject.toml](https://peps.python.org/pep-0621/)
- [PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
- [Ruff documentation](https://docs.astral.sh/ruff/)
- [pytest documentation](https://docs.pytest.org/)
- [Python Packaging User Guide](https://packaging.python.org/)
