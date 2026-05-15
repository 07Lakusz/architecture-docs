# ADR-005: ruff + ty for Linting and Type Checking

## Status

Accepted

## Context

The Python linting and type-checking landscape has consolidated:

| Legacy Tool | Modern Replacement |
|---|---|
| flake8 | ruff (linting) |
| black | ruff (formatting) |
| isort | ruff (import sorting) |
| mypy | ty (type checking) |
| pylint | ruff (most rules) |
| bandit | ruff (security rules) |

**ruff** handles linting, formatting, import sorting, and security. **ty** handles type checking with significantly faster performance than mypy.

## Decision

All Python projects use:
- **ruff** for linting and formatting (configuration in `pyproject.toml`)
- **ty** for type checking (configuration in `pyproject.toml` or `ty.toml`)

Legacy tools (mypy, black, flake8, isort, pylint, bandit) are not used.

## Consequences

**Positive:**
- Two tools replace six
- ruff is extremely fast (Rust-based)
- ty is faster than mypy with comparable accuracy
- Single configuration file (`pyproject.toml`)

**Negative:**
- ty is newer and has fewer configuration options than mypy
- Some mypy-specific features (plugins) are not available
- Migration required for existing mypy configurations

## Configuration

```toml
# pyproject.toml
[tool.ruff]
target-version = "py312"
line-length = 100

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "ANN", "S", "B", "A", "C4", "PTH"]

[tool.ty]
environment = [{ python-version = "3.12" }]
```

## Related

- [ADR-004: uv as Package Manager](004-uv-package-manager.md)
- [Python 2026 Standard](../standards/python-2026.md)
