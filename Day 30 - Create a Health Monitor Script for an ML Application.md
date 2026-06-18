# Day 30 - Create a Health Monitor Script for an ML Application
The xFusionCorp Industries MLOps team needs the fraud-detection-v2 candidate promoted all the way from the tracking server to live inference and monitored end to end. The backing stack (PostgreSQL, SeaweedFS, MLflow tracking server, three candidate runs) is already in place. Your task is to cover the remaining lifecycle work: model promotion, serving, and health monitoring.


The infrastructure is fully up: PostgreSQL container mlflow-db on port 5432, SeaweedFS on ports 8333 (S3 API) and 8888 (Filer UI), MLflow tracking server on port 5000 with the PostgreSQL backend and the mlflow-artifacts S3 bucket. The fraud-detection-v2 experiment contains three candidate runs (baseline, improved, regression) with logged f1_score metrics. The MLflow UI and SeaweedFS Filer buttons at the top of the lab can be opened to view each web UI.

The complete end state requires the following.

A registered model named fraud-detector-v2 exists in the MLflow Model Registry.
A champion alias on that model points at the version sourced from the fraud-detection-v2 run with the highest f1_score.
An mlflow models serve process listens on port 5001, serving the champion version (--env-manager=local is the supported choice for the lab). Export MLFLOW_TRACKING_URI=http://localhost:5000 in the serving shell so the models:/ URI can be resolved against the tracking server. The tracking server proxies the model download from SeaweedFS itself, so no S3 credentials are needed in the serving shell.
The served endpoint returns 200 on GET /health.
A shell script at /root/code/monitor.sh exists, is executable, probes the served model's /health endpoint once, and exits with status 0 when the endpoint is healthy.
The top run can be identified either through the MLflow UI Compare view or with a one-off MlflowClient.search_runs() call—whichever is preferable. The registration and alias assignment are likewise available from the UI (Models tab) or the SDK.

mlflow models serve is long-running; start it in the background, and ensure that the new process is listening on port 5001 before writing the monitoring script.

## Objective
Create a custom health monitor script that checks a local service endpoint and internal logic, returning appropriate exit codes.

## Task
1. Create a health endpoint in the application.
2. Create the monitor script (`monitor.sh`).
3. Make it executable.
4. Test the monitor.
5. Final verification.

## Solution

### Run these commands on the controlplane host:

#### Step 1: Create a health endpoint in the application
Assuming a simple Python/FastAPI app already exists on port 5001.

```python
@app.get("/health")
def health_check():
    return {"status": "ok"}
```

#### Step 2: Create the monitor script (/root/code/monitor.sh)

```bash
cat << 'EOF' > /root/code/monitor.sh
#!/bin/bash
STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5001/health)

if [ "$STATUS" -eq 200 ]; then
  echo "healthy"
  exit 0
fi

echo "unhealthy"
exit 1
EOF
```

#### Step 3: Make it executable:

```bash
chmod +x /root/code/monitor.sh
```

#### Step 4: Test the monitor

```bash
/root/code/monitor.sh
echo $?
```

**Expected output:**
```
healthy
0
```

#### Step 5: Final verification

```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:5001/health

/root/code/monitor.sh
echo $?
```

**Expected output:**
```
200
healthy
0
```

### Summary
Developing a custom health monitor script ensures that we can programmatically verify the health of our services. By checking specific endpoints and returning standard exit codes, this script can be easily integrated into automated monitoring systems or CI/CD pipelines to ensure service reliability.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5193f2a1-0ee1-4334-ae5a-33418be2291b" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e191a3e7-d839-4b65-9a2d-86e71e0bce8d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6c0ed936-29d9-46fa-83a3-eeaa9eabc267" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8af2b7d7-12f9-4ec5-8ff1-43157f2d3abf" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/507e5fd2-2054-4d59-912c-da132757cf85" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6f1128e0-3038-4e82-81a2-a597ae7eea34" />
