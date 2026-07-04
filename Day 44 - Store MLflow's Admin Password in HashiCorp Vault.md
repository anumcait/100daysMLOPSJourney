# Day 44 - Store MLflow's Admin Password in HashiCorp Vault

The xFusionCorp Industries ML platform team wants every credential that a lab-ops service needs—MLflow's admin password today, SeaweedFS's access keys and PostgreSQL passwords later—to come out of HashiCorp Vault at service-start time rather than be hardcoded into a startup script. A dev Vault is already running on port 8200, its web UI is reachable via the Vault button, and an MLflow boot wrapper on the host is polling Vault every 5 s for `secret/mlflow.admin_password`—but the wrapper can only launch MLflow once that KV entry exists. Your task is to enable the KV v2 engine in Vault, create the secret, and watch MLflow come up on port 5000.

The Vault UI is on port 8200 (Vault button opens the login page). The dev-mode root token is pre-created and written to `/root/code/vault-token`; paste the file's contents into the Vault Token login field. (Production deployments would use userpass / AppRole / OIDC instead, but the root token is the shortest path for a dev server.)

The MLflow wrapper picks up the new KV entry within ~5 s and execs mlflow server on port 5000. The MLflow UI button then opens the live tracker.

The end state must include:
- A KV v2 secrets engine is enabled at path `secret/` — GET `/v1/sys/mounts` returns `secret/` with type: `kv` and `options.version`: `"2"`.
- The secret at path `secret/mlflow` carries a non-empty `admin_password` key — GET `/v1/secret/data/mlflow` (with the root token) returns a JSON body whose `data.data.admin_password` is a non-empty string.
- GET `http://localhost:5000/` answers 200 – MLflow is running because the wrapper found the password.

Running services should not know their own secrets at image-build time. A Vault-first pattern lets you rotate a credential in Vault and restart the consumer to pick up the new value—no rebuild, no config patch, no secret in the commit history. This lab's single-service wrapper is the minimum viable version of that pattern; a real deployment replaces the root token with an AppRole login and adds audit logging.

## Objective
Enable a Key-Value (KV v2) secrets engine in HashiCorp Vault, register MLflow's administrator password secret under `/secret/mlflow`, and verify that the monitoring boot wrapper successfully extracts the credential to launch the MLflow tracking server.

## Task
- **Authenticate with Vault**: Access HashiCorp Vault via the CLI or HTTP API using the pre-configured root token at `/root/code/vault-token`.
- **Mount KV v2 Secrets Engine**: Enable the version 2 Key-Value secrets storage engine at the mount path `secret/`.
- **Register MLflow Admin Password**: Write a non-empty password entry for key `admin_password` inside the secret `secret/mlflow`.
- **Deploy and Validate Server**: Wait for the background polling wrapper (polling every 5 seconds) to detect the secret and bootstrap the MLflow server on port `5000`.

## Solution

### Step 1: Export Vault Credentials
Set the terminal environment variables so the local Vault client knows how to locate and authenticate with the server:
```bash
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=$(cat /root/code/vault-token)
```

### Step 2: Enable the Key-Value Secrets Engine
By default, the `secret/` path may not have the KV version 2 engine enabled. Enable it using the Vault CLI:
```bash
vault secrets enable -path=secret -version=2 kv
```

*Note: If the engine is already enabled, Vault will notify you that the path is in use. If the CLI is not available, you can also perform this step using `curl`:*
```bash
curl \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"type":"kv","options":{"version":"2"}}' \
  http://127.0.0.1:8200/v1/sys/mounts/secret
```

### Step 3: Populate the Secret
Write a new secret entry named `mlflow` inside the KV engine carrying the `admin_password` field:
```bash
vault kv put secret/mlflow admin_password='Admin@123'
```

*Alternatively, using the rest HTTP API:*
```bash
curl \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  -X POST \
  -d '{"data":{"admin_password":"Admin@123"}}' \
  http://127.0.0.1:8200/v1/secret/data/mlflow
```

### Step 4: Verify Secret Registration
Confirm that Vault records the secret version and values correctly:
```bash
vault kv get secret/mlflow
```

Or query the REST endpoint:
```bash
curl \
  -H "X-Vault-Token: $VAULT_TOKEN" \
  http://127.0.0.1:8200/v1/secret/data/mlflow
```
Expected output:
```json
{
  "data": {
    "data": {
      "admin_password": "Admin@123"
    }
  }
}
```

### Step 5: Verify MLflow Server Initialization
The wrapper script polls the `secret/mlflow` path in Vault. Once written, wait 5–10 seconds and test if the MLflow daemon successfully bootstraps:
```bash
curl -I http://127.0.0.1:5000/
```
Expected response:
```
HTTP/1.1 200 OK
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **HashiCorp Vault** | A centralized secrets management system that securely stores, encrypts, and audits API keys, passwords, certificates, and other sensitive credentials. |
| **KV Secrets Engine** | Vault's Key-Value secrets storage. Version 2 (v2) adds support for maintaining a history of secret versions, enabling safe rollback and key rotation. |
| **Decoupled Bootstrapping**| A container/VM design pattern where running services poll or request database passwords from Vault during start-up, removing plaintext credentials from build images. |
| **API-Driven Configuration**| Vault exposes its entire functionality (auth, engine mounts, read/write secrets) through a standard HTTP REST API, allowing platform toolings to interact without CLI dependencies. |
| **Secrets Engine Mounting** | The mechanism by which Vault maps request paths (e.g. `/secret`) to specific backend storage plugins (e.g. KV version 2) handling data operations. |

## Summary
To remove hardcoded admin credentials from the MLflow startup scripts, a HashiCorp Vault KV version 2 secrets engine was enabled at the `secret/` path. The administrator password (`Admin@123`) was registered under `secret/mlflow`. The local server boot wrapper detected this new KV entry during its polling cycle, retrieved the credential, and securely initiated the MLflow tracking service on port `5000`, validating a decoupled, Vault-first secret configuration pattern.
