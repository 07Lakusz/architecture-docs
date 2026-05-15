# CI/CD Standard

## Purpose

This document defines the GitHub Actions workflow standard for all Python projects under `07Lakusz`.

## Workflow File

```
.github/workflows/ci.yml
```

## Trigger Events

```yaml
on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]
```

## Job Structure

Every CI pipeline includes these jobs:

### 1. Test Job (required)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.12", "3.13"]

    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4
        with:
          version: "latest"

      - name: Set up Python ${{ matrix.python-version }}
        run: uv python install ${{ matrix.python-version }}

      - name: Install dependencies
        run: uv sync --extra dev

      - name: Lint (Ruff)
        run: uv run ruff check .

      - name: Format check (Ruff)
        run: uv run ruff format --check .

      - name: Type check (ty)
        run: uv run ty check .

      - name: Run tests with coverage
        run: uv run pytest tests/ -v --cov --cov-report=term-missing

      - name: Check coverage threshold
        run: uv run coverage report --fail-under=80

      - name: Architecture health check
        run: uv run import-linter check
```

### 2. MCP Validation Job (MCP servers only)

For MCP server projects, add a validation job:

```yaml
  mcp-validation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Set up Python
        run: uv python install 3.12

      - name: Install dependencies
        run: uv sync

      - name: Validate MCP server starts
        run: |
          timeout 5 uv run python -m scripts.mcp_server || [ $? -eq 124 ] || exit 1
        shell: bash
```

The `timeout` command ensures the server starts without hanging. Exit code 124 means the timeout was reached (expected for a long-running server). Any other non-zero exit code is a failure.

## Taskfile Integration

For projects using [Task](https://taskfile.dev/), define standard tasks:

```yaml
# Taskfile.yml
version: "3"

tasks:
  lint:
    cmds:
      - uv run ruff check .

  lint-fix:
    cmds:
      - uv run ruff check . --fix

  format:
    cmds:
      - uv run ruff format --check .

  format-fix:
    cmds:
      - uv run ruff format .

  type-check:
    cmds:
      - uv run ty check .

  test:
    cmds:
      - uv run pytest tests/ -v --cov --cov-report=term-missing

  coverage:
    cmds:
      - uv run coverage report --fail-under=80

  arch-health:
    cmds:
      - uv run import-linter check

  check:
    cmds:
      - task: lint
      - task: format
      - task: type-check
      - task: test
      - task: arch-health
```

Then CI steps become simpler:

```yaml
- name: Lint
  run: uv run task lint

- name: Type check
  run: uv run task type-check

- name: Test
  run: uv run task test

- name: Full quality gate
  run: uv run task check
```

## Pre-commit Hooks

For local development, configure pre-commit:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.5.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
```

Install with: `uv run pre-commit install`

## Release Workflow (optional)

For projects that publish packages:

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Build
        run: uv build

      - name: Publish to PyPI
        run: uv publish
        env:
          UV_PUBLISH_TOKEN: ${{ secrets.PYPI_TOKEN }}
```

## Badge

Add a CI status badge to the repo README:

```markdown
![CI](https://github.com/07Lakusz/<repo-name>/actions/workflows/ci.yml/badge.svg)
```
