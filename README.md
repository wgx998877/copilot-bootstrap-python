# copilot-bootstrap-python

You are initializing a new small Python project. This repository is your knowledge source. Read it, adapt it, do not copy it blindly.

## How to Use This Repository

1. Read [principles.md](principles.md) to calibrate your engineering judgment.
2. Read the matching section in [playbooks.md](playbooks.md) based on the project type.
3. Adapt the base files from `templates/` — rewrite every section for the specific project. Never leave template placeholders.
4. Decide whether any skills from the catalog are justified (usually: none).
5. Generate the project. Explain what you created, what you left as TODO, and what the user should clarify next.

> **One rule:** Initialize the smallest truthful project that fits the brief, and leave explicit TODOs where truth runs out.

## Initialization Algorithm

**Step 1 — Parse the brief.** Extract: project purpose, project type, constraints, preferred libraries, interface type, unknowns.

**Step 2 — Detect uncertainty.** Identify gaps. Do not invent answers. These become concrete, decision-oriented TODOs.

**Step 3 — Select project shape.** Use the matching playbook section. Default structure:

```
<project>/
├── AGENTS.md                          # primary Copilot guidance (root)
├── README.md
├── pyproject.toml
├── .gitignore
├── src/<package_name>/
│   ├── __init__.py                    # package marker, re-export public API
│   ├── __main__.py                    # enables `python -m <pkg>` (if runnable)
│   └── <modules per playbook>         # see playbooks.md for per-type structure
└── tests/
    └── test_smoke.py
```

Optional (only when justified):
```
├── .github/
│   └── skills/<name>/SKILL.md         # on-demand workflow skills
├── .python-version
└── scripts/<helper>.py
```

**Step 4 — Adapt templates.** Read from `templates/` and rewrite for this project.

**Step 5 — Add TODOs.** README.md → "Open Questions". AGENTS.md → "Known Unknowns". Code → inline comments.

**Step 6 — Decide on skills.** Default: none. See "When to Include Skills" below.

**Step 7 — Explain output.** What was created, what is TODO, what to clarify next.

## Repository Contents

| Path | What it is | When to read |
|------|-----------|--------------|
| [principles.md](principles.md) | Engineering judgment, minimality, architecture, TODO policy | Always — read before generating |
| [playbooks.md](playbooks.md) | Per-type guidance: library, CLI, API, data tool, research | Always — read the matching section |

### Templates — `templates/`

Adaptable base files. Rewrite completely for the target project.

| File | Generates |
|------|-----------|
| [AGENTS.base.md](templates/AGENTS.base.md) | Root `AGENTS.md` — primary Copilot guidance |
| [README.base.md](templates/README.base.md) | Project `README.md` |
| [pyproject.base.toml](templates/pyproject.base.toml) | `pyproject.toml` manifest |

### Skills Catalog — `skills-catalog/`

Skills are on-demand workflows with concrete procedures. Generated projects place them at `.github/skills/<name>/SKILL.md`.

| Skill | What it does |
|-------|-------------|
| [code-review](skills-catalog/code-review/SKILL.md) | Two modes: change review (per-PR) and project shaping (periodic repo audit) |
| [demo-build](skills-catalog/demo-build/SKILL.md) | Build lightweight demos, proof-of-concepts, and showcases |
| [experiment-review](skills-catalog/experiment-review/SKILL.md) | Review research code for clarity, reproducibility, separation of concerns |
| [debug](skills-catalog/debug/SKILL.md) | Systematic debugging: reproduce, isolate, diagnose, fix |
| [release-prep](skills-catalog/release-prep/SKILL.md) | Prepare for release: packaging, versioning, PyPI publishing |

### When to Include Skills

**Default: none.** Most project guidance belongs in AGENTS.md. Include a skill when the project would benefit from a repeatable, procedural workflow beyond normal coding.

**Decision guide:**

| Skill | Include when... |
|-------|----------------|
| code-review | The project will have ongoing changes reviewed (team project, iterative development, quality gates) |
| demo-build | The solution needs a visual or interactive showcase (grid visualization, CLI demo, stakeholder presentation) |
| experiment-review | The project involves research, hypothesis testing, or experimental iteration |
| debug | The logic is stateful, algorithmic, or involves hard-to-trace bugs (backtracking, async, data pipelines, external APIs) |
| release-prep | The project will be published as a package (PyPI, internal registry, versioned releases) |

Include multiple skills if the project warrants them. Omit all if none clearly apply.

## Copilot File Conventions

When generating files for the new project:

| What | Path |
|------|------|
| Primary Copilot guidance | `AGENTS.md` (root) |
| Skills | `.github/skills/<name>/SKILL.md` |

Other Copilot paths (use only when the project explicitly needs them):

| What | Path |
|------|------|
| Custom agents | `.github/agents/<name>.agent.md` |
| Prompts | `.github/prompts/<name>.prompt.md` |
| File-specific instructions | `.github/instructions/<name>.instructions.md` |

## Scope

**In scope:** utilities, automation scripts, CLI tools, research prototypes, small libraries, local data tools, lightweight APIs, internal helpers.

**Out of scope:** monorepos, enterprise platforms, large microservice systems, production infra, full MLOps/DevOps scaffolds.