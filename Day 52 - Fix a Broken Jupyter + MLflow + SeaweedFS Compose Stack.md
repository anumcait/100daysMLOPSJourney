# Day 52: Fix a Broken Jupyter + MLflow + SeaweedFS Compose Stack
The xFusionCorp Industries ML platform team has provided a local development stack comprised of Jupyter Lab for notebooks, MLflow for experiment tracking, and SeaweedFS for S3-compatible artifact storage, all encapsulated within a three-service docker compose deployment. A docker-compose.yml file is available at /root/code/ml-dev/, but it is currently misconfigured, resulting in the stack not making all three browser UIs accessible on their standard ports.

Your objective is to correct the docker-compose.yml configuration so that each service is accessible on its appropriate standard port without requiring login prompts.


The Docker daemon is already running and every image has been pre-pulled in the background at startup, so bringing the stack up returns in seconds on the first run. Run docker compose -f /root/code/ml-dev/docker-compose.yml up -d then docker compose -f /root/code/ml-dev/docker-compose.yml ps to see which UIs are reachable on their standard ports.

The project layout under /root/code/ml-dev/:

docker-compose.yml – Three services:
jupyter – Container ml-jupyter, host port 8888.
mlflow – Container ml-mlflow, host port 5000.
seaweedfs – Container ml-seaweedfs. SeaweedFS serves the S3 API on container port 8333 and the Filer UI on container port 8888. The lab's convention is host port 9000 for the S3 API and host port 9001 for the Filer UI.
The end state must include:

All three containers (ml-jupyter, ml-mlflow, ml-seaweedfs) reported Up by docker compose ps.
curl http://localhost:8888/ returns 200 or 302 – The Jupyter UI answers without prompting for a token.
curl http://localhost:5000/ returns 200 – The MLflow UI answers on the standard port.
curl http://localhost:9001/ returns 200 or 302 – The SeaweedFS Filer UI answers on its standard host port (the SeaweedFS S3 API stays on host 9000).
The three browser UIs (Jupyter, MLflow, SeaweedFS Filer) are the primary verification surface — open them from the buttons at the top of the lab.

## Objective

Fix a misconfigured Docker Compose stack so that:

- Jupyter Lab is accessible on **http://localhost:8888** without a login token.
- MLflow UI is accessible on **http://localhost:5000**.
- SeaweedFS S3 API is available on **http://localhost:9000**.
- SeaweedFS Filer UI is available on **http://localhost:9001**.

---

## Issues Found

- SeaweedFS port mappings were swapped:
  - `9001:8333`
  - `9000:8888`
- Jupyter started with its default configuration, which required a token to access the UI.

---

## Changes Made

### 1. Fixed SeaweedFS Port Mapping

**Before**

```yaml
ports:
  - "9001:8333"
  - "9000:8888"
```

**After**

```yaml
ports:
  - "9000:8333"
  - "9001:8888"
```

---

### 2. Disabled Jupyter Token Authentication

Added the following command to the Jupyter service:

```yaml
command:
  - start-notebook.sh
  - --ServerApp.token=
  - --ServerApp.password=
```

This allows the Jupyter UI to open directly without prompting for a token.

---

### 3. Verified MLflow Configuration

The MLflow service was already correctly configured to expose port **5000** and required no additional changes.

---

## Commands Used

Stop the existing stack:

```bash
docker compose -f /root/code/ml-dev/docker-compose.yml down
```

Start the updated stack:

```bash
docker compose -f /root/code/ml-dev/docker-compose.yml up -d
```

Check container status:

```bash
docker compose -f /root/code/ml-dev/docker-compose.yml ps
```

Verify endpoints:

```bash
curl -I http://localhost:8888/
curl -I http://localhost:5000/
curl -I http://localhost:9001/
```

---

## Expected Results

- ✅ `ml-jupyter` container is **Up**
- ✅ `ml-mlflow` container is **Up**
- ✅ `ml-seaweedfs` container is **Up**
- ✅ Jupyter UI is accessible on **localhost:8888** without a token
- ✅ MLflow UI is accessible on **localhost:5000**
- ✅ SeaweedFS Filer UI is accessible on **localhost:9001**
- ✅ SeaweedFS S3 API is available on **localhost:9000**

---

## Key Learnings

- Docker port mappings follow the format `host_port:container_port`.
- A small port mapping mistake can make services appear unavailable.
- Jupyter Notebook images require explicit configuration to disable token authentication.
- `docker compose ps` and `curl` are quick ways to verify that services are running and reachable.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/99403b7d-2387-4a6a-8ac9-3e2810f0a639" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/85a19139-072d-4138-a7c3-e257320dc66e" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9ddd6a60-f066-4529-a822-560f70e46668" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b9bc9555-6535-4827-9d2f-cd5a7d1d3a35" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2ccefa7f-1658-4d4d-a369-45ad2d0bab4e" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/fadd2d77-2b0f-45ca-89ab-ff28dcd9b631" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b76f6066-e185-4d77-976c-e67c11508e0e" />







