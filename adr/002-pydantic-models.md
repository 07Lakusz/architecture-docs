# ADR-002: Pydantic v2 for All Data Models

## Status

Accepted

## Context

MCP tool inputs and outputs cross a boundary between the agent (LLM) and the server (Python). Data must be validated, serialised, and documented at this boundary. Options include:

- **Dataclasses** — standard library, no validation, no JSON schema
- **TypedDict** — type hints only, no runtime validation
- **Pydantic v2** — validation, serialisation, JSON schema generation, field descriptions

## Decision

All data models in MCP servers use **Pydantic v2** (`pydantic>=2.0`). Models are defined in a dedicated `models/` or `scripts/models.py` module and serve as the single source of truth for data contracts.

## Consequences

**Positive:**
- Runtime validation rejects invalid data at the boundary
- `model_dump()` provides consistent JSON serialisation
- Field descriptions serve as inline documentation for agents
- `model_json_schema()` can auto-generate MCP tool input schemas
- Type checkers (ty, mypy) can verify model usage

**Negative:**
- Additional dependency (pydantic ~200KB)
- Slight import-time overhead vs dataclasses
- Team must know Pydantic v2 API (not v1)

## Related

- [Python 2026 Standard](../standards/python-2026.md) — type annotation discipline
- [MCP Compliance Standard](../standards/mcp-compliance.md) — tool design principles
