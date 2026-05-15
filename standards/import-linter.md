# Import Linter Standard

## Purpose

[import-linter](https://github.com/seddonym/import-linter) enforces architectural boundaries between Python packages. It prevents circular dependencies and ensures that the layered architecture is respected at the import level.

## Why It Matters

Without import linting, a `tools/` module might accidentally import from `strategies/`, creating a circular dependency that is invisible until runtime. Import linting catches this at CI time.

## Configuration

Configured in `pyproject.toml` under `[tool.importlinter]`.

### Root Packages

Define the top-level packages in your project:

```toml
[tool.importlinter]
root_packages = [
    "core",
    "models",
    "strategies",
    "tools",
]
include_external_packages = true
```

### Layer Contracts

Define the allowed import direction. Lower layers must not import from higher layers:

```toml
[[tool.importlinter.contracts]]
name = "Internal layering"
type = "layers"
layers = [
    "tools",       # Highest — can import from all below
    "templates",   # Can import from strategies, core, models
    "strategies",  # Can import from core, models
    "core",        # Can import from models only
    "models",      # Lowest — must not import from any other layer
]
```

This enforces the dependency rule: **imports must flow downward only**.

### Independence Contracts

Declare that certain packages must not import from each other:

```toml
[[tool.importlinter.contracts]]
name = "Independent domains"
type = "independence"
modules = [
    "core.scoring",
    "core.risk",
]
```

### Forbidden Contracts

Ban specific imports entirely:

```toml
[[tool.importlinter.contracts]]
name = "No print in production"
type = "forbidden"
source_modules = ["core", "strategies", "tools"]
imported_modules = ["builtins.print"]
```

## Layer Architecture Reference

The standard layer stack (top to bottom):

```
tools/        MCP tool wrappers — entry point for agent calls
  ↓
templates/    Pre-built analysis templates (optional layer)
  ↓
strategies/   Trading/analysis strategies
  ↓
core/         Business logic — scoring, risk, divergence, etc.
  ↓
models/       Pydantic data models — no internal imports
```

**Rules:**
- `models/` must not import from any other internal package
- `core/` may only import from `models/`
- `strategies/` may import from `core/` and `models/`
- `tools/` may import from any internal package

## CI Integration

Import linting runs as a separate CI step:

```yaml
- name: Architecture health check
  run: uv run import-linter check
```

Or via Taskfile:

```yaml
# Taskfile.yml
arch-health:
  cmds:
    - uv run import-linter check
```

## Common Failures

| Error | Cause | Fix |
|---|---|---|
| `Import from X to Y is not allowed` | Upward import violation | Move shared code to a lower layer |
| `Circular dependency between X and Y` | Two layers import each other | Extract shared code to a third module |
| `Module X imports forbidden module Y` | Banned import | Remove the import or use an allowed alternative |

## When to Use

Import linting is recommended for projects with:
- 3+ internal packages
- Clear layer separation
- Multiple developers (or agents) contributing code

For single-file scripts or projects with one package, import linting adds unnecessary complexity.
