# Day 55: Fix a Broken Dockerfile HEALTHCHECK and EXPOSE
The xFusionCorp Industries ML platform team deploys Flask-based inference APIs as Docker images, incorporating Docker-native HEALTHCHECK instructions. This allows operators to easily verify the serving status of the process by running docker inspect --format='{{.State.Health.Status}}'. However, the draft Dockerfile located at /root/code/ml-health/ currently does not meet these standards; executing docker inspect --format='{{.State.Health.Status}}' ml-health-api returns unhealthy, and docker inspect --format '{{.Config.ExposedPorts}}' ml-health:v1 shows no exposed ports. Your task is to rectify the HEALTHCHECK target and include the missing EXPOSE declaration.


The Docker daemon is already running. docker version can be run in a VS Code terminal to confirm. With the current image built and run as ml-health-api, docker inspect --format='{{.State.Health.Status}}' ml-health-api reports unhealthy and docker inspect --format '{{.Config.ExposedPorts}}' ml-health:v1 shows no exposed ports.

The project layout under /root/code/ml-health/:

app.py – Flask API serving GET /health (returns {"status": "ok"} / 200) and POST /predict (returns a rule-based fraud flag) on port 8085. Correct.
Dockerfile – python:3.11-slim, installs flask, copies app.py, carries a HEALTHCHECK + CMD. The corrected image is built as ml-health:v1 and run as a container named ml-health-api with host port 8085 published.
The end state must include:

docker inspect --format '{{.Config.ExposedPorts}}' ml-health:v1 reports map[8085/tcp:{}].
After docker run, docker inspect --format='{{.State.Health.Status}}' ml-health-api reads healthy within ~15 seconds.
curl http://localhost:8085/health returns {"status": "ok"} with HTTP 200.
HEALTHCHECK reruns its CMD every --interval seconds and flips the state to unhealthy after --retries consecutive failures. EXPOSE does not change networking (that is done by -p)—it writes image metadata so docker inspect and downstream orchestrators know which port the image intends to serve on.

## Task

The xFusionCorp Industries ML platform team deploys Flask-based inference APIs as Docker images with Docker-native `HEALTHCHECK` instructions.

The existing Dockerfile had two issues:

1. The `HEALTHCHECK` target was incorrect.
2. The Docker image did not expose the application port.

The Flask API serves:

- `GET /health` → returns `{"status": "ok"}` with HTTP 200
- `POST /predict` → rule-based fraud prediction

The application runs on port **8085**.

---

## Problem

The original Dockerfile contained:

```dockerfile
HEALTHCHECK --interval=5s --timeout=3s --start-period=3s --retries=3 \
  CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8085/healthz')" || exit 1
```

The health check failed because:

- The API endpoint is `/health`
- The configured endpoint `/healthz` does not exist

The Docker image also lacked:

```dockerfile
EXPOSE 8085
```

Without `EXPOSE`, Docker image metadata does not declare the intended service port.

---

## Fixed Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir flask

COPY app.py /app/app.py

EXPOSE 8085

HEALTHCHECK --interval=5s --timeout=3s --start-period=3s --retries=3 \
  CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8085/health')" || exit 1

CMD ["python3", "/app/app.py"]
```

---

## Build Image

```bash
cd /root/code/ml-health

docker build -t ml-health:v1 .
```

---

## Run Container

Remove the old container:

```bash
docker rm -f ml-health-api
```

Start the updated container:

```bash
docker run -d \
  --name ml-health-api \
  -p 8085:8085 \
  ml-health:v1
```

---

## Verification

### Check Exposed Ports

Command:

```bash
docker inspect --format '{{.Config.ExposedPorts}}' ml-health:v1
```

Expected output:

```text
map[8085/tcp:{}]
```

---

### Check Container Health

Wait around 15 seconds, then run:

```bash
docker inspect --format='{{.State.Health.Status}}' ml-health-api
```

Expected output:

```text
healthy
```

---

### Test API Endpoint

Command:

```bash
curl http://localhost:8085/health
```

Expected response:

```json
{"status":"ok"}
```

HTTP status:

```text
200 OK
```

---

## Key Concepts Learned

### Docker HEALTHCHECK

`HEALTHCHECK` allows Docker to monitor whether a containerized application is working correctly.

Example:

```dockerfile
HEALTHCHECK --interval=5s --timeout=3s --retries=3 CMD <health-check-command>
```

- `--interval` → how often Docker runs the health check
- `--timeout` → maximum time allowed for each check
- `--retries` → consecutive failures required before marking unhealthy

Docker changes the container health state:

- `starting`
- `healthy`
- `unhealthy`

---

### Docker EXPOSE

`EXPOSE` documents the port that the containerized application listens on.

Example:

```dockerfile
EXPOSE 8085
```

It adds image metadata:

```text
map[8085/tcp:{}]
```

Important:

`EXPOSE` does **not** publish the port.

Port publishing is done using:

```bash
docker run -p 8085:8085 image-name
```

---

## Final Status

✅ Fixed HEALTHCHECK endpoint  
✅ Added EXPOSE 8085  
✅ Rebuilt Docker image  
✅ Container reports healthy  
✅ Flask health endpoint returns HTTP 200

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/811feef2-7ea1-4cdd-8caf-575beffe03fb" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/fde76dea-84a0-4d4f-b060-04316d983806" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e13d79a4-ae31-4444-95b2-e565840bb0f1" />



