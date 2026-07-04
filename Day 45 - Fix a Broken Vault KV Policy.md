# Day 45 - Fix a Broken Vault KV Policy

The xFusionCorp Industries ML platform team extended the Day-44 Vault wiring: instead of the MLflow boot wrapper reading `secret/mlflow` with the dev root token, it now uses a narrow `mlflow-reader` token bound to a Vault policy of the same name. A teammate authored the policy wrong, so the wrapper's GETs come back as permission denied and MLflow never boots. Your task is to fix the `mlflow-reader` policy in the Vault UI so the narrow token can read its own secret, and MLflow comes up on port 5000.

The Vault UI is on port 8200 (Vault button). The dev-mode root token is at `/root/code/vault-root-token`—use it to log in to the UI (policy editing is a privileged operation). The narrow token the MLflow wrapper is using is at `/root/code/vault-token`; do NOT use this one for login, it cannot edit policies.

The MLflow wrapper's next poll (within ~5 s) after the policy is corrected succeeds. Click the MLflow UI button to confirm that the tracker is live on port 5000.

The end state must include:
- `GET /v1/sys/policies/acl/mlflow-reader` still returns the policy – Do not rename or delete it.
- The policy's rules grant read on `secret/data/mlflow` (capabilities list contains `read`).
- The narrow token at `/root/code/vault-token` can now get `/v1/secret/data/mlflow` and receive the `admin_password` key.
- `http://localhost:5000/` answers 200.

Policies are Vault's authorisation layer. An auth method issues a token with a set of policy names attached; every API call then resolves capabilities by the policy's path rules. Narrow tokens (one service, one path, one capability) are the production pattern—root tokens are for bootstrap only. When a service 403s against Vault, the path-and-capability match in the relevant policy is almost always the first thing to check.

## Objective
Correct the access permissions in a Vault HCL policy to transition capabilities on `secret/data/mlflow` from write/update to `read`, allowing a restricted deployment token to query the MLflow administrator password secret and bootstrap the tracking server.

## Task
- **Authenticate with Vault UI**: Access the Vault console on port `8200` using the privileged root token found at `/root/code/vault-root-token`.
- **Diagnose Policy Error**: Inspect the ACL policy `mlflow-reader` to check for syntax errors, misconfigured path templates, or insufficient permissions.
- **Modify Policy Rules**: Edit the policy to replace `create` and `update` capabilities with the `read` capability for `secret/data/mlflow`.
- **Validate Access**: Verify that the restricted CLI token stored in `/root/code/vault-token` can retrieve the secrets and that the MLflow server successfully boots up on port `5000`.

## Solution

### Step 1: Log Into HashiCorp Vault UI
1. Click on the **Vault UI** button in your interface or navigate to `http://localhost:8200`.
2. Extract the root bootstrap token:
   ```bash
   cat /root/code/vault-root-token
   ```
3. Select **Token** as the Authentication Method in the login dashboard and paste the retrieved token to authenticate.

### Step 2: Locate and Edit the Broken Policy
1. Select the **Policies** tab from the top navigation bar.
2. Select the policy named **mlflow-reader**.
3. Observe the current mapping rule inside the editor:
   ```hcl
   # MLflow reader policy -- narrow KV access for the boot wrapper.
   path "secret/data/mlflow" {
     capabilities = ["create", "update"]
   }
   ```
   *Note: Because the application only performs GET requests, "create" and "update" are incorrect. It needs the "read" capability.*

4. Click **Edit policy** and change the capabilities array to only include `read`:
   ```hcl
   # MLflow reader policy -- narrow KV access for the boot wrapper.
   path "secret/data/mlflow" {
     capabilities = ["read"]
   }
   ```
5. Click **Save** to commit the changes.

### Step 3: Trigger and Verify Application Recovery
Once saved, the wrapper container polls the credentials path again in 5 seconds and successfully authenticates. 

Verify the API authorization changes via CLI:
```bash
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=$(cat /root/code/vault-token)

# Query the KV secret using the narrow token client
vault kv get secret/mlflow
```
Expected Output:
```
====== Data ======
Key               Value
---               -----
admin_password    Admin@123
```

Check the health status of the MLflow server:
```bash
curl -I http://127.0.0.1:5000/
```
Expected output:
```
HTTP/1.1 200 OK
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Vault ACL Policies** | Authorization rules written in HCL (HashiCorp Configuration Language) that specify path templates and capabilities defining what clients can do with Vault objects. |
| **Telemetry Capabilities** | Explicit operations supported by Vault backends: `read` (retrieving variables), `create`/`update` (creating/modifying secrets), `delete` (canceling keys), and `list` (indexing paths). |
| **KV Engine Path Mapping** | For KV version 2 engines, Vault automatically injects `/data/` as a segment between the mount path (e.g. `secret/`) and secret path (e.g. `mlflow`). Policies must declare rules matching the full API path: `secret/data/mlflow`. |
| **Narrow Privilege Tokens** | Staging tokens bound strictly to single tasks and specific target endpoints rather than broad administrative root credentials, supporting the least-privilege principal. |

## Summary
The MLflow boot wrapper was failing to launch because its restricted access token `/root/code/vault-token` had write/update permissions rather than read permissions mapped to the Vault credential store path. By logging into the Vault UI using the privileged root token `/root/code/vault-root-token` and editing the `mlflow-reader` policy to grant the `read` capability on `secret/data/mlflow`, authorization was established. The daemon successfully queried the admin password on its next polling window and bootstrapped the MLflow tracker server on port `5000`.
