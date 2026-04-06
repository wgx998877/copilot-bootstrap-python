# {{PROJECT_NAME}}

{{ONE_SENTENCE_DESCRIPTION}}

<!-- Rewrite the description based on the user's brief. Do not leave template language. -->

## Getting Started

### Prerequisites

- Python >= {{min_python_version}}
- [uv](https://docs.astral.sh/uv/) (recommended) or pip

### Installation

```bash
# Using uv
uv sync

# Or using pip
pip install -e .
```

### Usage

```bash
# TODO: add primary usage example after initial implementation
```

## Project Structure

```
src/
└── {{package_name}}/
    ├── __init__.py
    └── main.py        # or core.py
tests/
└── test_smoke.py
```

## Development

```bash
# Run tests
uv run pytest

# Run the project
uv run python -m {{package_name}}
```

## Open Questions

- TODO: {{OPEN_QUESTION_1}}
- TODO: {{OPEN_QUESTION_2}}

<!-- Populate from genuine unknowns in the user's brief. Remove this comment. -->