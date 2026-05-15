# MCP Compliance Standard

## Purpose

This document defines the baseline requirements for all MCP servers under `07Lakusz`. Repo-specific `docs/mcp-compliance.md` files may extend this with project-specific additions but must not relax these requirements.

## Transport

All MCP servers use **stdio transport** exclusively. See [ADR-001](../adr/001-stdio-transport.md).

## Stdout Discipline

**CRITICAL:** Never write to stdout in any module that participates in MCP tool calls. Stdout is reserved for JSON-RPC messages only.

```python
# ✅ Correct — log to stderr
import sys
import logging
logging.basicConfig(stream=sys.stderr, level=logging.INFO)

# ❌ Wrong — corrupts JSON-RPC
print("debug message")
```

See [ADR-003](../adr/003-stderr-logging.md).

## Tool Design Principles

Each tool must have:

1. **A clear, descriptive `description`** — the LLM uses this to decide when to call it. Write descriptions as instructions, not labels.
2. **A complete `inputSchema`** — every parameter typed, required fields listed, defaults documented.
3. **Return value as `TextContent`** — always return `list[TextContent]` with JSON-serialised data.

## Data Models

All data structures use **Pydantic v2** models. See [ADR-002](../adr/002-pydantic-models.md).

Models provide:
- Validation at the boundary
- `model_dump()` for JSON serialisation
- Field descriptions as inline documentation
- Type safety for static analysis

## Agentic AI Best Practices

| Practice | Requirement |
| --- | --- |
| **Read-only semantic tunnel** | Server does not modify external systems unless explicitly designed to |
| **Flat, pre-processed output** | Tools return ready-to-use data, not raw nested structures |
| **Composable tool chain** | Individual tools can be chained; orchestrator tools available for common pipelines |
| **Idempotent operations** | Safe to retry any tool without side effects |
| **Structured logging** | All operations logged to stderr with context |

## Compliance Checklist

### MCP Specification

| Requirement | Status |
| --- | --- |
| JSON-RPC 2.0 protocol | Handled by `mcp` SDK |
| stdio transport | `server.run()` defaults to stdio |
| Tool registration via `list_tools` | All tools registered |
| Tool dispatch via `call_tool` | Async dispatch |
| `TextContent` return type | All handlers return `list[TextContent]` |
| No stdout logging | Uses stderr logging only |
| Descriptive tool names | snake_case, action-oriented |
| Complete input schemas | All parameters typed and documented |

### Python Standards

| Standard | Requirement |
| --- | --- |
| Google Python Style Guide | Google docstrings, type hints |
| `pathlib.Path` for filesystem | No `os.path` usage |
| Union types with `\|` | `str \| None` syntax |
| Exception chaining with `from` | Used throughout |
| ruff lint compliance | All checks passing |
| No `print()` in production code | Logging only |

## Cross-Reference

| Pattern | mcp-server-stock-analysis | mcp-kozlony-tracker |
| --- | --- | --- |
| SDK | `FastMCP` | `mcp.server.Server` |
| Transport | stdio | stdio |
| Data models | Pydantic in `models/` | Pydantic in `scripts/models.py` |
| Output format | JSON dicts | JSON dicts |
| Logging | loguru to stderr | logging to stderr |
| State management | Stateless | JSON file |
