# Python 2026 Code Standards

## The Standard Has Consolidated

In the period spanning 2024 to 2026, the Python ecosystem converged on a compact, coherent set of tools: **uv + ruff + ty + import-linter**. These utilities effectively replace the fragmented legacy stack of `pip`, `venv`, `flake8`, `black`, `isort`, `pylint`, `mypy`, and `bandit`, collapsing eight separate tools into four.

This consolidation ensures that code standards are fully automated, thereby removing the necessity for stylistic debates during technical reviews.

See [ADR-004: uv as Package Manager](../adr/004-uv-package-manager.md) and [ADR-005: ruff + ty](../adr/005-ruff-ty-lint-type.md).

## The Google Python Style Guide (GPSG)

We adopt GPSG as our foundational baseline for the following reasons:

- It provides **explicit guidance for ambiguous cases**, such as module docstrings, argument order, and exception chaining.
- It is **highly compatible with LLMs**, as these models are extensively trained on Google-style Python repositories.
- It is **strictly machine-enforceable** through `ruff` using the `pydocstyle` Google convention.

### Key GPSG Rules

| Rule | GPSG Guidance |
| --- | --- |
| **Docstrings** | Google format: `Args:`, `Returns:`, `Raises:`, `Example:` |
| **Imports** | Single import per line; wildcard imports are strictly prohibited |
| **Exceptions** | Always specify exception types; use `from` for exception chaining |
| **String formatting** | f-strings are the preferred standard |
| **None comparisons** | Use `if x is None:` rather than `if not x:` |
| **Mutable defaults** | Use `dataclasses.field(default_factory=list)` instead of `[]` |

## Modern Python Language Features (3.10–3.14+)

### Structural Pattern Matching (3.10+)

```python
# ✅ Modern — exhaustive, readable, and precise
match signal:
    case "BULLISH":
        return "buy"
    case "BEARISH":
        return "sell"
    case _:
        raise ValueError(f"Unknown signal: {signal}")
```

### Precise Type Narrowing with `TypeIs` (3.13+)

```python
from typing import TypeIs

def is_integer_list(val: list[object]) -> TypeIs[list[int]]:
    return all(isinstance(x, int) for x in val)
```

### Union Types and Generic Collections

- **Union Types (3.10+):** Use the pipe operator (`str | None`) instead of `Optional` or `Union`.
- **Generic Collections (3.9+):** Use built-in types (`list[str]`, `dict[str, float]`) rather than `typing.List` or `typing.Dict`.

### Pathlib for Filesystem Operations

Always use `pathlib.Path` for file manipulations. No `os.path` usage.

## Type Annotation Discipline

| Situation | Required Annotation |
| --- | --- |
| **Public API** | Always require full signatures for arguments and returns |
| **Type Narrowing** | Use `TypeIs[T]` for superior type inference |
| **The Any Rule** | Prohibited in `domain/` and `services/` layers (enforced by `ruff ANN401`) |
| **Class Attributes** | Must be declared in `__init__` or as class variables |

## Exception Handling

```python
# ✅ Specific and explicit handling
try:
    quote = fetch_quote(ticker)
except httpx.TimeoutException as exc:
    raise MarketDataTimeoutError(f"Timeout fetching {ticker}") from exc
```

## Project Structure

All Python projects follow a consistent layout:

```
project-name/
  pyproject.toml          # Project config, dependencies, tool settings
  uv.lock                 # Locked dependencies
  src/ or project_name/   # Source package
    __init__.py
    models/               # Pydantic models
    core/                 # Business logic
    tools/                # MCP tool wrappers (if applicable)
  tests/                  # Test suite
    conftest.py
    test_*.py
  docs/                   # Documentation
    (link to 07Lakusz/architecture-docs for shared standards)
  .github/
    workflows/
      ci.yml              # CI pipeline
```

## CI Pipeline

All projects use GitHub Actions with a standard workflow:

1. Checkout code
2. Install uv
3. Sync dependencies (`uv sync`)
4. Lint (`ruff check .`)
5. Format check (`ruff format --check .`)
6. Type check (`ty check .`)
7. Test (`uv run pytest tests/ -v`)
