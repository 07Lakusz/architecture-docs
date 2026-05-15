# Architecture Docs - 07Lakusz

Centralised architecture decision records, coding standards, and design patterns for all repositories under the `07Lakusz` GitHub account.

## Purpose

This repo holds **cross-cutting concerns** - decisions and standards that apply across multiple repositories. Each repo keeps its own `docs/` folder for repo-specific notes and links back to this hub for shared content.

**Rule of thumb:** If a standard or decision applies to more than one repo, it belongs here. If it is specific to one repo's domain, it stays there.

## Structure

```
architecture-docs/
  README.md              - This file (hub index)
  adr/                   - Architecture Decision Records
  standards/             - Coding standards and conventions
  templates/             - Reusable templates for repo-level docs
```

## Repositories

| Repository | Description | Docs |
|---|---|---|
| [mcp-server-stock-analysis](https://github.com/07Lakusz/mcp-server-stock-analysis) | Financial analysis MCP server | [docs/](https://github.com/07Lakusz/mcp-server-stock-analysis/tree/main/docs) |
| [mcp-kozlony-tracker](https://github.com/07Lakusz/mcp-kozlony-tracker) | Magyar Koezlony monitor | [docs/](https://github.com/07Lakusz/mcp-kozlony-tracker/tree/main/docs) |
| [mcp-agent-crm](https://github.com/07Lakusz/mcp-agent-crm) | MCP-based CRM agent | - |
| [looper-agent](https://github.com/07Lakusz/looper-agent) | Agent framework component | - |

## Architecture Decision Records

| ADR | Title | Status |
|---|---|---|
| [ADR-001](adr/001-stdio-transport.md) | Stdio-Only Transport for MCP Servers | Accepted |
| [ADR-002](adr/002-pydantic-models.md) | Pydantic v2 for All Data Models | Accepted |
| [ADR-003](adr/003-stderr-logging.md) | Stderr-Only Logging in Stdio MCP Servers | Accepted |
| [ADR-004](adr/004-uv-package-manager.md) | uv as Sole Python Package Manager | Accepted |
| [ADR-005](adr/005-ruff-ty-lint-type.md) | ruff + ty for Linting and Type Checking | Accepted |
| [ADR-006](adr/006-loguru-logging.md) | loguru for Application Logging | Accepted |
| [ADR-007](adr/007-pytest-testing.md) | pytest as Sole Test Framework | Accepted |

## Standards

| Standard | Description |
|---|---|
| [Python 2026](standards/python-2026.md) | Python language features, tooling stack, and coding conventions |
| [MCP Compliance](standards/mcp-compliance.md) | Baseline MCP server requirements for all projects |
| [Testing](standards/testing.md) | pytest, pytest-cov, mocking strategy, fixture design, parametrisation |
| [Import Linter](standards/import-linter.md) | Architectural boundary enforcement between packages |
| [CI/CD](standards/ci-cd.md) | GitHub Actions workflow standard, Taskfile, pre-commit, release |
| [Logging](standards/logging.md) | loguru usage, stderr discipline, structured context, level guidelines |

## Templates

| Template | Use Case |
|---|---|
| [MCP Server Docs](templates/mcp-server-docs.md) | Starter docs/ folder for new MCP server repos |

## Contributing

1. Create ADRs in `adr/` using the format `NNN-short-title.md`
2. Update this README index when adding new entries
3. Link from repo-level `docs/` folders back to relevant entries here
