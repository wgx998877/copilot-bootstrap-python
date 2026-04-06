# AGENTS.md — {{PROJECT_NAME}}

<!-- This file is the primary persistent guidance for GitHub Copilot in this project. -->
<!-- Rewrite every section to reflect the actual project. Do not leave template language. -->

## What This Project Does

{{PROJECT_PURPOSE}}

## What This Project Does Not Do (Yet)

{{NON_GOALS}}

## Architecture

<!-- Describe the actual project structure. Examples by type:
     Library:  __init__.py (public API) + <domain>.py (logic, e.g. parser.py, encoder.py)
     CLI:      __main__.py (entry) + main.py (arg parsing, IO) + <domain>.py (logic)
     API:      app.py (routes) + <domain>.py (business logic)
     Research: <experiment>.py (experiment logic) + scripts/run_experiment.py
     Data:     main.py (IO) + <domain>.py (transformation logic)
     Name modules after what they do, not generically. Adjust to match the actual project. -->

- Single-package layout under `src/{{package_name}}/`
- `__init__.py` — package marker; keep minimal, re-export public API only
- {{ENTRYPOINT_DESCRIPTION}}
- {{CORE_LOGIC_DESCRIPTION}}
- `tests/test_smoke.py` — verifies import + basic happy path; grow with the code
- Flat module structure — no sub-packages unless complexity demands it

<!-- If the package should be runnable via `python -m {{package_name}}`, add __main__.py:
     from {{package_name}}.main import main
     main()
-->

## Key Interfaces

<!-- Describe the primary ways users interact with this project. -->
<!-- Examples: CLI commands, public API functions, input/output formats. -->

- TODO: {{KEY_INTERFACE_1}}
- TODO: {{KEY_INTERFACE_2}}

## Dependencies

<!-- List chosen dependencies and why. Remove if none. -->

None yet. Prefer stdlib unless the brief justifies a dependency.

## Testing Strategy

- Run tests: `uv run pytest`
- Smoke test verifies import and basic happy path
- Grow coverage as the code grows — don't test code that doesn't exist yet

<!-- Adjust testing approach for this project. E.g., mock external APIs, test data fixtures. -->

## Development Guidelines

- **Separate IO from logic.** Keep IO/wiring in the entrypoint; keep core logic pure and testable.
- **Prefer stdlib.** Only add a dependency when it has a clear benefit over stdlib.
- **Functions over classes** unless state management is genuinely needed.
- **No premature abstraction.** No ABCs before two implementations, no config systems before hardcoded values are a problem.
- **Clarity beats cleverness.** Code should be readable on first pass.
- **Honest TODOs over fake precision.** Mark unknowns explicitly; don't invent answers.

### Pythonic Conventions

- Use type hints on public function signatures.
- Use `pathlib.Path` over `os.path` for file operations.
- Use context managers (`with`) for resource handling (files, connections).
- Use f-strings for string formatting.
- Use `dataclasses` or `NamedTuple` for structured data instead of plain dicts or tuples.
- Follow naming: `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE` for constants.
- Use `if __name__ == "__main__"` guard in runnable modules.

## Known Unknowns

<!-- Genuine uncertainties from the brief. Remove when resolved. -->

- TODO: {{KNOWN_UNKNOWN_1}}
- TODO: {{KNOWN_UNKNOWN_2}}
