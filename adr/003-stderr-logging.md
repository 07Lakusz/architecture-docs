# ADR-003: Stderr-Only Logging in Stdio MCP Servers

## Status

Accepted

## Context

MCP servers using stdio transport communicate with the client via JSON-RPC over **stdin/stdout**. Any output to stdout that is not a valid JSON-RPC message will corrupt the communication channel. This means:

- `print()` calls break the server
- Logging to stdout breaks the server
- Only structured JSON-RPC responses may touch stdout

## Decision

All logging in stdio MCP servers goes to **stderr** exclusively. Two acceptable approaches:

1. **Python `logging` module** configured with `stream=sys.stderr`
2. **loguru** logger (defaults to stderr)

No module in the MCP tool call path may write to stdout.

## Consequences

**Positive:**
- JSON-RPC communication is never corrupted
- Log output is still visible in terminal / agent logs
- loguru provides structured context via `logger.bind()`

**Negative:**
- Developers must remember the rule (no `print()` for debugging)
- Accidental stdout writes cause hard-to-debug failures
- Requires linting rule or code review discipline

## Enforcement

Add a comment header to `mcp_server.py` and all tool modules:

```python
# CRITICAL: Never write to stdout. Use stderr logging only.
```

## Related

- [ADR-001: Stdio-Only Transport](001-stdio-transport.md)
- [MCP Compliance Standard](../standards/mcp-compliance.md)
