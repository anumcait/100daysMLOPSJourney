# Day 78: Parallelise Tests via a Gitea Actions Matrix Strategy

## Objective

Convert the `test` job in the `fraud-detector` Gitea Actions workflow from running all test suites serially to using a matrix strategy.

The three test suites are:

- `tests/test_train.py`
- `tests/test_data_quality.py`
- `tests/test_model_contract.py`

The goal is to run each suite in its own parallel matrix job while leaving the `lint` job unchanged.

## Repository

- Repository: `fraud-detector`
- Branch: `add-test-matrix`
- Pull Request: `Convert test job to matrix strategy`

## Original Configuration

The original `test` job ran all tests serially:

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install pytest + runtime deps
      run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
    - name: Run all tests
      run: python3 -m pytest tests -v
```

## Solution

Added a Gitea Actions matrix strategy:

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      suite:
        - train
        - data_quality
        - model_contract

  steps:
    - uses: actions/checkout@v4
    - name: Install pytest + runtime deps
      run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
    - name: Run ${{ matrix.suite }} tests
      run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

## Matrix Mapping

| Matrix value | Test file |
|---|---|
| `train` | `tests/test_train.py` |
| `data_quality` | `tests/test_data_quality.py` |
| `model_contract` | `tests/test_model_contract.py` |

The expression:

```text
tests/test_${{ matrix.suite }}.py
```

automatically maps each matrix value to its corresponding test file.

## Final Workflow

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
        run: ruff check src tests

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        suite:
          - train
          - data_quality
          - model_contract

    steps:
      - uses: actions/checkout@v4
      - name: Install pytest + runtime deps
        run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
      - name: Run ${{ matrix.suite }} tests
        run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

## Git Commands

The changes were committed and pushed with:

```bash
git add .gitea/workflows/ci.yml
git commit -m "Convert test job to matrix strategy"
git push origin add-test-matrix
```

Commit created:

```text
ca36d40
```

## CI Verification

The latest Gitea Actions run created four jobs:

- ✅ `lint`
- ✅ `test (data_quality)`
- ✅ `test (model_contract)`
- ✅ `test (train)`

All jobs completed successfully.

The PR therefore has a successful CI status with three separate `test` matrix entries.

## Before vs After

### Before

```text
test
 ├── test_train.py
 ├── test_data_quality.py
 └── test_model_contract.py
```

All suites ran serially in one job.

### After

```text
test (train)
test (data_quality)
test (model_contract)
```

Each suite runs in its own matrix cell and can execute in parallel.

## Key Takeaway

A matrix strategy allows one CI job definition to run multiple parameterised executions without duplicating the job configuration.

For this task:

```yaml
strategy:
  matrix:
    suite:
      - train
      - data_quality
      - model_contract
```

creates three independent test jobs.

### Screenshots
