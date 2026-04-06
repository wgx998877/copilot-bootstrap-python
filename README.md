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
│   ├── __init__.py
│   └── main.py or core.py
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
| [package-main.py](templates/package-main.py) | `src/<pkg>/main.py` — entrypoint |
| [core.py](templates/core.py) | `src/<pkg>/core.py` — core logic |
| [test_smoke.py](templates/test_smoke.py) | `tests/test_smoke.py` — minimal test |

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

**Default: include none.** Most project guidance belongs in AGENTS.md, not in separate skills.

Include a skill only when ALL of these are true:
1. The workflow is **repeated** — it will be invoked more than once during the project's life.
2. The workflow is **procedural** — it has concrete steps, not just general advice.
3. The workflow is **distinct** from normal coding — it's a separate mode of work (reviewing, debugging, releasing), not "write good code."

**Decision guide:**

| If the brief mentions... | Consider |
|-------------------------|---------|
| "ongoing reviews" or "team project" | code-review |
| "demo", "showcase", "stakeholder presentation" | demo-build |
| "research", "experiment", "hypothesis" | experiment-review |
| "publish to PyPI", "release", "versioning" | release-prep |
| Nothing about these | No skills |

**Never include:** debug. It's in the catalog as a reference for the developer's own use, not as a default for generated projects. Debugging is too universal to be a project-specific skill.

**Why no agents?** For small projects, skills are sufficient. Custom agents (`.github/agents/`) add value when you need **tool restrictions** (e.g., read-only reviewer) or **context isolation** (subagent returns a single result). Small projects rarely need either. If a project outgrows skills, the user should create agents custom-tailored to their workflow — not copied from a catalog.

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