# ADR-001: Stdio-Only Transport for MCP Servers

## Status

Accepted

## Context

The Model Context Protocol (MCP) supports multiple transport mechanisms: stdio, HTTP/SSE, and WebSocket. For local agentic AI servers (nanobot, Claude Desktop, Cursor), the transport choice affects:

- **Security** — network transports expose attack surface
- **Complexity** — HTTP servers require port management, auth, lifecycle handling
- **Reliability** — stdio is the simplest, most battle-tested transport
- **Compatibility** — all major MCP clients support stdio

## Decision

All MCP servers under `07Lakusz` use **stdio transport exclusively**. No HTTP or WebSocket transport will be added unless a specific remote deployment requirement emerges.

## Consequences

**Positive:**
- Minimal attack surface — no network ports
- Simplest possible server lifecycle (spawn, communicate, exit)
- Universal client compatibility
- No auth/secret management for transport layer

**Negative:**
- No remote access (must run locally or via SSH tunnel)
- One server instance per client (no shared server)
- Debugging requires MCP Inspector or log inspection

## Related

- [ADR-003: Stderr-Only Logging](003-stderr-logging.md) — stdout is reserved for JSON-RPC
- [MCP Compliance Standard](../standards/mcp-compliance.md)
