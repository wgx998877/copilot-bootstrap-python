---
name: release-prep
description: "Prepare a Python project for first release or version bump. Use when publishing to PyPI, cutting a release, setting up versioning, writing changelogs, or reviewing package metadata."
---

# Release Prep

Prepare a small Python project for release — PyPI publishing, versioning, metadata, packaging checks.

## Procedure

### 1. Verify package metadata

Check `pyproject.toml` has all required fields:

```toml
[project]
name = "my-package"              # PyPI name (lowercase, hyphens)
version = "0.1.0"                # semver
description = "One-line summary" # required for PyPI
readme = "README.md"
requires-python = ">=3.10"
license = {text = "MIT"}         # or appropriate license
authors = [{name = "Name", email = "email@example.com"}]
dependencies = [...]

[project.urls]
Homepage = "https://github.com/..."
Issues = "https://github.com/.../issues"
```

**Common mistakes:**
- Missing `license` — PyPI will accept it but users won't trust it
- Wrong `requires-python` — test on the minimum version you claim
- `name` conflicts — check `pip index versions <name>` before publishing
- Missing `description` — shows as blank on PyPI

### 2. Verify package builds cleanly

```bash
# Install build tool
uv pip install build
# Build sdist and wheel
uv run python -m build
# Check the built package
ls dist/
# Inspect wheel contents — make sure only intended files are included
unzip -l dist/*.whl | head -30
```

**Check for problems:**
- Are test files included? They shouldn't be in the wheel.
- Are large data files included? Check `[tool.hatch.build]` or `MANIFEST.in` to exclude them.
- Is the package importable from the built wheel?

```bash
# Test install from the built wheel
uv pip install dist/*.whl --force-reinstall
uv run python -c "import <package_name>; print(<package_name>.__version__)"
```

### 3. Verify the public API

```python
# What does your package export?
import <package_name>
print(dir(<package_name>))
# or check __init__.py
```

**Checklist:**
- [ ] `__init__.py` exports only intended public names
- [ ] All public functions have type hints on parameters and return values
- [ ] All public functions have docstrings with at least: one-line summary, parameters, return value
- [ ] No internal helpers accidentally exposed

### 4. Test on clean install

```bash
# Create a fresh venv and test
uv venv /tmp/release-test && source /tmp/release-test/bin/activate
pip install dist/*.whl
python -c "from <package_name> import <main_function>; print('OK')"
# Run a basic usage scenario
deactivate && rm -rf /tmp/release-test
```

### 5. Security scan

```bash
# Secrets — must be clean before publishing
grep -rn "password\|secret\|api_key\|token\|private_key" src/ | grep -v "#\|TODO\|mock\|fake"
# Dangerous patterns — each match is a release blocker unless justified
grep -rn "eval(\|exec(\|pickle\.load\|shell=True\|verify=False\|yaml\.load" src/
# Dependency vulnerabilities
uv pip install pip-audit && uv run pip-audit
# Files that shouldn't ship
find . -name ".env*" -o -name "*.pem" -o -name "*.key"
```

### 6. Version strategy

For small projects, use simple semver:

| Change type | Version bump | Example |
|------------|-------------|---------|
| Bug fix, no API change | Patch | 0.1.0 → 0.1.1 |
| New feature, backward compatible | Minor | 0.1.1 → 0.2.0 |
| Breaking API change | Major | 0.2.0 → 1.0.0 |
| Pre-1.0 anything goes | Minor for breaking | 0.1.0 → 0.2.0 |

**Where to put the version:**
```toml
# Option A: in pyproject.toml (simple, recommended for small projects)
[project]
version = "0.1.0"

# Option B: dynamic from __init__.py
[project]
dynamic = ["version"]
[tool.hatch.version]
path = "src/<package_name>/__init__.py"
```

### 7. Write a changelog

For first release, keep it simple:

```markdown
# Changelog

## 0.1.0 (2024-01-15)

Initial release.

- Core functionality: <what it does>
- CLI entrypoint: `<command>` (if applicable)
- Tested on Python 3.10+
```

For subsequent releases:
```markdown
## 0.2.0 (2024-02-01)

### Added
- New function: `process_batch()` for bulk operations

### Changed
- `transform()` now returns a dataclass instead of a dict (breaking)

### Fixed
- Handle empty input without crashing (#12)
```

### 8. Publish

```bash
# Test publish to TestPyPI first
uv pip install twine
uv run twine upload --repository testpypi dist/*

# Verify on TestPyPI
pip install --index-url https://test.pypi.org/simple/ <package_name>

# Real publish
uv run twine upload dist/*
```

**Pre-publish checklist:**
- [ ] All tests pass: `uv run pytest`
- [ ] Package builds: `uv run python -m build`
- [ ] Clean install works from built wheel
- [ ] Version bumped in `pyproject.toml`
- [ ] Security scan passed: `uv run pip-audit` (no unaccepted vulnerabilities)
- [ ] No secrets in source: `grep -rn "password\|secret\|api_key\|token\|private_key" src/ tests/ | grep -v "#\|TODO\|mock\|fake"`
- [ ] Changelog updated
- [ ] README is accurate and up to date
- [ ] No TODOs that block release (check: `grep -rn "TODO.*block" .`)

## Anti-Patterns

- Publishing without testing install from the built wheel
- Version 1.0.0 on first release when the API isn't stable (start at 0.1.0)
- `MANIFEST.in` that accidentally includes test data or `.env` files
- No changelog — users can't tell what changed between versions
- Publishing to PyPI as the first test (use TestPyPI first)