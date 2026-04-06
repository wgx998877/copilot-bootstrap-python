---
name: experiment-review
description: "Review research and prototype code for experimental clarity, reproducibility, and separation of concerns. Use when reviewing experiments, checking reproducibility, promoting experiment code, or cleaning up research repos."
---

# Experiment Review

Review research and prototype code. Enforce experimental clarity, reproducibility, and clean separation between experiment scripts and reusable logic.

## Procedure

### 1. Map the experiment landscape

```bash
# Find experiment scripts
find . -name "*.py" -path "*/scripts/*" -o -name "run_*.py" -o -name "experiment_*.py"
# Find core logic
find src/ -name "*.py" | head -20
# Find results
find . -name "*.csv" -o -name "*.json" -o -name "results*" | head -20
```

Questions to answer:
- Which files are experiments? Which are reusable?
- Where do results live?
- Can you tell what each experiment tests from its filename alone?

### 2. Check experiment clarity

Every experiment script should answer these questions **in its first 10 lines** (comments or docstring):

```python
"""Experiment: Does attention entropy correlate with cyclomatic complexity?

Hypothesis: Higher attention entropy → higher cyclomatic complexity.
Dataset: 500 Python functions from OSS repos.
Key params: model=codegen-350M, layers=[4,8,12], metric=shannon_entropy.
Output: results/entropy_correlation.csv
"""
```

**Red flags:**
- Script named `test3.py` or `new_approach.py` with no description
- Parameters buried inside functions instead of at the top
- No way to tell what the experiment does without reading 100+ lines

### 3. Check separation of concerns

**Good:**
```
src/attn_complexity/
    core.py          # extract_attention(), compute_entropy() — pure, testable
    utils.py         # load_dataset(), save_results() — IO helpers
scripts/
    run_entropy.py   # thin: parse args, call core, save results
```

**Bad:**
```
experiment.py        # 500 lines: loads data, runs model, computes metrics, saves CSV, makes plots
```

**Specific checks:**
- Can you `from pkg.core import function` and call it in a test without any file/network IO? If not, extract the pure logic.
- Is `scripts/run_*.py` thin? It should be: load data → call core → save. Under 50 lines for simple experiments.
- Are experiment-specific hacks marked? `# HACK: workaround for tokenizer bug in v2.1`

### 4. Check reproducibility

```bash
# Random seeds — should exist in every experiment script
grep -rn "seed\|random_state\|manual_seed" scripts/ src/
# Data sources — should be pinned or documented
grep -rn "data_path\|dataset\|DATA_DIR" scripts/
```

**Reproducibility checklist:**
- [ ] Random seeds set and passed through all randomness sources (Python `random`, `numpy`, `torch`)
- [ ] Data source documented (path, version, download command, or generation script)
- [ ] Dependency versions pinned in `pyproject.toml` (exact pins OK for research)
- [ ] Key parameters saved alongside results (not just in the script)

**Parameter logging pattern:**
```python
import json
params = {"model": "codegen-350M", "layers": [4, 8, 12], "seed": 42, "n_samples": 500}
# Save params with results
with open("results/entropy_run_001.json", "w") as f:
    json.dump({"params": params, "results": results}, f, indent=2)
```

### 5. Check results management

- **No silent overwrites:** Results should include a timestamp, run ID, or be in dated directories. `results.csv` that gets overwritten every run is unacceptable.
- **Structured format:** JSON or CSV with headers, not print statements piped to a file.
- **Comparison support:** Can you compare two runs by reading their result files? Include params in the output.

**Good pattern:**
```python
from datetime import datetime
run_id = datetime.now().strftime("%Y%m%d_%H%M%S")
output_path = f"results/entropy_{run_id}.json"
```

### 6. Decide: keep rough or promote?

Research code has different quality bars:

| Layer | Quality bar |
|-------|------------|
| Experiment scripts (`scripts/`) | Rough-but-readable. Hardcoded params OK if visible. Print debugging OK. |
| Core logic (`src/pkg/core.py`) | Clean and testable. Would survive a code review. |
| Utilities (`src/pkg/utils.py`) | Correct but not necessarily elegant. |

When an experiment stabilizes and the code is worth keeping:
1. Extract proven logic from scripts into `core.py`
2. Add tests for the extracted functions
3. Archive or delete the raw experiment script
4. Update AGENTS.md to reflect what's now stable

## Anti-Patterns

- One 800-line script that does everything (load, compute, plot, save)
- `results.csv` overwritten on every run with no record of parameters
- Core logic that only works when called from within the experiment script's global state
- "I'll clean this up later" but the experiment has been running for 3 months
- Installing packages globally with `pip install` instead of in the project venv
