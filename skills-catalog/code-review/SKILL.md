---
name: code-review
description: "Review code changes or audit overall project health. Use when reviewing PRs, diffs, refactoring, security auditing, evaluating scope drift, trimming unnecessary code, auditing dependencies, or doing periodic project shaping."
---

# Code Review

Two modes: **change review** (per-PR/diff) and **project shaping** (periodic repo audit). Use whichever fits the task.

---

## Mode 1: Change Review

Review a specific change (PR, diff, or set of modifications).

### Procedure

1. Read `AGENTS.md` to understand project constraints and current architecture.
2. Identify every file touched. For each, answer: does this change fit the project's stated scope?
3. Walk through the diff function-by-function. Apply the checks below.
4. Classify issues as **blocking** (must fix) or **suggestion** (should consider).
5. Note good choices explicitly — positive signal matters.

### Architecture Check

- **Narrowest solution test:** Could this be a function instead of a class? A module instead of a package? If yes, it should be.
- **IO boundary:** Does new code mix logic with IO? Search for `open(`, `requests.`, `print(`, `sys.stdout` inside files that should be pure logic. Flag them.
- **Import direction:** Core modules should never import from entrypoints, CLI, or IO modules. Check `from` and `import` lines.
- **New files:** Every new file must answer "why does this exist separately?" in one sentence. If it can't, merge it.

### Dependency Check

When `pyproject.toml` is modified:
- **Justify:** What does this dependency do that stdlib can't? Be specific.
- **Size check:** Run `pip show <pkg>` — check `Requires` field. How many transitive deps does it pull?
- **Alternatives:** Is there a lighter option? (e.g., `httpx` vs. `requests` vs. `urllib3` vs. `urllib.request`)
- **Pin width:** Use `>=` lower bound, not `==` exact pins, unless reproducibility demands it.

### Code Quality Check

- **Naming:** Read function/variable names aloud. If you need to read the body to understand what it does, the name is wrong.
- **Nesting depth:** More than 3 levels of indentation → extract a function or invert the condition with early return.
- **Dead code:** Search for unused imports (`ruff check --select F401`), unreachable branches, commented-out blocks. Remove them.
- **Magic values:** Hardcoded numbers or strings repeated more than once → named constant.
- **Error handling:** `except Exception` is almost always wrong. Catch the specific error. Check that error messages include actionable context (what happened, what input caused it, what to do).

### Security Check

Run the combined grep, then check each match against the table:

```bash
grep -rn "password\|secret\|api_key\|token\|eval(\|exec(\|shell=True\|os\.system\|pickle\.load\|yaml\.load\|verify=False\|tempfile\.mktemp" --include="*.py" .
```

| Pattern | Risk | Fix |
|---------|------|-----|
| `password`, `secret`, `api_key`, `token` | Hardcoded credentials | Environment variable or secrets manager |
| `eval(`, `exec(`, `compile(` | Code injection | `ast.literal_eval()`, `json.loads()`, explicit parsing |
| `shell=True`, `os.system`, `os.popen` | Command injection | `subprocess.run([...])` with list args |
| `pickle.load`, `yaml.load`, `marshal.load` | Unsafe deserialization | `json`, `yaml.safe_load`, `tomllib` |
| `verify=False`, `http://` | Insecure network | Enable TLS verification, use `https` |
| `tempfile.mktemp()` | Race condition | `tempfile.NamedTemporaryFile()` or `mkstemp()` |
| f-string/`.format()` in SQL `execute()` | SQL injection | Parameterized queries (`?` placeholders) |
| User input in `open()`/`Path()` without validation | Path traversal | `Path.resolve()` + verify under expected dir |

Secrets and injection matches are always **blocking**. Network and path issues are blocking when handling user input.

### Test Check

- **Coverage direction:** New logic should have at least one test exercising the happy path. Edge cases for anything that handles user input or external data.
- **Test quality:** Tests should assert behavior ("given X, returns Y"), not implementation ("calls method Z"). If a test breaks when you refactor internals without changing behavior, it's a bad test.
- **Proportionality:** A 10-line function doesn't need 100 lines of tests. Match test effort to risk.

### Output Format

```
## Blocking
- file.py:42 — Issue. Concrete fix or alternative.

## Suggestions
- file.py:15 — Observation. Why it matters. Alternative.

## Good
- Specific positive observations about the change.
```

---

## Mode 2: Project Shaping

Periodic audit of the entire repository. Run after a batch of changes, before milestones, or when the project feels overgrown.

### Procedure

1. Read `AGENTS.md` and `README.md` — restate the project's purpose in one sentence.
2. List all files. For each: is this file actively used? Does it serve the stated purpose?
3. Check dependency health.
4. Audit TODOs.
5. Produce a shaping report.

### Scope Alignment

- Compare current code against the "What this project does" section of AGENTS.md. Has the project drifted?
- List any modules or features that don't serve the core purpose. These are trim candidates.
- If the project has genuinely grown, update AGENTS.md to reflect reality — don't let guidance and code diverge.

### Structural Audit

- **File count test:** For a small project, can you explain every file's purpose to someone in 60 seconds? If not, there are too many files.
- **Abstraction test:** For every class, ask: does this have more than one concrete subclass? If no, it probably shouldn't be an ABC.
- **Layer test:** Can you trace a request from input to output touching 3 or fewer modules? If not, there are too many layers.
- **Dead file test:** `grep -r "import <module>"` — if a module isn't imported anywhere (and isn't an entrypoint), consider removing it.

### Dependency Audit

```bash
# List all declared dependencies
grep -A 20 'dependencies' pyproject.toml
# For each, check if it's actually imported
grep -r "import <pkg>" src/
# Check for unused deps
```

Remove any dependency not actively imported. Demote dev-only deps to `[project.optional-dependencies]`.

### TODO Audit

```bash
# Find all TODOs
grep -rn "TODO" --include="*.py" --include="*.md" .
```

For each TODO:
- Still relevant? → Keep.
- Decision has been made? → Resolve and remove.
- Stale or vague? → Rewrite or remove.
- New uncertainty discovered? → Add a new TODO.

### Shaping Report Format

```
## Project: <name>
## Purpose: <one sentence>

### Scope: [on track / drifted]
- Observations.

### Trim candidates
- Files or modules to remove or merge.

### Dependency changes
- Remove: <pkg> (reason)
- Keep: <pkg> (justification)

### TODO updates
- Resolve: <TODO> (decision)
- Add: <new TODO>
- Remove: <stale TODO>

### Recommended actions (priority order)
1. ...
2. ...
```
