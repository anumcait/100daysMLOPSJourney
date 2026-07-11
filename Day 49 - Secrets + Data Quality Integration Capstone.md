# Day 49 - Secrets + Data Quality Integration Capstone
The xFusionCorp Industries ML platform team is preparing to cut their first end-to-end release of the fraud-detector repository. This release comprises a three-job Gitea Actions workflow that includes: pulling the MLflow credential from Vault, verifying data quality by gating on a Great Expectations checkpoint, and registering the trained model in MLflow. All four services—Vault, MLflow, Gitea, and the Actions runner—are operational. However, the first job of the workflow remains incomplete, as the step for reading from Vault is noted as a TODO. Your final task is to complete this section by scripting the step to pull the MLflow credential from Vault, and subsequently, execute the release across its various user interfaces: staging the credential in Vault, initiating and merging a pull request in Gitea, and promoting the registered model in MLflow.


Each of the four UIs has a button at the top of the lab:

Gitea (port 3000) – gitea-admin / gitea2026. The fraud-detector repo sits on main; a feature branch production-release is pre-pushed. No pull request has been opened yet.
Vault (port 8200) – log in with the token at /root/code/vault-token. The KV v2 engine is enabled at secret/; secret/mlflow is empty.
MLflow UI (port 5000) – the Models page is empty.
Data Docs – rendered by the data-quality job once the workflow runs.
The workflow at .gitea/workflows/production.yml on the production-release branch has three jobs (fetch-secret → data-quality → register-model). The data-quality and register-model jobs are complete; the fetch-secret job's Vault-read step is left as a # TODO in the working clone at /root/code/fraud-detector. The workflow reads a Vault KV key, runs the schema_check GE checkpoint, and registers the trained model as fraud-detector in MLflow; it only triggers on pull_request against main.

The end state must include:

secret/mlflow has a non-empty mlflow_password key (any value works).
The fetch-secret job on the production-release branch reads the KV v2 path secret/data/mlflow and its mlflow_password key (the authored step).
A pull request exists from production-release → main and has been merged.
The workflow run on that PR's head commit reaches combined status success (all three jobs green).
fraud-detector is registered in MLflow with the production alias pointing at one of its versions.
Each of the four pieces lives behind a different UI and, in a real team, a different owner: Vault (security), Gitea (the dev lead opening + merging the PR), MLflow (the ML engineer promoting the model), Data Docs (the data team reviewing the quality report). The capstone walks all four. Order matters for the first step: stage the Vault secret before opening the PR, otherwise the workflow's very first job fails and the reader has to re-trigger.

## Objective

Complete the missing Vault integration in the production release workflow and execute a full end-to-end MLOps pipeline using Vault, Gitea Actions, Great Expectations, and MLflow.

---

## Task

The `production.yml` workflow contained three jobs:

- `fetch-secret`
- `data-quality`
- `register-model`

The `fetch-secret` job was incomplete. The objective was to:

- Store the MLflow credential in Vault.
- Implement the missing Vault read step.
- Push the changes to the `production-release` branch.
- Create and merge a Pull Request.
- Verify the workflow execution.
- Confirm model registration and production promotion in MLflow.

---

## Architecture

```text
                Pull Request
                      │
                      ▼
             Gitea Actions Workflow
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 fetch-secret    data-quality    register-model
      │               │                │
      ▼               ▼                ▼
 HashiCorp Vault   Great Expectations   MLflow Registry
      │               │                │
      └───────────────┴────────────────┘
                      │
                      ▼
              Production Release
```

---

## Implementation

### 1. Store the MLflow Secret in Vault

Configured the Vault environment and stored the required credential.

```bash
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=$(cat /root/code/vault-token)

vault kv put secret/mlflow mlflow_password="my-secret-password"
```

---

### 2. Update the Workflow

Edited the workflow file:

```text
/root/code/fraud-detector/.gitea/workflows/production.yml
```

Replaced the TODO section with a script that:

- Reads the Vault token.
- Calls the Vault REST API.
- Retrieves the `mlflow_password`.
- Fails the workflow if the secret is missing.
- Prints only the password length.

---

### 3. Commit and Push the Changes

```bash
git checkout production-release

git add .gitea/workflows/production.yml

git commit -m "Implement Vault secret retrieval"

git push origin production-release
```

---

### 4. Create the Pull Request

Created a Pull Request:

```text
production-release → main
```

This automatically triggered the Gitea Actions workflow.

---

### 5. Verify the Workflow

The workflow executed successfully.

| Job | Status |
|------|--------|
| fetch-secret | ✅ Success |
| data-quality | ✅ Success |
| register-model | ✅ Success |

---

### 6. Merge the Pull Request

After all checks passed, the Pull Request was merged into the `main` branch.

---

### 7. Verify MLflow

Opened the MLflow UI and confirmed:

| Item | Value |
|------|-------|
| Registered Model | fraud-detector |
| Version | 1 |
| Alias | @production |

---

## Result

Successfully completed the end-to-end MLOps production release by:

- Storing credentials securely in Vault.
- Reading secrets during workflow execution.
- Validating data quality using Great Expectations.
- Registering the trained model in MLflow.
- Promoting the model using the `production` alias.
- Completing the release through a Pull Request workflow.

---

## Key Commands

```bash
export VAULT_ADDR=http://localhost:8200
export VAULT_TOKEN=$(cat /root/code/vault-token)

vault kv put secret/mlflow mlflow_password="my-secret-password"

git checkout production-release
git add .gitea/workflows/production.yml
git commit -m "Implement Vault secret retrieval"
git push origin production-release
```

---

## Summary

This capstone integrated HashiCorp Vault, Gitea Actions, Great Expectations, and MLflow into a single automated release pipeline. The workflow securely retrieved credentials, validated dataset quality, registered the machine learning model, and promoted it to production through a Pull Request-based CI/CD process.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/f994126c-5a5f-426d-ae4b-d3620baf4cb9" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3fb79e06-b5c4-4fc2-85cc-6793a56a8f19" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/37c88398-c782-4bdd-b899-03f037d68727" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/967f8671-578c-41b2-8c48-81e7a770670a" />
<img width="50" height="300" alt="image" src="https://github.com/user-attachments/assets/f91e02f7-5441-4e07-9286-b6e2e7690fdc" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/be7ce556-a97c-42f2-bca7-6d37747e4ce4" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c1bc17c4-0a15-469a-b12d-7002ea274695" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/09b75b0a-8bc0-492a-ac74-85cee23daf55" />









