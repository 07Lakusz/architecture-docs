# ADR-004: uv as Sole Python Package Manager

## Status

Accepted

## Context

The Python packaging ecosystem has converged on **uv** (by Astral) as the unified replacement for pip + virtualenv + pip-tools. It provides:

- Dependency resolution (replacing pip)
- Virtual environment management (replacing venv/virtualenv)
- Lock file generation (replacing pip-tools/poetry lock)
- Build backend (replacing setuptools for most cases)
- Tool installation (replacing pipx)

## Decision

All Python projects under `07Lakusz` use **uv** as the sole package manager. `pyproject.toml` is the single source of truth for dependencies. A `uv.lock` file is committed to the repository.

## Consequences

**Positive:**
- Single tool replaces 4+ legacy tools
- Fast resolution (Rust-based)
- Reproducible builds via lock file
- Consistent workflow across all projects

**Negative:**
- Relatively new tool (first stable release 2024)
- Some edge cases with native extensions
- Team must install uv separately

## Workflow

```bash
# Initialise a new project
uv init --python 3.12

# Add a dependency
uv add requests pydantic

# Add dev dependencies
uv add --dev pytest ruff ty

# Run commands in the virtual environment
uv run pytest
uv run ruff check .

# Sync lock file
uv lock
```

## Related

- [ADR-005: ruff + ty](../adr/005-ruff-ty-lint-type.md)
- [Python 2026 Standard](../standards/python-2026.md)
