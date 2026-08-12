# Day 77: Fix a Failing Data-Quality Job in Gitea Actions
The xFusionCorp Industries ML platform team requires data-schema tests to be executed as a continuous integration (CI) gate for every pull request, ensuring that poor training data is detected before it affects the model. A team member has submitted a pull request titled Add data-quality CI gate in the fraud-detector repository; however, the newly added data-quality job has failed on its initial execution. Your objective is to examine the failed run log in Gitea Actions, determine the cause of the job failure, rectify the workflow, and push your changes so that the pull request is successful.


The Gitea UI is running on port 3000 (the Gitea button opens the login page). Admin credentials: gitea-admin / gitea2026. The repo lives at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector, already checked out on branch add-data-validation.

The pre-opened PR's workflow at .gitea/workflows/ci.yml declares three jobs: lint, test (both green), and data-quality (meant to run the data-schema tests, currently red). Open the failed data-quality run from the PR's Checks tab to read its log.

The end state must include:

The data-quality job is still declared in the workflow (do not delete the job itself).
The data-quality job's pytest step references a .py file that exists on the add-data-validation branch.
After the latest push, the PR's head commit's combined status reaches success (all three jobs green).
The point of a red CI run is not just the red pill in the PR—it is the log underneath it. A workflow can look fine by static inspection and still fail at runtime.

## Objective

Fix the failing `data-quality` CI job in the `fraud-detector` repository so that all three CI jobs pass and the pull request reaches a successful combined status.

## Problem

The pull request **Add data-quality CI gate** introduced a new `data-quality` job in `.gitea/workflows/ci.yml`.

The `lint` and `test` jobs were passing, but `data-quality` was failing.

## Root Cause

The workflow referenced a test file that does not exist on the `add-data-validation` branch:

    tests/test_data_validation.py

The correct test file is:

    tests/test_data_quality.py

The `data-quality` job therefore failed because pytest was given the wrong file path.

## Fix

Change this line in `.gitea/workflows/ci.yml`:

    run: python3 -m pytest tests/test_data_validation.py -v

to:

    run: python3 -m pytest tests/test_data_quality.py -v

The `data-quality` job must remain in the workflow.

## Correct Workflow

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
        steps:
          - uses: actions/checkout@v4
          - name: Install pytest
            run: pip install --break-system-packages pytest
          - name: Run pytest
            run: python3 -m pytest tests/test_train.py -v

      data-quality:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install pytest + pandas
            run: pip install --break-system-packages pytest pandas
          - name: Run data-quality tests
            run: python3 -m pytest tests/test_data_quality.py -v

## Commit and Push

    cd /root/code/fraud-detector
    git add .gitea/workflows/ci.yml
    git commit -m "fix: correct data-quality test path"
    git push origin add-data-validation

## Verification

After pushing, open the pull request and check the **Checks** tab.

Expected result:

- CI / lint — Success
- CI / test — Success
- CI / data-quality — Success
- Combined PR status — **Success**

## Key Lesson

A CI workflow can look structurally correct while still failing at runtime. When a CI job is red, inspect the actual run log first.

The failure was caused by a pytest path pointing to a nonexistent file.

The fix:

    - run: python3 -m pytest tests/test_data_validation.py -v
    + run: python3 -m pytest tests/test_data_quality.py -v

### Screenshots

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/0db9362f-143e-49a6-a1ec-4313c70013ba" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/50b9ee8d-b148-43ea-963a-2d47cbe86879" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5790dc8a-bf32-46cb-8540-c1fff0ce1541" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/cdb6cf7c-6beb-4e71-8163-d56c94cf05ae" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/438748c3-88d8-4d9c-94e1-c8305ee4289d" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/6ff5737f-c5f9-438e-9e30-e62a7ccc3dab" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/9e554977-fbcb-4334-bfc9-d38c980461ae" />







