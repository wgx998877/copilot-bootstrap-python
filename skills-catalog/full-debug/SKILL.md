---
name: full-debug
description: "Systematically debug Python runtime failures. Use when diagnosing tracebacks, unexpected output, test failures, flaky tests, or performing root-cause analysis."
---

# Full Debug

The complete debugging workflow for Python runtime failures — from triage through verified fix. Use the full procedure for non-obvious bugs; skip to the relevant step when the cause is clear.

## Procedure

### 1. Reproduce the failure

Before debugging, make the failure reliable:

```bash
# Run the failing command and capture exact error
uv run python -m <package>.main <args> 2>&1 | tee /tmp/debug_output.txt
# Or for tests
uv run pytest tests/test_failing.py -x -v 2>&1 | tee /tmp/test_output.txt
```

If the failure is intermittent:
- Run it 10 times: `for i in $(seq 10); do uv run pytest tests/test_flaky.py -x && echo "PASS $i" || echo "FAIL $i"; done`
- Check for timing dependencies, random seeds, or external service flakiness
- Add `PYTHONHASHSEED=0` to eliminate hash randomization as a factor

### 2. Quick-check common gotchas

Before investing in deep investigation, rule out these frequent causes (30 seconds):

- **Mutable default arguments:** `def f(items=[])` — the list is shared across calls. Fix: `def f(items=None): items = items or []`
- **Late binding closures:** `lambdas = [lambda: i for i in range(3)]` — all return 2. Fix: `lambda i=i: i`
- **Shallow copy:** `copy = original[:]` doesn't deep-copy nested structures.
- **String vs. bytes:** Mixing `str` and `bytes` in file/network operations.
- **Relative imports:** Running a module directly vs. as part of a package changes import behavior.
- **Working directory:** `open("data.csv")` is relative to where you ran the command, not where the script lives. Use `Path(__file__).parent / "data.csv"`.

If one of these matches, fix and verify. Otherwise continue to step 3.

### 3. Read the traceback carefully

Python tracebacks read bottom-to-top. The actual error is the **last line**. The cause is usually in **your code's last frame** (not inside library code).

```
Traceback (most recent call last):
  File "src/pkg/main.py", line 42, in process    ← YOUR CODE (look here)
    result = transform(data)
  File "src/pkg/transform.py", line 18, in transform   ← YOUR CODE (root cause likely here)
    return data["key"]
KeyError: 'key'                                    ← ACTUAL ERROR
```

**Common errors and what they really mean:**
| Error | Usually means |
|-------|-------------|
| `KeyError` | Wrong key name, or data structure isn't what you expected. Print the actual keys. |
| `AttributeError: 'NoneType'` | A function returned `None` unexpectedly. Check the function that produced the value. |
| `TypeError: takes N args` | Calling with wrong # of args, or calling a non-callable. Check the call site. |
| `ValueError` | Input has the right type but wrong content. Check the value and any assumptions about format. |
| `RecursionError` | Unbounded recursion — missing base case or circular references. Check termination conditions. |
| `ImportError` | Wrong package name, missing dep, or circular import. Check `from` path. |
| `FileNotFoundError` | Wrong working directory or wrong relative path. Print `os.getcwd()` and the actual path. |

### 4. Isolate the problem

**Bisection approach** — narrow down to the smallest reproducing case:

```python
# Instead of debugging the full pipeline:
result = load_data() | transform() | validate() | save()

# Test each step independently:
data = load_data()
print(f"After load: type={type(data)}, len={len(data)}, sample={data[:2]}")

transformed = transform(data)
print(f"After transform: type={type(transformed)}, keys={transformed.keys() if isinstance(transformed, dict) else 'N/A'}")
# ... find which step breaks
```

**Minimal reproduction** — create the smallest input that triggers the bug:
```python
# Don't debug with full production data
# Find the minimal failing case
assert transform({"key": "value"}) == expected  # works
assert transform({}) == expected                  # fails! — empty input not handled
```

### 5. Inspect state at the failure point

**Quick inspection (print debugging):**
```python
# Strategic prints — always include the variable name and type
print(f"DEBUG data={data!r} type={type(data)}")
print(f"DEBUG keys={list(data.keys()) if hasattr(data, 'keys') else 'not a dict'}")
```

**Breakpoint debugging:**
```python
# Add at the suspicious line
breakpoint()  # drops into pdb

# In pdb:
# p variable      — print value
# pp variable     — pretty-print
# l               — show source around current line
# w               — show full call stack
# u/d             — move up/down the call stack
# n               — next line
# c               — continue
```

**Structured logging** (when print debugging isn't enough):
```python
import logging
logging.basicConfig(level=logging.DEBUG, format="%(name)s %(levelname)s %(message)s")
log = logging.getLogger(__name__)
log.debug("data=%r", data)  # lazy formatting — no f-string here
```

**For test failures:**
```bash
# Quick triage — short traceback, stop on first failure
uv run pytest tests/test_failing.py -x --tb=short
# Deep dive — full traceback with local variables
uv run pytest tests/test_failing.py -x --tb=long -l
# Drop into debugger on failure
uv run pytest tests/test_failing.py -x --pdb
```

### 6. Fix and verify

After finding the root cause:
1. Write a test that reproduces the bug (this test should fail right now)
2. Fix the code
3. Verify the test passes
4. Run the full test suite to check for regressions: `uv run pytest`

```bash
# Verify fix
uv run pytest tests/test_bug_fix.py -x -v
# Check no regressions
uv run pytest
```

### 7. Clean up

- Remove all `print(f"DEBUG ...")` statements
- Remove any `breakpoint()` calls
- Keep the regression test

## Anti-Patterns

- Adding `try/except: pass` to silence the error instead of fixing the cause
- Debugging by reading code instead of running it — always reproduce first
- Changing multiple things at once — change one thing, test, repeat
- Leaving debug prints in committed code
- "Works on my machine" without checking: Python version, dependencies, working directory, environment variables