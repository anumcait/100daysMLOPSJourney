# Day 29: Configure MLflow with Remote Tracking Server and Artifact Store
The xFusionCorp Industries ML platform team rolled out a shared MLflow deployment backed by a production-style tracking store (PostgreSQL) and a production-style artefact store (SeaweedFS, S3-compatible). The MLflow server is running and metadata writes to PostgreSQL succeed, but a teammate noticed that no artefact ever lands in the mlflow-artifacts SeaweedFS bucket — the connection from MLflow to SeaweedFS is not wired correctly. Diagnose what is broken in the MLflow server's startup configuration, fix it, and re-run the pre-staged smoke-test so one round trip lands metadata in PostgreSQL and the model artefact in SeaweedFS.


The pre-staged state:

PostgreSQL container mlflow-db is running on port 5432 (database mlflow, credentials mlflow / mlflow123).
SeaweedFS is running on port 8333 (S3 API) / 8888 (Filer UI), credentials weedadmin / weedadmin123, with a pre-created bucket mlflow-artifacts.
MLflow tracking server is running on port 5000 and was launched by /root/code/start-mlflow.sh. Its log is at /tmp/mlflow.log.
Reference scripts: /root/code/start-mlflow.sh (the MLflow startup command), /root/code/restart-mlflow.sh (kills the running server and re-launches via start-mlflow.sh), and /root/code/log_test_run.py (the smoke-test that exercises one full round trip).
Run the smoke-test once to observe the failure:

   python3 /root/code/log_test_run.py

The MLflow run appears in the MLflow UI (the metadata write to PostgreSQL succeeds), but the model artefact upload step raises an error because the MLflow server cannot reach the SeaweedFS bucket. The SeaweedFS Filer confirms /buckets/mlflow-artifacts/ is still empty.

Inspect /root/code/start-mlflow.sh and reconcile its environment with the SeaweedFS endpoint. Save the file, then restart the MLflow server:
   bash /root/code/restart-mlflow.sh

Re-run the smoke-test:
   python3 /root/code/log_test_run.py

The end state must include:
The test-remote experiment exists on the MLflow server with at least one successful run, visible in the MLflow UI.
The mlflow-artifacts bucket on SeaweedFS holds the run's model artefact (MLmodel + model.pkl), visible in the SeaweedFS Filer under /buckets/mlflow-artifacts/.
The PostgreSQL mlflow database holds the MLflow schema (the run's metadata).
PostgreSQL listens on port 5432 with a binary protocol — it is not reachable from a web browser. Use docker exec mlflow-db psql -U mlflow -d mlflow for manual inspection if needed.

## Task Description
The xFusionCorp Industries team is migrating their MLflow setup to a production-grade architecture using a centralized tracking server. The setup includes a PostgreSQL database for metadata and a SeaweedFS (S3-compatible) bucket for artifact storage. However, the current configuration is failing during artifact uploads because the tracking server is attempting to use the default AWS S3 endpoint instead of the local SeaweedFS endpoint.

### Requirements:
1. Identify and fix the missing environment variable in the MLflow startup script `/root/code/start-mlflow.sh`.
2. Set the `MLFLOW_S3_ENDPOINT_URL` to point to the SeaweedFS S3 API (`http://localhost:8333`).
3. Ensure `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are correctly configured for SeaweedFS.
4. Restart the MLflow server using `/root/code/restart-mlflow.sh`.
5. Validate the fix by running the smoke test script `/root/code/log_test_run.py`.
6. Confirm that artifacts are successfully stored in the `mlflow-artifacts` bucket in SeaweedFS.

## Solution

### Run these commands on the controlplane host:

#### Step 1: Update the MLflow Startup Script
Edit `/root/code/start-mlflow.sh` to include the SeaweedFS S3 endpoint and credentials.

```bash
# /root/code/start-mlflow.sh (Updated)
#!/bin/bash
set -e

# SeaweedFS Credentials
export AWS_ACCESS_KEY_ID=weedadmin
export AWS_SECRET_ACCESS_KEY=weedadmin123

# Point MLflow to the SeaweedFS S3 API
export MLFLOW_S3_ENDPOINT_URL=http://localhost:8333

exec mlflow server \
  --backend-store-uri postgresql://mlflow:mlflow123@localhost:5432/mlflow \
  --artifacts-destination s3://mlflow-artifacts \
  --host 0.0.0.0 --port 5000 \
  --allowed-hosts '*' --cors-allowed-origins '*'
```

#### Step 2: Restart the MLflow Tracking Server
Use the restart script to apply the changes.

```bash
bash /root/code/restart-mlflow.sh
```

#### Step 3: Run the Smoke Test
Verify that the server can now log runs and upload artifacts using the test script.

```bash
python3 /root/code/log_test_run.py
```

**Expected output:**
`test-remote run logged successfully`

#### Step 4: Verify Artifacts in SeaweedFS
Confirm the files reached the storage bucket.

```bash
# Using curl to list the bucket contents
curl http://localhost:8888/buckets/mlflow-artifacts/
```
Alternatively, browse the SeaweedFS Filer UI at `http://<host>:8888/buckets/mlflow-artifacts/`.

### Summary of Fix:
The root cause was the missing `MLFLOW_S3_ENDPOINT_URL` environment variable. Without it, MLflow defaulted to the standard AWS S3 endpoint, which caused connection failures. Adding the explicit SeaweedFS endpoint ensures all artifact operations are routed to the local storage service.

### Screenshots
<img width="1575" height="778" alt="image" src="https://github.com/user-attachments/assets/328b292b-da8a-47ae-b4c9-5180f9f60e9b" />
