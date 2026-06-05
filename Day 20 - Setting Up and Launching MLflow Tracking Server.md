# Day 20: Setting Up and Launching MLflow Tracking Server

The xFusionCorp Industries ML team is expanding its experimentation capabilities. While DVC handles data and pipeline versioning, the team needs MLflow to provide a rich UI for real-time metric tracking, parameter comparison, and model artifact management. Today, you will initialize the tracking infrastructure.

## Objective
Configure and launch a persistent MLflow Tracking Server using a SQLite backend and a local artifact store, ensuring the server is accessible through the lab's proxy.

## Task Overview
- **Storage Strategy**: Use SQLite for the backend store (metadata) and a local directory for the artifact store (files).
- **Network Configuration**: Bind the server to all interfaces and configure CORS/Allowed Hosts for proxy compatibility.
- **Persistence**: Run the server as a background process using `nohup`.

## Steps to Complete

### 1. Create Storage Directories
Ensure the directories for the backend database and the artifacts are in place.
```bash
mkdir -p /root/code/mlflow-backend
mkdir -p /root/code/mlflow-artifacts
```

### 2. Start the MLflow Tracking Server
Launch the server in the background, specifying the SQLite URI and the artifact destination.
```bash
nohup mlflow server \
  --host 0.0.0.0 \
  --port 5000 \
  --backend-store-uri sqlite:////root/code/mlflow-backend/mlflow.db \
  --artifacts-destination /root/code/mlflow-artifacts \
  --cors-allowed-origins '*' \
  --allowed-hosts '*' \
  > /root/mlflow-server.log 2>&1 &
```

### 3. Verify Server Status
Confirm the process is running and the database has been initialized.
- **Process Check**: `ps -ef | grep mlflow`
- **Database Check**: `ls -l /root/code/mlflow-backend/mlflow.db`
- **Port Check**: `ss -tulpn | grep 5000`

### 4. Access the UI
Open the **MLflow UI** button at the top of your lab interface. You should see the MLflow dashboard with the "Default" experiment already created.

## Summary
By the end of this day, you have a functional MLflow Tracking Server that serves as the central hub for all future model training experiments.
