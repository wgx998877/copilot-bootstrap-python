---
name: demo-build
description: "Build lightweight demos, proof-of-concepts, and visualization layers. Use when creating demos, showcases, interactive examples, or adding a simple UI to show what a project does."
---

# Demo Build

Build clean, minimal demos that showcase a project's capabilities without polluting the core codebase.

## Procedure

### 1. Understand what to demonstrate

Read `AGENTS.md` and the core source. Identify:
- What is the most impressive or useful thing this project does?
- What's the simplest input → output path that shows it working?
- Who is the audience? (developer, stakeholder, user)

### 2. Choose the right demo format

| Audience | Best format |
|----------|------------|
| Developers | CLI script that exercises core functions with sample data |
| Stakeholders | Streamlit/Gradio app with visible input/output |
| Data people | Jupyter notebook with inline results |
| API consumers | `curl` examples or a simple HTML page |
| General | Terminal recording (asciinema) or screenshot sequence |

**Decision tree:**
- Can the demo be one script under 100 lines? → `scripts/demo.py`
- Does it need a UI? → `demos/<name>/app.py` with Streamlit/Gradio
- Does it need narrative? → Jupyter notebook in `demos/`
- Is it just API calls? → `demos/README.md` with `curl` examples

### 3. Build the demo

**Structure for script demos:**
```python
# scripts/demo.py
"""Demo: <what this shows in one sentence>

Run: uv run python scripts/demo.py
"""
from <package> import <core_function>

# Sample input — self-explanatory, no external files needed
sample = ...

# Run the core logic
result = core_function(sample)

# Display results clearly
print(f"Input:  {sample}")
print(f"Output: {result}")
```

**Structure for app demos:**
```
demos/<name>/
├── README.md       # one-command run instruction + what it shows
├── app.py          # demo entrypoint
└── sample_data/    # only if needed, keep small
```

**Key implementation rules:**
- Import from the project's package. Never copy-paste core logic into demo code.
- Demo-specific dependencies go in `[project.optional-dependencies]` under a `demo` group:
  ```toml
  [project.optional-dependencies]
  demo = ["streamlit>=1.0"]
  ```
- Use sample/synthetic data, never production data. Inline small samples directly in the script.
- Handle missing optional deps gracefully:
  ```python
  try:
      import streamlit as st
  except ImportError:
      sys.exit("Install demo deps: uv pip install -e '.[demo]'")
  ```

### 4. Make it runnable in one command

Every demo README must start with:
```
## Run
uv run python scripts/demo.py
# or
uv run streamlit run demos/<name>/app.py
```

If setup is needed, script it. If it takes more than one command, simplify.

### 5. Show expected output

Include in the demo README:
- What the user will see (terminal output, screenshot description, or URL)
- What the demo proves about the project
- Known limitations of the demo vs. full capabilities

## Anti-Patterns

- Demo code that imports from `demos/` into `src/` (wrong direction)
- Demos that require database setup, API keys, or network access to show basic functionality
- Over-styled UI when the point is to show logic
- Demos that silently fail or show tracebacks instead of helpful messages
- "Demo frameworks" — if you're building infrastructure to run demos, you've gone too far
