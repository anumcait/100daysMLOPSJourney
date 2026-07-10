# Day 49 - Secrets + Data Quality Integration Capstone

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
