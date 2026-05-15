# ADR-007: pytest as Sole Test Framework

## Status

Accepted

## Context

Python has two main test frameworks:

- **`unittest`** (stdlib) — class-based, verbose, Java-influenced
- **`pytest`** — function-based, fixtures, parametrisation, plugin ecosystem

For projects under `07Lakusz`, the requirements are:
1. Minimal boilerplate (no class inheritance for tests)
2. Powerful fixture system for shared setup/teardown
3. Parametrised tests for data-driven testing
4. Coverage integration via `pytest-cov`
5. Async support via `pytest-asyncio`
6. Mocking via `pytest-mock`

## Decision

All Python projects use **pytest** as the sole test framework. `unittest` is not used.

## Consequences

**Positive:**
- Concise test functions (no class required)
- Fixture system eliminates copy-pasted setup code
- `@pytest.mark.parametrize` for data-driven tests
- Rich plugin ecosystem (cov, asyncio, mock, xdist)
- Excellent error output with diff highlighting

**Negative:**
- External dependency (but included in all `dev` dependencies)
- Magic fixture injection can confuse beginners
- Slower than `unittest` for very small suites (negligible)

## Required Plugins

| Plugin | Purpose |
|---|---|
| `pytest-cov` | Coverage reporting |
| `pytest-asyncio` | Async test support |
| `pytest-mock` | Mocking (mocker fixture) |

## Coverage Threshold

Minimum 80% branch coverage, 85% line coverage. Enforced in CI.

## Related

- [Testing Standard](../standards/testing.md)
- [CI/CD Standard](../standards/ci-cd.md)
