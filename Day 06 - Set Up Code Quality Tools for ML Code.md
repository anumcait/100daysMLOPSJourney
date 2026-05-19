# Day 6: Set Up Code Quality Tools for ML Code

## Task Description
The xFusionCorp Industries ML team enforces code quality standards by running `ruff` and `black` on every pull request. The project at `/root/code/fraud-detection/` contains a draft `pyproject.toml` and sample sources under `src/`, but currently fails checks for both tools.

The goal is to fix the configurations and source code to ensure that:
1. Both `ruff` and `black` are configured with a maximum line length of `120` in `pyproject.toml`.
2. `ruff` is configured with lint rule selections including `E` (pycodestyle errors), `F` (Pyflakes), `W` (pycodestyle warnings), and `I` (isort for imports) under the `[tool.ruff.lint]` section (schema format for `ruff` versions 0.1+).
3. Both tools exit cleanly (status code 0) when checks are run against the source code (`src/`).

---

## Solution

To achieve this, we need to modify the configuration file and run code auto-formatting and lint fixing.

### 1. Configure `pyproject.toml`
Create or update `/root/code/fraud-detection/pyproject.toml` with the standard line length and linting requirements:

```toml
[tool.black]
line-length = 120

[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "W", "I"]
```

### 2. Format and Lint the Source Files
Navigate to the project directory and run formatting/linting corrections:

```bash
cd /root/code/fraud-detection

# Format imports and fix common linting errors automatically using Ruff
ruff check src/ --fix

# Auto-format the code structure using Black
black src/
```

Running these commands will automatically sort imports (with `isort`/`I`), fix easily resolvable style issues (`E`, `F`, `W`), and restructure the code layout using `black` to adhere to the 120-character line limit.

---

## Verification

Validate that both formatting and lint checks exit successfully without errors:

```bash
# Check for linting issues (exits with status 0 if clean)
ruff check src/

# Verify that formatting is completely compliant (exits with status 0 if clean)
black --check src/
```

When both commands return no errors, the repository is fully compliant with the team's automated code quality checks and ready for PR integration!

### Screenshots
