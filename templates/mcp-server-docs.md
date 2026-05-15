# MCP Server Docs Template

Use this template to create the `docs/` folder for a new MCP server repository. Copy the relevant sections and customise for your project.

## File: `docs/mcp-compliance.md`

```markdown
# MCP Compliance & Design Reference — [project-name]

## Purpose

[Brief description of what this MCP server does and its domain.]

This server adheres to the [MCP Compliance Standard](https://github.com/07Lakusz/architecture-docs/blob/main/standards/mcp-compliance.md). This document captures only repo-specific additions.

## Tool Reference

| Tool | Purpose | Idempotent |
| --- | --- | --- |
| `tool_name` | [Description] | Yes/No |

## Deployment

### Local stdio (nanobot / Claude Desktop)

\`\`\`json
{
  "mcpServers": {
    "[server-name]": {
      "command": "uv",
      "args": ["run", "python", "-m", "scripts.mcp_server"],
      "cwd": "/path/to/[project-name]"
    }
  }
}
\`\`\`

## Repo-Specific Notes

[Any deviations from the standard, domain-specific patterns, or project-specific conventions.]
```

## File: `docs/README.md` (optional)

For servers with complex usage, add a usage guide:

```markdown
# [Project Name] — Usage Guide

## Overview
[What the server does, what tools it exposes.]

## Quick Start
[How to install, configure, and connect.]

## Tool Reference
[Detailed tool documentation with input/output examples.]

## Architecture
[Internal architecture if relevant.]

## Cross-Reference
- [Architecture Docs](https://github.com/07Lakusz/architecture-docs) — shared standards and ADRs
- [Python 2026 Standards](https://github.com/07Lakusz/architecture-docs/blob/main/standards/python-2026.md)
- [MCP Compliance](https://github.com/07Lakusz/architecture-docs/blob/main/standards/mcp-compliance.md)
```
