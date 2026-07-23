# Day 61: Deploy a Model-Serving Container via Portainer
The xFusionCorp Industries ML platform team utilizes Portainer to manage every model-serving container. This operational layer allows for image inspection, container deployment, and runtime log management from a single web console. The fraud-detector:v1 image has been successfully built, and Portainer is currently accessible on port 9090.

Your task is to deploy the serving container named fraud-api using the Portainer UI. You are required to bind-mount the pre-staged /root/code/serving/ directory to ensure the container can locate app.py and model.pkl. Additionally, you must publish the host port 8085 and verify that the endpoints /health and /predict respond correctly on the host.

Portainer is already running on port 9090. The Portainer UI button at the top of the lab opens the login page. Admin credentials: username admin, password xFusionCorp2026! (pre-initialised at startup). The deploy is driven entirely from Portainer's Add container form in the local environment.

The project layout under /root/code/serving/:
app.py – FastAPI app loading /app/model.pkl and exposing /health + POST /predict. Correct.
Dockerfile – python:3.11-slim + fastapi + uvicorn + joblib + sklearn, EXPOSE 8085, CMD ["uvicorn", "app:app", ...]. Correct.
model.pkl – RandomForest trained at startup.
The image fraud-detector:v1 has already been built from the Dockerfile. The container is expected to bind-mount /root/code/serving/ → /app so the same image picks up new app.py or model.pkl versions on restart, and to publish host port 8085 → container 8085.

The end state must include:

Portainer is reachable on port 9090.
docker inspect fraud-api reports the container as running, using image fraud-detector:v1, with the bind-mount /root/code/serving → /app.
curl http://localhost:8085/health returns {"status":"ok"} with HTTP 200.
curl -X POST http://localhost:8085/predict -H 'Content-Type: application/json' -d '{"amount":3200,"hour":23,"num_tx_past_day":5}' returns a JSON body with an is_fraud field of 0 or 1.
Portainer's Add container form collects every field the equivalent docker run command would—name, image, port mapping, volume mount, env vars, restart policy—and runs the deploy through the mounted /var/run/docker.sock. The host path entered in the Volumes tab must match the lab's host filesystem exactly (/root/code/serving).

## Objective

Deploy the `fraud-detector:v1` model-serving container using Portainer UI.

Requirements:

- Container name: `fraud-api`
- Image: `fraud-detector:v1`
- Bind mount: `/root/code/serving` → `/app`
- Port mapping: `8085:8085`
- Verify `/health` and `/predict` endpoints

---

## Portainer Setup

Portainer URL:

```text
http://<host>:9090
```

Login:

```text
Username: admin
Password: xFusionCorp2026!
```

Portainer Version:

```text
Portainer Community Edition 2.39.4 LTS
```

Docker environment:

```text
unix:///var/run/docker.sock
```

---

## Verify Portainer Docker Connection

Check Docker socket mount:

```bash
docker inspect portainer --format '{{json .Mounts}}'
```

Expected:

```text
/var/run/docker.sock -> /var/run/docker.sock
```

---

# Deploy Container Through Portainer

Navigate:

```
Portainer → local → Containers → Add container
```

## Container Configuration

| Setting | Value |
|---|---|
| Name | `fraud-api` |
| Image | `fraud-detector:v1` |

---

## Port Mapping

Configure:

| Host | Container |
|---|---|
| 8085 | 8085 |

Equivalent Docker option:

```bash
-p 8085:8085
```

---

## Volume Mount

Add a bind mount:

| Host Path | Container Path |
|---|---|
| `/root/code/serving` | `/app` |

Equivalent Docker option:

```bash
-v /root/code/serving:/app
```

This allows the container to load:

```
/app/app.py
/app/model.pkl
```

---

## Restart Policy

Set:

```
Always
```

---

# Equivalent Docker Command

```bash
docker run -d \
  --name fraud-api \
  -p 8085:8085 \
  -v /root/code/serving:/app \
  --restart always \
  fraud-detector:v1
```

---

# Verification

## Check Container

```bash
docker ps | grep fraud-api
```

Confirm:

- Container is running
- Image is `fraud-detector:v1`

---

## Check Mount

```bash
docker inspect fraud-api
```

Expected:

```
/root/code/serving -> /app
```

---

# API Testing

## Health Check

```bash
curl http://localhost:8085/health
```

Expected response:

```json
{"status":"ok"}
```

HTTP status:

```
200 OK
```

---

## Prediction Test

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

Expected response:

```json
{"is_fraud":0}
```

or

```json
{"is_fraud":1}
```

---

# Final Checklist

- [x] Portainer available on port 9090
- [x] Local Docker environment connected
- [x] Container deployed through Portainer
- [x] Container name: `fraud-api`
- [x] Image: `fraud-detector:v1`
- [x] Port mapping: `8085:8085`
- [x] Bind mount: `/root/code/serving:/app`
- [x] Health endpoint working
- [x] Prediction endpoint working

---

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b2cfa07a-fd18-4bba-bc75-cd3139ef540c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/dbfec68b-0ff2-49bb-9892-b3e86753dad5" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ca528a78-0c4a-47f3-aa34-d55ce7525ed9" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4e211915-c3bf-4e85-961d-84e8c585852b" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3886d002-c0e6-45d1-b0d3-09dc526b36a2" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c3fe4822-e81f-4a17-8ba6-3fa9db43f0c2" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/38730e6b-f61e-45be-ac8e-86548427ef4b" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c7eeff0a-edd4-4c40-bd7a-2b926f9784ec" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9528deb6-7e95-4d79-bfe1-9307e18cd869" />
 <img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8683f120-babe-44e1-8e57-7260e73dd9cd" />
 











