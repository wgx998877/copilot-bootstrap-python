# AGENTS.md — {{PROJECT_NAME}}

<!-- This file is the primary persistent guidance for GitHub Copilot in this project. -->
<!-- Rewrite every section to reflect the actual project. Do not leave template language. -->

## Project Framing

### What this project does

{{PROJECT_PURPOSE}}

### What this project is not trying to solve (yet)

{{NON_GOALS}}

## Architecture

### Current shape

- Single-package layout under `src/{{package_name}}/`
- Entrypoint in `main.py`, core logic in `core.py`
- Flat module structure — no sub-packages unless complexity demands it

### Principles

- Start with the narrowest architecture that satisfies current needs
- Separate core logic from IO — keep domain logic testable
- Isolate side effects (file reads, API calls, database access)
- No abstract base classes before two concrete implementations exist
- Prefer reversible decisions

## Dependency Discipline

- Prefer the standard library when it covers 80% of the need
- Every added dependency must have a clear operational benefit
- Avoid framework gravity — do not let one library dictate architecture
- Keep `pyproject.toml` dependencies intentional and few

## Code Structure

- Separate entrypoints from core logic
- Avoid deep nesting — prefer flat, explicit module boundaries
- Functions over classes unless state management is genuinely needed
- Clarity beats cleverness

## Development Rules

- Use `uv` for dependency management
- Use `pytest` for testing
- Run `uv run pytest` to validate changes
- Keep tests proportional — do not write tests for code that doesn't exist yet
- One smoke test at minimum; grow test coverage with the code

## Change Philosophy

- Prioritize reversible decisions
- Do not optimize prematurely
- Preserve simplicity as the project grows
- Every file and abstraction must justify its existence
- Make future change easy, but do not pre-build for hypothetical scale

## Quality Bar

- Code should be readable on first pass
- No dead code, no junk files
- Honest TODOs over fake precision
- The repo should feel intentionally small

## Known Unknowns

- TODO: {{KNOWN_UNKNOWN_1}}
- TODO: {{KNOWN_UNKNOWN_2}}
- TODO: {{KNOWN_UNKNOWN_3}}

<!-- Populate from genuine unknowns in the user's brief. Remove this comment. -->

## Open Decisions

- TODO: {{OPEN_DECISION_1}}
- TODO: {{OPEN_DECISION_2}}

<!-- These are design choices that have not yet been made. Remove this comment. -->

## Copilot Customization

This project uses `AGENTS.md` (this file) as the primary guidance.

If additional customization is added later, use these paths:
- `.github/skills/<name>/SKILL.md` — on-demand workflow skills
- `.github/prompts/<name>.prompt.md` — reusable prompt templates
- `.github/instructions/<name>.instructions.md` — file-specific instructions
