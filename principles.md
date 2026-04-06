# Principles

Engineering judgment and process rules for project initialization.

## Initialization Philosophy

Project initialization is a **structured first draft**, not a finished design. The user's brief is the primary source of truth.

**Authority order:** user brief → explicit constraints → bootstrap guidance → fallback defaults.

**Rules:**
1. **Adapt, don't copy.** Select, rewrite, trim, and combine patterns based on the actual brief.
2. **The brief is sovereign.** Never override the user's description with bootstrap defaults.
3. **Start small.** One package layout, one manifest, one or two code files, one test, one guidance file.
4. **Preserve uncertainty.** If a decision can't be grounded in the brief, leave a concrete TODO.
5. **No fake completeness.** Honest incompleteness beats polished fiction.
6. **Every file must justify itself.** Create it because the project needs it, not because a template says so.
7. **Guidance over boilerplate.** A well-written AGENTS.md is more valuable than skeleton files.

## Minimality

**Always include:** `AGENTS.md`, `README.md`, `pyproject.toml`, `.gitignore`, `src/<pkg>/__init__.py`, one or two source modules (per playbook), `tests/test_smoke.py`.

**Sometimes include (only if justified):** `.python-version`, `scripts/` helper, `.github/skills/`.

**Never include by default:** CI, Docker, deployment configs, complex config systems, elaborate docs, benchmarking, release automation, multiple skills or agents.

**The test:** Does the user's brief justify this file existing right now? If "not really" — don't create it.

## Architecture

- Start with the **narrowest architecture** that satisfies current needs.
- **Separate IO from logic** early. The entrypoint handles IO, arg parsing, and wiring. Logic modules stay pure and testable. Name modules after what they do (e.g. `parser.py`, `solver.py`), not generically (`core.py`, `utils.py`).
- **One clean happy path first** — before edge cases, generality, or plugin systems.
- **No premature layers** — no ABCs before two implementations, no plugin systems before one stable path, no config frameworks before hardcoded values are a problem.
- **Isolate side effects** — functions that touch files, APIs, or databases should be identifiable and contained.
- **Prefer reversible decisions** — flat structures, simple functions, hardcoded values. All can be upgraded later.
- **Stable interfaces over internal elegance** — get signatures, CLI args, and formats right. Internals can be refactored.

## Dependencies

- Prefer the standard library when it covers 80% of the need.
- Every dependency is a commitment: maintenance, API surface, version compat, transitive deps, security.
- Avoid framework gravity — don't let one library dictate architecture.
- Keep `pyproject.toml` dependencies intentional and few.

## Code Quality

- **Clarity beats cleverness.** Readable on first pass. No metaprogramming, deep inheritance, or "elegant" patterns that require explanation.
- **Prefer the narrowest solution.** Function > class > framework > service.
- **Correctness first, then clarity, then performance.** Only optimize with evidence.
- **Separate entrypoints from logic.** The entrypoint parses input, calls core, formats output. Core is importable and testable.
- **Tests: proportional.** One smoke test at init. Grow coverage with the code.

### Pythonic Patterns

- Type hints on public function signatures — aids both readers and Copilot.
- `pathlib.Path` over `os.path`. Context managers (`with`) for resource handling.
- `dataclasses` or `NamedTuple` for structured data instead of plain dicts.
- f-strings for formatting. `snake_case` / `PascalCase` / `UPPER_SNAKE` naming.
- `if __name__ == "__main__"` guard in runnable modules.

### Modern Package Structure

- `__init__.py` — keep minimal: re-export public API, nothing else.
- `__main__.py` — enables `python -m <package>`. Add when the package should be directly runnable. Contains only: `from <pkg>.main import main; main()`.
- `py.typed` marker file — add if the library exports type hints for consumers.

## TODO Policy

TODOs are a **first-class feature**. They mark where the brief didn't provide enough information.

**Format — concrete and decision-oriented:**
```
TODO: confirm storage choice — SQLite for local use vs. PostgreSQL if deployed
TODO: define CLI command surface after first usage patterns emerge
TODO: clarify expected input schema (JSON lines? CSV? single JSON object?)
```

**Where:** README.md → "Open Questions". AGENTS.md → "Known Unknowns". Code → inline comments.

**Anti-patterns:** vague TODOs (`fix this later`), TODOs for things already decided, replacing uncertainty with invented answers.
