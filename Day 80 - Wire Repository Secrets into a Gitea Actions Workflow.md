# Day 80: Wire Repository Secrets into a Gitea Actions Workflow
The xFusionCorp Industries ML platform team requires that every pull request (PR) in the fraud-detector repository logs a training run to MLflow and registers the resulting model version. However, they prefer not to include the tracking URL or an API token in plaintext within the workflow file. Gitea and a local MLflow 3.x server are currently operational, and a teammate has initiated a PR titled Register trained model on every push. Currently, the first run fails because the register job is attempting to access MLFLOW_TRACKING_URI and MLFLOW_TOKEN from the environment, but these variables are not populated. Your task is to provision these two values as repository secrets and integrate them into the workflow to ensure the run succeeds.


The Gitea UI is on port 3000 (Gitea button); admin credentials gitea-admin / gitea2026. The MLflow UI is on port 5000 (MLflow UI button). The repo is at http://localhost:3000/gitea-admin/fraud-detector and a working clone is at /root/code/fraud-detector, already checked out on branch add-registry-push. The PR is pre-opened.

The shipped .gitea/workflows/ci.yml declares a register job that runs python3 -m src.register. src/register.py reads MLFLOW_TRACKING_URI and MLFLOW_TOKEN from os.environ and exits non-zero if either is missing, so on the first run the job fails with that error. Two pieces are needed: the repository secrets MLFLOW_TRACKING_URI (value http://localhost:5000) and MLFLOW_TOKEN (any non-empty string, e.g. fraud-detector-ci-token — the lab's MLflow does not enforce auth, but the script refuses to run without the value so a missing secret surfaces as a clear failure), created under the repo's Settings → Actions → Secrets; and the workflow wiring that exports each secret into the register job's environment under the same name.

The end state must include:

GET /api/v1/repos/gitea-admin/fraud-detector/actions/secrets lists both MLFLOW_TRACKING_URI and MLFLOW_TOKEN.
The register job in the workflow references each secret via ${{ secrets.<NAME> }} inside an env: block (job-level or step-level).
The PR head commit's combined status is success.
MLflow's registered-model endpoint reports fraud-detector with at least one version.
Repository secrets are the CI version of an environment-specific config file. The YAML stays identical between dev, staging, and prod; only the secret values change. This is also the pattern you extend when you add a PyPI token, an S3 access key, or a Kubernetes kubeconfig to a workflow—never paste the value into the committed file.

## Objective

Configure repository secrets in Gitea and wire them into the `register` job of the `fraud-detector` CI workflow so that every PR can train and register a model with MLflow without storing sensitive configuration directly in the workflow.

## Environment

* Repository: `gitea-admin/fraud-detector`
* Branch: `add-registry-push`
* Gitea: `http://localhost:3000`
* MLflow: `http://localhost:5000`
* Working clone: `/root/code/fraud-detector`
* Workflow: `.gitea/workflows/ci.yml`
* PR: `Register trained model on every push`

## Problem

The `register` job runs:

```bash
python3 -m src.register
```

The registration script requires these environment variables:

```text
MLFLOW_TRACKING_URI
MLFLOW_TOKEN
```

Initially, these variables were not populated, causing the `register` job to fail.

The solution is to:

1. Create both values as Gitea repository secrets.
2. Reference the secrets from the workflow using `${{ secrets.<NAME> }}`.
3. Run the workflow again and verify the model registration.

## Step 1: Configure the Workflow

The `register` job was updated to expose the repository secrets through an `env` block:

```yaml
register:
  runs-on: ubuntu-latest
  env:
    MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
    MLFLOW_TOKEN: ${{ secrets.MLFLOW_TOKEN }}
  steps:
    - uses: actions/checkout@v4
    - name: Install registration deps
      run: pip install --break-system-packages mlflow numpy scikit-learn joblib pandas
    - name: Train and register
      run: python3 -m src.register
```

The actual secret values are **not** stored in the YAML file.

## Step 2: Commit the Workflow Change

Switch to the PR branch:

```bash
cd /root/code/fraud-detector
git checkout add-registry-push
```

Check the changes:

```bash
git status
git diff -- .gitea/workflows/ci.yml
```

Stage the workflow:

```bash
git add .gitea/workflows/ci.yml
```

Commit:

```bash
git commit -m "Wire MLflow secrets into register job"
```

Push the PR branch:

```bash
git push origin add-registry-push
```

The resulting commit was:

```text
0a66a366caed1b3ef17f95ac0f421cf455c21133
```

## Step 3: Create the Repository Secrets

In Gitea:

```text
fraud-detector
  -> Settings
  -> Actions
  -> Secrets
```

Create:

### MLFLOW_TRACKING_URI

```text
Name:
MLFLOW_TRACKING_URI

Value:
http://localhost:5000
```

### MLFLOW_TOKEN

```text
Name:
MLFLOW_TOKEN

Value:
fraud-detector-ci-token
```

The token does not need to be a real authentication token in this lab. The registration script only requires it to be non-empty.

## Step 4: Verify the Secrets Through the Gitea API

Run:

```bash
curl -u gitea-admin:gitea2026 \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/actions/secrets
```

Expected result:

```json
[
  {
    "name": "MLFLOW_TRACKING_URI",
    "created_at": "..."
  },
  {
    "name": "MLFLOW_TOKEN",
    "created_at": "..."
  }
]
```

The lab returned both secrets:

```text
MLFLOW_TRACKING_URI
MLFLOW_TOKEN
```

## Step 5: Verify the Gitea Actions Run

The workflow was triggered by commit:

```text
0a66a366caed1b3ef17f95ac0f421cf455c21133
```

Actions run:

```text
#4
```

The workflow contains these jobs:

```text
lint
test
register
```

The `lint` job completed successfully.

The important job is:

```text
register
```

It must finish with:

```text
Success
```

## Step 6: Verify the PR Status

Open the PR:

```text
Register trained model on every push
```

The PR head commit must have a combined status of:

```text
success
```

The workflow must pass all required jobs.

## Step 7: Verify MLflow Registration

Open the MLflow UI and check the registered models.

The required registered model is:

```text
fraud-detector
```

It must have at least one model version.

The final MLflow state should therefore contain:

```text
fraud-detector
  Version 1
  ...
```

## Useful Verification Commands

### Check repository secrets

```bash
curl -u gitea-admin:gitea2026 \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/actions/secrets
```

### Check the workflow jobs

```bash
curl -u gitea-admin:gitea2026 \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/actions/runs/4/jobs
```

### Check the commit status

```bash
curl -u gitea-admin:gitea2026 \
  "http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/commits/0a66a366caed1b3ef17f95ac0f421cf455c21133/status"
```

A successful result should report:

```text
success
```

### Check the MLflow registered model

```bash
curl "http://localhost:5000/api/2.0/mlflow/registered-models/get?name=fraud-detector"
```

## Final Checklist

* [x] Repository secret `MLFLOW_TRACKING_URI` created.
* [x] Repository secret `MLFLOW_TOKEN` created.
* [x] `register` job references `${{ secrets.MLFLOW_TRACKING_URI }}`.
* [x] `register` job references `${{ secrets.MLFLOW_TOKEN }}`.
* [x] Workflow change committed to `add-registry-push`.
* [x] Workflow change pushed to Gitea.
* [ ] `register` job confirmed successful.
* [ ] PR combined status confirmed as `success`.
* [ ] MLflow `fraud-detector` confirmed with at least one version.

## Key Takeaway

Repository secrets allow the workflow YAML to remain identical across environments while keeping environment-specific configuration outside the source code.

Instead of committing:

```yaml
MLFLOW_TRACKING_URI: http://localhost:5000
MLFLOW_TOKEN: fraud-detector-ci-token
```

use:

```yaml
env:
  MLFLOW_TRACKING_URI: ${{ secrets.MLFLOW_TRACKING_URI }}
  MLFLOW_TOKEN: ${{ secrets.MLFLOW_TOKEN }}
```

This same pattern can be used for other sensitive CI configuration such as:

* PyPI tokens
* AWS/S3 credentials
* Kubernetes kubeconfig credentials
* Cloud provider credentials
* Deployment API tokens

Never commit the actual secret value to the repository.
