# Logging Standard

## Purpose

This document defines logging requirements for all Python projects under `07Lakusz`. The key constraint is that **stdio MCP servers must never write to stdout** — all log output goes to stderr.

## Library Choice

| Context | Library | Reason |
|---|---|---|
| **MCP servers (stdio)** | `loguru` | Defaults to stderr, structured, zero-config |
| **CLI tools** | `loguru` | Same benefits, plus colourful output |
| **Libraries (reusable)** | `logging` (stdlib) | No external dependency for consumers |

## The Stdio Constraint

In stdio MCP servers, stdout is reserved for JSON-RPC messages. Any stray `print()` or stdout write corrupts the protocol.

```python
# ✅ Correct — loguru defaults to stderr
from loguru import logger

logger.info("Processing ticker: {}", ticker)
logger.error("Failed to fetch data: {}", exc)

# ❌ Wrong — corrupts JSON-RPC
print("Processing ticker: {}".format(ticker))
```

**Enforcement:** Add a comment header to every module in the MCP tool call path:

```python
# CRITICAL: Never write to stdout. Use logger (stderr) only.
```

## Loguru Configuration

### Basic Setup

```python
import sys
from loguru import logger

# Remove default handler and add stderr-only
logger.remove()
logger.add(
    sys.stderr,
    format="{time:YYYY-MM-DD HH:mm:ss} | {level:<8} | {name}:{function}:{line} | {message}",
    level="INFO",
    colorize=True,
)
```

### Structured Context with `logger.bind()`

```python
# Bind context to a logger instance for a specific operation
ticker_logger = logger.bind(ticker="AAPL")
ticker_logger.info("Fetching quote")       # Output includes ticker=AAPL
ticker_logger.info("Quote received")       # Same context
```

### Level Guidelines

| Level | Use For |
|---|---|
| `DEBUG` | Detailed diagnostics — function entry/exit, variable values |
| `INFO` | Normal operations — "processing started", "fetch complete" |
| `WARNING` | Recoverable issues — "retrying after timeout", "stale cache used" |
| `ERROR` | Operation failures — "API returned 500", "PDF extraction failed" |
| `CRITICAL` | Unrecoverable — "config file missing", "database corrupted" |

### Exception Logging

```python
# ✅ Log exception with full traceback
try:
    data = fetch_data(ticker)
except Exception:
    logger.exception("Failed to fetch data for {}", ticker)
    raise

# ✅ Or log and re-raise with context
try:
    data = fetch_data(ticker)
except requests.Timeout as exc:
    logger.error("Timeout fetching {}: {}", ticker, exc)
    raise MarketDataTimeoutError(ticker) from exc
```

## Stdlib Logging (for libraries)

When writing reusable libraries that should not depend on loguru:

```python
import logging

logger = logging.getLogger(__name__)

def process(data: dict) -> dict:
    logger.info("Processing %d records", len(data))
    try:
        result = transform(data)
    except ValueError as exc:
        logger.error("Transform failed: %s", exc)
        raise
    return result
```

Configure the handler in the application entry point, not the library:

```python
# In your main.py or mcp_server.py
import logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s | %(levelname)-8s | %(name)s | %(message)s",
    stream=sys.stderr,  # Always stderr for MCP servers
)
```

## What to Log

| Event | Level | Example |
|---|---|---|
| Tool call received | DEBUG | `check_new_issues called with state_path=...` |
| External API call | INFO | `Fetching RSS from magyarkozlony.hu` |
| Cache hit/miss | DEBUG | `Cache hit for AAPL (age: 120s)` |
| Retry attempt | WARNING | `Retry 2/3 for API call after 500 error` |
| Operation complete | INFO | `Processed 3 new issues, 0 skipped` |
| Operation failed | ERROR | `PDF extraction failed for issue 2026/46` |
| Unexpected exception | ERROR | `Unhandled exception in process_new_issues` |

## What NOT to Log

- API keys, tokens, or credentials
- Full request/response bodies (log summaries, not payloads)
- PII (personally identifiable information)
- Excessive DEBUG in production (use environment-based level control)

## Environment-Based Configuration

```python
import os
from loguru import logger

log_level = os.getenv("LOG_LEVEL", "INFO")
logger.remove()
logger.add(sys.stderr, level=log_level)
```

Run with: `LOG_LEVEL=DEBUG uv run python -m scripts.mcp_server`

## Anti-Patterns

| Anti-Pattern | Better Approach |
|---|---|
| `print()` for debugging | `logger.debug()` — can be filtered |
| String concatenation in log messages | `logger.info("Value: {}", val)` — lazy evaluation |
| Logging at wrong level (everything INFO) | Use appropriate levels for filtering |
| Silent exception swallowing | Always log before re-raising |
| Logging sensitive data | Redact or omit |
