# Repository Guidelines

This document explains how to contribute to **stock_test** – a lightweight stock‑analysis toolkit. It covers project layout, development workflow, coding style, testing, and pull‑request etiquette.

## Project Structure & Module Organization

- `src/` – Production code (Python modules).
- `tests/` – Unit tests mirroring the source tree.
- `scripts/` – Utility scripts for data import/export.
- `assets/` – Static resources such as CSV fixtures.
  
The top‑level `__init__.py` files expose public APIs; keep them minimal.

## Build, Test, and Development Commands

| Command | What it does |
|---------|--------------|
| `make install` | Installs the package into a venv (`pip install -e .`). |
| `make lint` | Runs `ruff check src/ tests/` to enforce style. |
| `make test` | Executes all unit tests with coverage (`pytest --cov=src`). |
| `make run` | Starts the CLI demo script (`python scripts/demo.py`). |

All commands are available via the Makefile; running `make help` shows options.

## Coding Style & Naming Conventions

* **Python** – 4‑space indentation, line length ≤ 88 characters. Use type hints where appropriate.
* **Functions / methods** – snake_case names.
* **Classes** – PascalCase.
* **Constants** – UPPER_SNAKE_CASE.
* The project uses `ruff` for linting and auto‑formatting; run `make lint` before committing.

## Testing Guidelines

The test suite is written with **pytest**. Test files live under `tests/` and are named `test_*.py`. Each test module mirrors the source package structure.

*Run tests*: `make test`
*Coverage threshold*: 80 % (enforced by `tox`).

Add new tests when you introduce a public API change or fix a bug. Ensure they run locally before submitting.

## Commit & Pull Request Guidelines

### Commit Messages

Follow the conventional‑commits style seen in the repo history:

```
feat: add new data loader
fix: correct ticker normalization logic
docs: update README examples
``` 

Keep subject lines short (< 50 chars) and capitalize the first word.

### Pull Requests

* Provide a concise title that reflects the change.
* Include a description with:
    - Motivation / problem solved.
    - Key changes (diff snippets if relevant).
    - Tests added or updated.
* Link any related issue (`Fixes #123`).
* Add screenshots only for UI changes – not applicable here.
* Ensure all CI checks pass before merging.

All PRs should target the `develop` branch; merge to `main` via protected‑branch rules.

---

Feel free to reach out on the project’s Slack channel for questions or guidance.

