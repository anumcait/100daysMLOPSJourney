# Day 82: Compose Gitea Workflows via `workflow_call`
The xFusionCorp Industries ML platform team is currently enhancing the fraud-detector continuous integration (CI) process. The existing main.yml file contains three similar inline jobs that have been duplicated, requiring the same changes to be implemented for each on every pull request (PR). A team member has already separated the lint, test, and report stages into distinct files located in the .gitea/workflows/ directory—each configured to declare on: workflow_call—and has opened a PR that modifies one job (lint) to call the reusable workflow. Your task is to complete this refactor by converting the two remaining inline jobs in main.yml into uses: calls, thereby enabling the main run to expand into three nested workflow_call executions.

The Gitea UI is on port 3000 (Gitea button). Admin credentials: gitea-admin / gitea2026. The repo is at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector, already checked out on branch add-reusable-workflows. The PR is pre-opened.

Four workflow files ship in .gitea/workflows/. lint.yml, test.yml, and report.yml are reusable callees, each triggered by on: workflow_call. main.yml is the caller: jobs.lint already calls ./.gitea/workflows/lint.yml via uses: (the example wiring), while jobs.test and jobs.report are still inline runs-on + steps jobs that duplicate logic already parked in the callee files. Those two inline job bodies need to become single uses: lines mirroring the lint job — a job cannot declare both uses: and steps:, so the inline blocks are removed entirely.

The end state must include:

lint.yml, test.yml, and report.yml each declare on: workflow_call on the PR head branch.
main.yml defines jobs lint, test, and report, each with a uses: key pointing at the matching ./.gitea/workflows/<name>.yml.
No job in main.yml mixes uses: with steps: (illegal in Actions YAML).
The PR head commit's combined status reaches success.

Reusable workflows turn a monolithic main.yml into a small graph of composable pieces. Each callee becomes the canonical definition for its concern (lint / test / report); any number of callers can consume it. When you later add a release.yml workflow for tagged pushes, it can reuse the same test.yml callee—no more copy-paste sync issues across files.

## Objective

Refactor the `fraud-detector` Gitea CI workflow so that the main workflow uses reusable workflows for the `lint`, `test`, and `report` stages.

The existing `main.yml` had three jobs. The `lint` job already called a reusable workflow, while `test` and `report` contained duplicated inline logic.

The goal was to convert `test` and `report` into reusable workflow calls.

---

## Repository

Repository:

`gitea-admin/fraud-detector`

Working directory:

`/root/code/fraud-detector`

Branch:

`add-reusable-workflows`

Reusable workflows:

* `.gitea/workflows/lint.yml`
* `.gitea/workflows/test.yml`
* `.gitea/workflows/report.yml`

Caller workflow:

* `.gitea/workflows/main.yml`

---

## Before

The `lint` job was already using a reusable workflow:

```yaml
jobs:
  lint:
    uses: ./.gitea/workflows/lint.yml
```

However, `test` and `report` were still inline jobs:

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install pytest + runtime deps
      run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
    - name: Run pytest
      run: python3 -m pytest tests -v
```

```yaml
report:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install report deps
      run: pip install --break-system-packages numpy scikit-learn joblib matplotlib
    - name: Train
      run: python3 -m src.train
    - name: Plot confusion matrix
      run: python3 -m src.plot
    - name: Upload report
      uses: actions/upload-artifact@v3
      with:
        name: model-report
        path: artifacts/
```

---

## Solution

The two inline jobs were replaced with `uses:` calls:

```yaml
name: Main

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    uses: ./.gitea/workflows/lint.yml

  test:
    uses: ./.gitea/workflows/test.yml

  report:
    uses: ./.gitea/workflows/report.yml
```

Each job now delegates its implementation to the corresponding reusable workflow.

---

## Reusable Workflow Structure

Each reusable workflow declares:

```yaml
on:
  workflow_call:
```

This allows another workflow to call it using `uses:`.

The relationship is:

```text
main.yml
   |
   +-- lint  ---> .gitea/workflows/lint.yml
   |
   +-- test  ---> .gitea/workflows/test.yml
   |
   +-- report ---> .gitea/workflows/report.yml
```

This creates a small graph of composable workflows instead of keeping all CI logic in one large file.

---

## Important Rule

A reusable workflow job cannot mix `uses:` with `steps:`.

Incorrect:

```yaml
jobs:
  test:
    uses: ./.gitea/workflows/test.yml
    steps:
      - run: echo "This is invalid"
```

Correct:

```yaml
jobs:
  test:
    uses: ./.gitea/workflows/test.yml
```

Therefore, the old `runs-on:` and `steps:` sections had to be removed completely from `test` and `report`.

---

## Git Commands Used

Navigate to the repository:

```bash
cd ~/code/fraud-detector
```

Check the changes:

```bash
git diff -- .gitea/workflows/main.yml
```

Check repository status:

```bash
git status
```

Stage the workflow:

```bash
git add .gitea/workflows/main.yml
```

Commit the refactor:

```bash
git commit -m "refactor test and report to reusable workflows"
```

Push the branch:

```bash
git push origin add-reusable-workflows
```

---

## Verification

Verify that all reusable workflows contain `workflow_call`:

```bash
grep -n "workflow_call" .gitea/workflows/*.yml
```

Verify that `main.yml` contains the three reusable workflow calls:

```bash
grep -n "uses:" .gitea/workflows/main.yml
```

Expected:

```text
uses: ./.gitea/workflows/lint.yml
uses: ./.gitea/workflows/test.yml
uses: ./.gitea/workflows/report.yml
```

---

## Gitea Actions Result

The pushed commit was:

`55ee7e2df5`

The Gitea Actions run expanded into three jobs:

* `lint` — Success
* `test` — Completed successfully
* `report` — Completed successfully
* `model-report` artifact — Generated

The successful workflow demonstrates that the three reusable workflows were invoked correctly.

---

## Key Learnings

### 1. Reusable workflows

A workflow can be designed as a reusable component using:

```yaml
on:
  workflow_call:
```

Another workflow can then invoke it with:

```yaml
uses: ./.gitea/workflows/workflow-name.yml
```

### 2. Avoid duplicated CI logic

Instead of maintaining identical CI steps in multiple workflows, keep the implementation in one reusable workflow.

For example:

```text
test.yml
    ^
    |
main.yml
    ^
    |
future release.yml
```

Both workflows can reuse the same testing implementation.

### 3. `uses:` replaces the job implementation

When a job calls a reusable workflow:

```yaml
test:
  uses: ./.gitea/workflows/test.yml
```

the caller does not define:

```yaml
runs-on:
steps:
```

The reusable workflow owns those details.

### 4. Composable CI

Reusable workflows make it easier to build larger CI/CD pipelines from smaller components.

For example:

```text
Pull Request
     |
     +-- lint
     |
     +-- test
     |
     +-- report
```

A future release workflow could reuse the same `test.yml` without copying the test steps.

---

## Final State

The final `main.yml` contains exactly three jobs:

```yaml
jobs:
  lint:
    uses: ./.gitea/workflows/lint.yml

  test:
    uses: ./.gitea/workflows/test.yml

  report:
    uses: ./.gitea/workflows/report.yml
```

All three stages are now independently reusable, eliminating duplicated workflow logic and making future CI/CD changes easier to maintain.

**Result:** Successfully refactored `main.yml` to use reusable `lint`, `test`, and `report` workflows and verified the Gitea Actions run successfully.

### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/6b826890-d6af-42cb-a756-ef993559255b" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/d29f2ab4-b071-4be1-aaa4-f3ae16dafa7e" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/57534ebd-0fd1-4786-a9a8-ed61f021b9a2" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/dd18eb50-12b2-4e63-82bc-5edd369dd361" />





