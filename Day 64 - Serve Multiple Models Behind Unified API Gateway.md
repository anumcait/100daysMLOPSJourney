# Day 64: Serve Multiple Models Behind a Unified API Gateway
The xFusionCorp Industries ML platform team operates three models for fraud detection and customer management, accessible via a single Nginx reverse proxy on port 8085: /fraud/, /churn/, and /recommend/, each routing to its respective Flask container. The fraud and churn services are already integrated in the docker-compose.yml and nginx.conf files. The directory for the recommend service, which contains a functioning application and Dockerfile, is available on disk. Your task is to integrate the recommend service into the docker-compose.yml, add the corresponding upstream and location block to the Nginx configuration, launch the stack, and verify that all routes respond appropriately.


The Docker daemon is already running. Base images (python:3.11-slim, nginx:alpine) are being pulled in the background at startup, so the first docker compose up -d returns in seconds.

The project layout under /root/code/serving/multi-model/:

fraud/app.py + fraud/Dockerfile – Flask service returning {"service": "fraud", "is_fraud": ...}. Correct.
churn/app.py + churn/Dockerfile – Flask service returning {"service": "churn", "churn_risk": ...}. Correct.
recommend/app.py + recommend/Dockerfile – Flask service returning {"service": "recommend", "items": [...]}. Correct — but not yet referenced by compose or nginx.
docker-compose.yml – Declares fraud, churn, and nginx services. The recommend service block is missing.
nginx.conf – Routes /fraud/ and /churn/ to their container upstreams. The recommend upstream + location block is missing.
The end state must include:

docker-compose.yml declares a recommend service that builds from ./recommend and carries container_name: mm-recommend.
nginx.conf declares a recommend upstream (server recommend:5000;) and a location /recommend/ block that proxies to it.
docker compose ps reports all four containers (mm-fraud, mm-churn, mm-recommend, mm-nginx) as Up.
curl -X POST http://localhost:8085/fraud/predict -d '{...}' returns a JSON body with "service": "fraud".
curl -X POST http://localhost:8085/churn/predict -d '{...}' returns a JSON body with "service": "churn".
curl -X POST http://localhost:8085/recommend/predict -d '{...}' returns a JSON body with "service": "recommend" and a non-empty items array.
Model the new entries on the existing fraud and churn blocks—same structure, same naming convention. After editing both files, docker compose up -d reads the new compose entry and builds the recommend image; nginx mounts the updated config from the host filesystem at container start.

## Objective

Integrate a new **recommend** model into an existing multi-model ML serving stack using **Docker Compose** and **Nginx**. The stack exposes three Flask services behind a single reverse proxy:

- `/fraud/`
- `/churn/`
- `/recommend/`

---

## Project Structure

```text
/root/code/serving/multi-model/
├── docker-compose.yml
├── nginx.conf
├── fraud/
│   ├── app.py
│   └── Dockerfile
├── churn/
│   ├── app.py
│   └── Dockerfile
└── recommend/
    ├── app.py
    └── Dockerfile
```

The `fraud` and `churn` services were already configured. The `recommend` service existed but was not referenced by either Docker Compose or Nginx.

---

## Step 1: Update `docker-compose.yml`

Added a new service for the recommendation model.

```yaml
services:
  fraud:
    build: ./fraud
    container_name: mm-fraud

  churn:
    build: ./churn
    container_name: mm-churn

  recommend:
    build: ./recommend
    container_name: mm-recommend

  nginx:
    image: nginx:alpine
    container_name: mm-nginx
    ports:
      - "8085:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - fraud
      - churn
      - recommend
```

---

## Step 2: Update `nginx.conf`

Added an upstream for the recommendation service.

```nginx
upstream recommend {
    server recommend:5000;
}
```

Added a new location block inside the `server` section.

```nginx
location /recommend/ {
    proxy_pass http://recommend/;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

The configuration mirrors the existing `/fraud/` and `/churn/` routes.

---

## Step 3: Start the Stack

```bash
cd /root/code/serving/multi-model
docker compose up -d
```

---

## Step 4: Verify Containers

```bash
docker compose ps
```

Expected output:

```
NAME             STATUS
mm-fraud         Up
mm-churn         Up
mm-recommend     Up
mm-nginx         Up
```

---

## Step 5: Verify the APIs

### Fraud Prediction

```bash
curl -X POST http://localhost:8085/fraud/predict \
  -H "Content-Type: application/json" \
  -d '{}'
```

Expected:

```json
{
  "service": "fraud",
  "is_fraud": false
}
```

---

### Churn Prediction

```bash
curl -X POST http://localhost:8085/churn/predict \
  -H "Content-Type: application/json" \
  -d '{}'
```

Expected:

```json
{
  "service": "churn",
  "churn_risk": 0.24
}
```

---

### Recommendation Service

```bash
curl -X POST http://localhost:8085/recommend/predict \
  -H "Content-Type: application/json" \
  -d '{}'
```

Expected:

```json
{
  "service": "recommend",
  "items": [
    "item1",
    "item2",
    "item3"
  ]
}
```

---

## Key Learnings

- Docker Compose makes it simple to add new microservices by defining additional service blocks.
- Nginx can expose multiple backend services behind a single API gateway using upstreams and location blocks.
- Reverse proxies simplify client access by presenting a unified endpoint while routing requests internally.
- Consistent service naming in Docker Compose allows Nginx to resolve containers automatically through Docker's internal DNS.

---

## Outcome

Successfully integrated the **recommend** model into the existing ML serving platform by:

- Adding the `recommend` service to Docker Compose.
- Configuring Nginx to proxy `/recommend/` requests.
- Launching all four containers successfully.
- Verifying that all three model endpoints responded through the unified API gateway.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/7ad6b138-9665-4dd7-b27c-17b6e51ac0b0" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/eb625f1c-09b8-4530-b956-11b20858aca3" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e520d6ea-a6ba-4d0c-a3b2-bc529eaf54c8" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/641ed7c1-7b9c-4472-bf2a-3b2ec0796d03" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d372e58c-4560-4b0b-8436-5d8b4f0c5afc" />





