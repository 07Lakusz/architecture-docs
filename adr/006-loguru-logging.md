# ADR-006: loguru for Application Logging

## Status

Accepted

## Context

Python offers two logging approaches:

- **`logging` (stdlib)** — always available, verbose configuration, handler/formatter hierarchy
- **`loguru`** — single import, sensible defaults, structured context via `bind()`, automatic stderr output

For MCP servers and CLI tools, the key requirements are:
1. Default to stderr (never stdout in stdio MCP servers)
2. Minimal boilerplate
3. Structured context without manual formatter configuration
4. Exception logging with automatic traceback capture

## Decision

All MCP servers and CLI tools use **loguru** as the logging library. The stdlib `logging` module is reserved for reusable libraries that should not impose external dependencies.

## Consequences

**Positive:**
- Zero-config stderr output — no handler/formatter boilerplate
- `logger.bind()` provides structured context cleanly
- `logger.exception()` captures full tracebacks automatically
- Colourised output in terminals

**Negative:**
- External dependency (~200KB)
- Different API from stdlib `logging` (learning curve for contributors)
- Not suitable for libraries (they should use stdlib `logging`)

## Configuration Pattern

```python
import sys
from loguru import logger

logger.remove()
logger.add(
    sys.stderr,
    format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {name}:{function}:{line} | {message}",
    level="INFO",
)
```

## Related

- [ADR-003: Stderr-Only Logging](003-stderr-logging.md)
- [Logging Standard](../standards/logging.md)
