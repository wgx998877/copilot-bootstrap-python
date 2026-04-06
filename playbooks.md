# Playbooks

Select the section matching the user's project type.

## Project Type Decision Tree

- Importable package, no CLI → **Tiny Library**
- Terminal command with args → **CLI Tool**
- HTTP endpoints → **Mini API**
- Input → transform → output (files/data) → **Local Data Tool**
- Hypothesis-driven, iterative → **Research Prototype**

If the brief spans multiple types, pick the dominant one and note the overlap as a TODO.

---

## Tiny Library

**When:** Small, reusable Python library. Importable package with a public API, minimal CLI.

**Structure:** `__init__.py` (public API re-exports) + `<domain>.py` (logic — name after what it does, e.g. `parser.py`, `encoder.py`). No `main.py` unless brief mentions CLI. Add `py.typed` if publishing type hints.

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

**Structure:** `__init__.py` + `__main__.py` (enables `python -m <pkg>`) + `main.py` (arg parsing, IO) + `<domain>.py` (logic, no IO — e.g. `solver.py`, `converter.py`). Tests in `tests/`.

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

**Structure:** `__init__.py` + `<experiment>.py` (experiment logic) + optional `utils.py` + optional `scripts/run_experiment.py`. Tests in `tests/`.

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

**Structure:** `__init__.py` + `<domain>.py` (transformation logic, e.g. `transform.py`, `pipeline.py`) + `main.py` (entrypoint, IO). Optional `__main__.py` if runnable via `python -m`. Tests in `tests/`.

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

**Structure:** `__init__.py` + `app.py` (routes, API definition) + `<domain>.py` (business logic, no framework deps, e.g. `models.py`, `service.py`). Tests in `tests/`.

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
