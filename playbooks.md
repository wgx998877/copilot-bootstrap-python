# Playbooks

Select the section matching the user's project type.

---

## Tiny Library

**When:** Small, reusable Python library. Importable package with a public API, minimal CLI.

**Structure:** `src/<pkg>/__init__.py` + `core.py`. No `main.py` unless brief mentions CLI.

**pyproject.toml:** Minimal `requires-python`, empty or near-empty `dependencies`. Consider `[project.optional-dependencies]` for dev extras.

**AGENTS.md focus:** Public API surface, what the library does/doesn't do, versioning intentions, backward compatibility.

**Typical TODOs:**
```
TODO: define the public API surface (__init__.py exports)
TODO: decide minimum Python version support
TODO: clarify whether this library will be published to PyPI or remain internal
```

**Avoid:** CLI entrypoints (unless requested), plugin systems, framework integrations, heavy dependencies.

---

## CLI Tool

**When:** Command-line tool invoked from terminal. Takes arguments, produces output or side effects.

**Structure:** `src/<pkg>/main.py` (entrypoint: arg parsing, IO) + `core.py` (logic: no IO, testable).

**pyproject.toml:** Add `[project.scripts]` entry: `<tool-name> = "<pkg>.main:main"`. Use `argparse` (stdlib) unless brief specifies `click`/`typer`.

**AGENTS.md focus:** CLI command surface (subcommands, flags), input/output formats, error-handling expectations, interactive vs. batch.

**Typical TODOs:**
```
TODO: define CLI command surface — subcommands, flags, positional args
TODO: decide output format (human-readable, JSON, both)
TODO: clarify error-handling policy (fail fast vs. collect and report)
```

**Avoid:** Complex config systems, fancy output formatting, plugin architectures, daemon modes.

---

## Research Prototype

**When:** Experimental/research codebase. Emphasis on exploration, iteration speed, and clarity of experimental logic.

**Structure:** `src/<pkg>/core.py` (experiment logic) + optional `utils.py` + optional `scripts/run_experiment.py`.

**pyproject.toml:** Pin key experiment dependencies. Use `[project.optional-dependencies]` for heavier deps.

**AGENTS.md focus:** Research question/hypothesis, separation of experiment vs. utility code, reproducibility (seeds, data versioning), which parts are stable vs. changing. Permit rough code in experiment scripts, enforce clarity in core logic.

**Typical TODOs:**
```
TODO: define the primary experiment entry point and parameters
TODO: decide on data input format and source
TODO: determine reproducibility requirements (random seeds, data snapshots)
TODO: decide whether results go to files, database, or stdout
```

**Avoid:** Production-grade error handling in experiments, complex config abstractions, premature optimization, building a "framework" before one experiment works.

---

## Local Data Tool

**When:** Data processing, transformation, or analysis tool. Input → transform → output. Typically runs locally.

**Structure:** `src/<pkg>/core.py` (transformation logic) + `main.py` (entrypoint, IO). Optional `data/` directory.

**pyproject.toml:** Add data-processing deps only when justified (pandas, polars). Consider `[project.scripts]` if tool has CLI.

**AGENTS.md focus:** Expected input format(s)/source(s), output format/destination, performance expectations, data size expectations, read-only vs. in-place vs. new-file output.

**Typical TODOs:**
```
TODO: confirm expected input data format and schema
TODO: decide output format (CSV, JSON, Parquet, SQLite, stdout)
TODO: clarify expected data volume and performance requirements
TODO: determine whether streaming or batch processing is appropriate
```

**Avoid:** Generic ETL frameworks, over-abstracted data source adapters, database ORMs for file-based workflows, elaborate schema validation before schema is stable.

---

## Mini API

**When:** Small HTTP API or web service. Few endpoints (1–10), lightweight framework.

**Structure:** `src/<pkg>/app.py` (routes, API definition) + `core.py` (business logic, no framework deps).

**pyproject.toml:** Add chosen framework (`fastapi`, `flask`) + server (`uvicorn`). Keep minimal.

**AGENTS.md focus:** Endpoint surface (routes, methods, request/response shapes), internal vs. external traffic, auth requirements, persistence approach, error response format.

**Typical TODOs:**
```
TODO: define API endpoint surface (routes, methods, payloads)
TODO: decide on authentication approach (none, API key, OAuth)
TODO: clarify persistence — in-memory, SQLite, external database
TODO: determine error response format and status code conventions
```

**Avoid:** Docker/Kubernetes at init, complex middleware, ORM before data model is stable, API versioning before v1 exists.
