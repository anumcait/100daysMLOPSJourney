# Day 76 — Create CI Pipeline for ML Code Linting and Testing
The xFusionCorp Industries ML platform team requires that pull requests for the fraud-detector repository automatically run linting and testing processes prior to the review stage. A local Gitea server is currently operational with the fraud-detector repository pre-pushed on the main branch. Additionally, a self-hosted Actions runner has been registered and is awaiting jobs. Your objective is to complete the integration of the CI workflow template, which is located at .gitea/workflows/ci.yml.template on the main branch, within a feature branch. Once completed, open a pull request, monitor the Checks tab for successful approvals, and subsequently merge the pull request into the main branch.


The Gitea UI is running on port 3000 (the Gitea button opens the login page). Admin credentials: gitea-admin / gitea2026. The repo is at http://localhost:3000/gitea-admin/fraud-detector, with a working clone at /root/code/fraud-detector (on main).

The scaffold under /root/code/fraud-detector/ lints clean and has passing tests: src/train.py (a deterministic synthetic training script), tests/test_train.py (three passing unit tests), pyproject.toml (Ruff + pytest config), and .gitea/workflows/ci.yml.template — a pre-written CI workflow with the on: triggers and lint + test job skeletons already wired, plus # TODO: markers on the two run: lines that need filling in. Ruff and pytest are both installed on the host. Gitea Actions only schedules *.yml / *.yaml files, so the .template suffix keeps the file inert until it is renamed to ci.yml.

The end state must include:

A workflow file .gitea/workflows/ci.yml on branch add-ci that parses as YAML and declares both a lint and a test job.
One pull request in the repo targeting main, with add-ci as its head branch.
That PR's head commit reports combined status success on GET /api/v1/repos/gitea-admin/fraud-detector/commits/{sha}/status.
The PR is merged into main (merged: true on the pulls API).
Gitea Actions uses the same YAML syntax as GitHub Actions, so the workflow you ship here is also the kind of file you would drop into any github.com repo. Real CI engineers rarely author these from scratch — they inherit a template and edit the two or three lines that are project-specific, which is exactly what the .template scaffold mirrors.

## Objective

Configure a Gitea Actions CI pipeline for the `fraud-detector` repository so that pull requests targeting `main` automatically run linting and testing.

The workflow is created from the provided `.gitea/workflows/ci.yml.template`, committed on a feature branch, validated through a pull request, and merged into `main`.

## Environment

- Repository: `gitea-admin/fraud-detector`
- Local clone: `/root/code/fraud-detector`
- Gitea: `http://localhost:3000`
- Feature branch: `add-ci`
- Base branch: `main`
- Gitea version: `1.22.3`
- Self-hosted Actions runner: already registered

## Existing Project Structure

```text
fraud-detector/
├── src/
│   └── train.py
├── tests/
│   └── test_train.py
├── pyproject.toml
└── .gitea/
    └── workflows/
        └── ci.yml.template
```

## Step 1 — Enter the Repository

```bash
cd /root/code/fraud-detector
git status
```

## Step 2 — Create the Feature Branch

```bash
git checkout -b add-ci
```

If the branch already exists:

```bash
git checkout add-ci
```

## Step 3 — Rename the Workflow Template

Gitea Actions only schedules `.yml` and `.yaml` workflow files.

```bash
mv .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml
```

## Step 4 — Complete the Workflow

Edit:

```bash
nano .gitea/workflows/ci.yml
```

Use:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install ruff
        run: pip install --break-system-packages ruff
      - name: Run ruff
        run: ruff check .

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install pytest
        run: pip install --break-system-packages pytest
      - name: Run pytest
        run: pytest
```

The two TODO commands are:

```yaml
run: ruff check .
```

and:

```yaml
run: pytest
```

## Step 5 — Run Tests Locally

```bash
ruff check .
pytest
```

Expected pytest result:

```text
3 passed
```

## Step 6 — Stage the Workflow

```bash
git add .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml
git status
```

Git should recognize the change as a rename:

```text
renamed: .gitea/workflows/ci.yml.template -> .gitea/workflows/ci.yml
```

## Step 7 — Commit

```bash
git commit -m "Add CI pipeline for linting and tests"
```

## Step 8 — Push the Feature Branch

```bash
git push -u origin add-ci
```

Verify:

```bash
git status
```

Expected:

```text
On branch add-ci
Your branch is up to date with 'origin/add-ci'.

nothing to commit, working tree clean
```

## Step 9 — Create the Pull Request

Open the repository in Gitea and create a pull request:

```text
Base: main
Head: add-ci
```

## Step 10 — Monitor CI Checks

Open the PR's **Checks** section.

The workflow should contain:

```text
CI / lint
CI / test
```

Wait until both are successful:

```text
CI / lint   Successful
CI / test   Successful
```

Do not merge before both checks pass.

## Step 11 — Merge the Pull Request

After both checks succeed:

1. Click **Merge Pull Request**
2. Click **Confirm Merge**

The PR should show:

```text
Merged
```

and:

```text
Pull request successfully merged and closed
```

## Step 12 — Verify Main

```bash
git checkout main
git pull
git log --oneline -3
```

The merge commit should now be present on `main`.

## Final Checklist

- [x] `.gitea/workflows/ci.yml` exists
- [x] Workflow is on `add-ci`
- [x] `lint` job exists
- [x] `test` job exists
- [x] Ruff uses `ruff check .`
- [x] Tests use `pytest`
- [x] Ruff passes
- [x] Pytest passes
- [x] PR targets `main`
- [x] PR head is `add-ci`
- [x] Lint check succeeds
- [x] Test check succeeds
- [x] PR is merged into `main`

## Key Lesson

Gitea Actions uses the same YAML workflow syntax as GitHub Actions.

In real CI work, engineers commonly start with an existing workflow template and modify the project-specific commands instead of writing the workflow from scratch.

For this task:

1. Rename the `.template` workflow.
2. Add the Ruff command.
3. Add the pytest command.
4. Commit and push `add-ci`.
5. Create the PR.
6. Wait for successful CI checks.
7. Merge the PR into `main`.

The final merge is essential because the task requires the pull request to have:

```text
merged: true
```

### Screenshots

