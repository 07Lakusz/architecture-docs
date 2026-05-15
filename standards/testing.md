# Testing Standard

## Purpose

This document defines the testing requirements for all Python projects under `07Lakusz`. Repo-specific test files extend this baseline with domain-specific fixtures and test cases.

## Tooling

| Tool | Purpose | Configuration |
|---|---|---|
| **pytest** | Test runner | `pyproject.toml` `[tool.pytest.ini_options]` |
| **pytest-cov** | Coverage reporting | `pyproject.toml` `[tool.coverage.*]` |
| **pytest-asyncio** | Async test support | `asyncio_mode = "auto"` |
| **pytest-mock** | Mocking (mocker fixture) | Preferred over `unittest.mock` directly |

## Coverage Requirements

| Metric | Minimum |
|---|---|
| **Branch coverage** | 80% |
| **Line coverage** | 85% |

Coverage is enforced in CI. Builds fail if thresholds are not met.

```toml
# pyproject.toml
[tool.coverage.run]
source = ["."]
branch = true

[tool.coverage.report]
fail_under = 80
show_missing = true
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "if __name__ == .__main__.",
]
```

## Test Structure

```
tests/
  conftest.py          # Shared fixtures (module-scoped, function-scoped)
  test_<module>.py     # One test file per source module
  test_issue_NN_*.py   # Regression tests for specific issues
  test_integration.py  # Integration tests (if applicable)
```

### Naming Conventions

- Test files: `test_<module>.py` or `test_issue_NN_<description>.py`
- Test functions: `test_<what>_<condition>_<expected>`
- Test classes: `Test<GroupName>` (only when grouping related tests)

```python
# ✅ Clear, descriptive names
def test_calculate_rsi_with_insufficient_data_returns_none():
def test_fetch_raises_on_http_404():
def test_process_new_issues_skips_already_processed():

# ❌ Vague names
def test_rsi():
def test_fetch():
def test_process():
```

## Fixture Design

### Scope Discipline

| Scope | Use For | Example |
|---|---|---|
| `function` (default) | Fresh state per test | Mock objects, temporary files |
| `module` | Expensive setup shared across tests | Sample DataFrames, parsed RSS |
| `session` | One-time setup for all tests | Database connections (rare) |

### Fixture Principles

1. **Keep `conftest.py` lean** — only truly shared fixtures go here
2. **Prefer factory fixtures** over static data when tests need variation
3. **Use `tmp_path`** for filesystem operations — never write to the project directory
4. **Document fixtures** with docstrings explaining what they provide

```python
import pytest
import pandas as pd
import numpy as np


@pytest.fixture
def sample_dataframe() -> pd.DataFrame:
    """Return a 100-row OHLCV DataFrame with SMA, RSI, MACD columns."""
    np.random.seed(42)
    dates = pd.date_range("2024-01-01", periods=100, freq="D")
    prices = pd.Series(100 * (1 + np.random.normal(0, 0.02, 100)).cumprod(), index=dates)
    return pd.DataFrame({
        "Open": prices * 0.99,
        "High": prices * 1.01,
        "Low": prices * 0.98,
        "Close": prices,
        "Volume": np.random.randint(1_000_000, 5_000_000, 100),
    })


@pytest.fixture
def sample_dataframe_factory():
    """Factory fixture — returns a function that creates DataFrames with custom parameters."""
    def _make(rows: int = 100, seed: int = 42) -> pd.DataFrame:
        np.random.seed(seed)
        dates = pd.date_range("2024-01-01", periods=rows, freq="D")
        prices = pd.Series(100 * (1 + np.random.normal(0, 0.02, rows)).cumprod(), index=dates)
        return pd.DataFrame({"Close": prices})
    return _make
```

## Mocking Strategy

### Use `pytest-mock` (mocker fixture)

```python
def test_fetch_raises_on_timeout(mocker):
    mocker.patch("requests.get", side_effect=requests.Timeout)
    with pytest.raises(MarketDataTimeoutError):
        fetch_quote("AAPL")
```

### What to Mock

| Mock | Do Not Mock |
|---|---|
| HTTP requests (requests, httpx) | Pure functions (calculations, transformations) |
| Filesystem I/O (use `tmp_path` instead) | Pydantic model construction |
| External APIs | Simple data structures |
| Time-dependent code (`datetime.now()`) | Internal module imports |

### Patch at the Right Level

```python
# ✅ Patch where it is used, not where it is defined
mocker.patch("core.fetcher.requests.get", side_effect=Timeout)

# ❌ Fragile — breaks if import path changes
mocker.patch("requests.get", side_effect=Timeout)
```

## Parametrised Tests

Use `@pytest.mark.parametrize` for testing multiple inputs against the same logic:

```python
@pytest.mark.parametrize("signal,expected", [
    ("BULLISH", "buy"),
    ("BEARISH", "sell"),
    ("NEUTRAL", "hold"),
])
def test_signal_to_action(signal, expected):
    assert signal_to_action(signal) == expected


@pytest.mark.parametrize("invalid_input", [
    "", None, "UNKNOWN", 123,
])
def test_signal_to_action_rejects_invalid(invalid_input):
    with pytest.raises(ValueError):
        signal_to_action(invalid_input)
```

## Async Tests

For async functions, use `pytest-asyncio`:

```toml
# pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"
```

```python
@pytest.mark.asyncio
async def test_async_fetch_returns_data():
    result = await fetch_data("AAPL")
    assert "price" in result
```

## Regression Tests

When a bug is fixed, add a regression test with the issue number:

```python
# tests/test_issue_7_fractional_shares.py
"""Regression test for issue #7: fractional share rounding."""

def test_fractional_shares_round_to_4_decimals():
    result = calculate_position_size(
        account_value=10_000,
        entry_price=150.25,
        stop_price=145.00,
    )
    # Must preserve 4 decimal places for fractional shares
    assert result["shares"] == round(result["shares"], 4)
```

## Test Markers

Define custom markers in `pyproject.toml`:

```toml
[tool.pytest.ini_options]
markers = [
    "slow: marks tests as slow (deselect with '-m \"not slow\"')",
    "integration: marks integration tests requiring external services",
    "regression: marks regression tests for specific issues",
]
```

```python
@pytest.mark.slow
def test_full_backtest_pipeline():
    """Takes ~30 seconds — skip in quick runs."""
    ...

@pytest.mark.integration
def test_live_api_call():
    """Requires network access — skip in CI if needed."""
    ...
```

Run only fast tests: `pytest -m "not slow"`

## CI Integration

Tests run on every push and PR. The CI pipeline:

1. Runs the full test suite with coverage
2. Fails if coverage is below threshold
3. Reports coverage summary in the workflow output

```yaml
- name: Run tests with coverage
  run: uv run pytest tests/ -v --cov --cov-report=term-missing

- name: Check coverage threshold
  run: uv run coverage report --fail-under=80
```

## Anti-Patterns

| Anti-Pattern | Better Approach |
|---|---|
| Tests that depend on execution order | Each test is independent |
| Testing implementation details | Test behaviour (inputs → outputs) |
| Large test files (>500 lines) | Split by module or concern |
| Copy-pasting setup code | Use fixtures |
| `assert True` or no assertion | Every test must assert something |
| Catching all exceptions blindly | Assert specific exception types |
| Tests that hit the real network | Mock or use recorded responses |
