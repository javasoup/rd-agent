# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RD-Agent is a Microsoft-developed LLM-Agent framework for automating Research & Development in data-driven industries. It automates data science model development, quantitative trading strategies, and ML engineering tasks. Python 3.10+ required.

## Build & Development Commands

```bash
make dev              # Install dev environment with all optional deps
make install          # Install package in editable mode
make lint             # Run all linters (mypy, ruff, isort, black, toml-sort)
make auto-lint        # Auto-fix lint issues (runs auto-isort, auto-black, auto-toml-sort)
make test-offline     # Run offline tests (no API calls) with coverage
make test             # Run full test suite with coverage
make docs-gen         # Build Sphinx documentation
```

Run a single test:
```bash
python -m pytest test/path/to/test_file.py -k "test_name"
```

Offline test marker: tests that don't need API calls are marked with `@pytest.mark.offline`.

## Code Style

- **Line length**: 120 characters (black and ruff)
- **Formatter**: black (profile used by isort)
- **Linter**: ruff with `select = ["ALL"]` and specific ignores
- **Type checking**: mypy with strict settings, currently enforced on `rdagent/core/` only
- **Import sorting**: isort with black profile
- **TOML**: toml-sort on pyproject.toml

CI runs `make lint docs-gen test-offline` on Python 3.10 and 3.11.

## Commit Convention

PR titles and commits must follow conventional commits with max 100 char header:
`build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`

## Architecture

### Four-Layer Design

1. **`rdagent/core/`** — Abstract base classes and interfaces only. Key abstractions: `Scenario`, `Experiment`, `ExperimentPlan`, `Task`, `Hypothesis`, `Developer`, `EvolvingStrategy`, `KnowledgeBase`, `Feedback`. Strict mypy enforced here.

2. **`rdagent/components/`** — Reusable concrete implementations of core abstractions: `agent/` (LLM agents with RAG/MCP), `coder/` (code generation), `runner/` (experiment execution), `proposal/` (hypothesis generation), `workflow/` (orchestration), `knowledge_management/`, `document_reader/`.

3. **`rdagent/scenarios/`** — Domain-specific implementations: `qlib/` (quantitative trading factors & models), `data_science/` (ML engineering), `general_model/` (paper-to-implementation), `kaggle/` (competition automation), `shared/` (cross-scenario utilities).

4. **`rdagent/app/`** — End-to-end applications wiring scenarios together. These are removable without breaking the system. CLI entry point: `rdagent/app/cli.py` (Typer-based, registered as `rdagent` console script).

### Supporting Systems

- **`rdagent/oai/`** — LLM interaction layer. Default backend: LiteLLM (`rdagent.oai.backend.LiteLLMAPIBackend`). Configuration via `LLMSettings` in `rdagent/oai/llm_conf.py`.
- **`rdagent/log/`** — Structured logging with Streamlit-based UI for monitoring.
- **`rdagent/utils/`** — Shared utilities including Docker environment management, workflow helpers.

### Configuration System

Settings use `pydantic-settings` with `ExtendedBaseSettings` (defined in `rdagent/core/conf.py`). This custom base class chains env sources from parent settings classes. Priority: init values > env vars > parent class env vars > .env file > defaults. The CLI loads `.env` from CWD at startup via `dotenv`.

### CLI Commands

```bash
rdagent fin_factor          # Evolve trading factors
rdagent fin_model           # Evolve trading models
rdagent fin_quant           # Joint factor & model optimization
rdagent fin_factor_report   # Extract factors from financial reports
rdagent general_model <url> # Implement models from papers
rdagent data_science        # ML engineering automation
rdagent ui [--port 19899]   # Launch Streamlit monitoring UI
rdagent health_check        # Verify Docker & environment
```
