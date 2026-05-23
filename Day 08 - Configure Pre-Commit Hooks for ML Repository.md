# Day 8: Configure Pre-Commit Hooks for ML Repository

## Task Description
The xFusionCorp Industries ML team enforces code quality on every commit via `pre-commit` hooks. The goal is to correct the draft `.pre-commit-config.yaml` file in the git repository at `/root/code/fraud-detection/` so that it successfully executes when running `pre-commit run --all-files`.

The corrected configuration must declare the following five hooks:
1. `trailing-whitespace` (sourced from `pre-commit/pre-commit-hooks`, pinned to a current release)
2. `end-of-file-fixer` (sourced from `pre-commit/pre-commit-hooks`, pinned to a current release)
3. `check-yaml` (sourced from `pre-commit/pre-commit-hooks`, pinned to a current release)
4. `ruff` (sourced from `astral-sh/ruff-pre-commit`, pinned to a current release)
5. `black` (sourced from `psf/black-pre-commit-mirror`, pinned to a current release)

Every repository entry in the configuration must include a `rev:` field. Once corrected, register the hooks with git and run them against the tracked files.

---

## Solution

To achieve this, we configure `.pre-commit-config.yaml` at the project root and install/run the hooks.

### 1. Configure `.pre-commit-config.yaml`
Create or update `/root/code/fraud-detection/.pre-commit-config.yaml` with the following configuration:

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.13
    hooks:
      - id: ruff

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 25.1.0
    hooks:
      - id: black
```

### 2. Install and Run the Pre-Commit Hooks
Navigate to the project directory, install the pre-commit Git hooks, and execute them against the tracked repository files:

```bash
cd /root/code/fraud-detection

# Install the hooks to register them with Git
pre-commit install

# Run the hooks manually against all tracked files in the repo
pre-commit run --all-files
```

---

## Verification

To verify that the pre-commit pipeline works cleanly:

### 1. Execute the Run Command
Run the hooks manually to make sure all five pass successfully:

```bash
pre-commit run --all-files
```

Expected output:
```text
trim trailing whitespace.................................................Passed
fix end of files.........................................................Passed
check yaml...............................................................Passed
ruff.....................................................................Passed
black....................................................................Passed
```

### 2. Commit Hook Automatic Trigger
Any subsequent `git commit` command will trigger these checks automatically before allowing the commit to be finalized. If any checks fail, `pre-commit` will format or lint the code and abort the commit, allowing you to review changes, stage them, and attempt the commit again.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e079f187-dad2-4473-91eb-13a109ba3f88" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8e0e8cd5-1cfe-4813-8b2a-37f9431e43b8" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c9e01dbe-9836-44ce-b2ed-1bcda8ef7399" />
<img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/e954cbe5-bee7-47b9-8931-8f43600c9602" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/cc83202d-bf10-4352-b90c-820a79351e6c" />







