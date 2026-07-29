# Day 66: Production Model Serving with Docker Compose
The xFusionCorp Industries ML platform team operates a fraud-detection model in production, supported by a comprehensive observability stack. This stack includes a Flask API with Prometheus instrumentation, Redis for per-IP rate limiting, nginx as the public reverse proxy, Prometheus for metrics collection, and Grafana for dashboarding. Currently, the pre-staged stack located at /root/code/serving/production/ does not reach a clean end state. Your objective is to correct the wiring, bring the stack online, and create a Grafana dashboard that visualizes the model API's request rate.


The Docker daemon is already running. Every image the compose stack references is being pre-pulled in the background at startup, so docker compose up -d returns in seconds. The Grafana admin password is grafana2026.

Bring the stack up and observe where it falls short: cd /root/code/serving/production && docker compose up -d && docker compose ps shows the container states, and docker compose logs surfaces the wiring faults behind any container that does not settle.

The project layout under /root/code/serving/production/:

app/app.py – Flask API with /health, /predict (Redis-backed per-IP rate limit), and /metrics (once the exporter is wired). Needs attention.
app/Dockerfile – python:3.11-slim + flask + redis + prometheus-flask-exporter + joblib + sklearn. Correct.
model.pkl – Trained at startup on the shared synthetic fraud dataset.
docker-compose.yml – Defines model-api, redis, nginx (publishes 8085), prometheus (publishes 9090), grafana (publishes 3000, admin password grafana2026), and a traffic-generator sidecar (continuously POSTs to /predict so Grafana has live request-rate data to plot). Correct.
prometheus.yml – Scrape config for the model-api job. Needs attention.
nginx.conf – Reverse-proxy config with an upstream model_backend block + location / forwarding every request. Needs attention.
grafana/provisioning/datasources/prometheus.yml – Pre-provisions a Prometheus datasource pointing at http://prometheus:9090, so the Grafana task focuses on dashboard creation.
The Grafana UI button opens the console once the stack is up; the dashboard should carry at least one panel that queries the Prometheus datasource.

The end state must include:

All six containers (model-api, prod-redis, prod-nginx, prod-prometheus, prod-grafana, prod-traffic) are reported running by docker inspect.
curl -s http://localhost:5000/metrics returns a Prometheus exposition-format body (HTTP 200).
curl -X POST http://localhost:8085/predict -d '{...}' through nginx returns a JSON is_fraud response.
curl http://localhost:9090/api/v1/targets reports the model-api job's health as up.
curl -u admin:grafana2026 http://localhost:3000/api/datasources lists a Prometheus datasource.
curl -u admin:grafana2026 http://localhost:3000/api/search?type=dash-db returns at least one user-created dashboard, and that dashboard's JSON carries at least one panel.
The Flask app listens on container port 5000. The prometheus-flask-exporter publishes standard HTTP request counters that a Grafana panel can query to plot the API's request rate.

## Objective

Deploy and fix a production-grade fraud detection model serving stack with:

- Flask API
- Prometheus metrics exporter
- Redis-based IP rate limiting
- nginx reverse proxy
- Prometheus monitoring
- Grafana dashboarding
- Docker Compose orchestration

The goal was to bring the pre-staged production stack into a clean working state and create a Grafana dashboard showing API request rate.

---

## Project Structure

```
production/
├── app/
│   ├── app.py
│   └── Dockerfile
├── model.pkl
├── docker-compose.yml
├── prometheus.yml
├── nginx.conf
└── grafana/
    └── provisioning/
        └── datasources/
            └── prometheus.yml
```

---

# Initial Issues Found

The Docker Compose configuration was already correct.

The issues were in:

- `app/app.py`
- `prometheus.yml`
- `nginx.conf`

---

# Fix 1: Enable Prometheus Metrics in Flask

## Problem

`prometheus_flask_exporter` was imported but not initialized.

The `/metrics` endpoint was unavailable because the exporter was never attached to the Flask application.

## Before

```python
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
```

## After

```python
from prometheus_flask_exporter import PrometheusMetrics

app = Flask(__name__)
metrics = PrometheusMetrics(app)
```

This exposed:

```
GET /metrics
```

with Prometheus-compatible metrics.

---

# Fix 2: Correct Prometheus Target

## Problem

Prometheus was scraping the wrong port.

Flask runs inside the container on:

```
5000
```

but Prometheus was configured for:

```
8000
```

## Before

```yaml
scrape_configs:
  - job_name: model-api
    static_configs:
      - targets:
          - model-api:8000
```

## After

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: model-api
    static_configs:
      - targets:
          - model-api:5000
```

---

# Fix 3: Correct nginx Reverse Proxy

## Problem

nginx was forwarding traffic to port `8000`.

The Flask API listens on port `5000`.

## Before

```nginx
upstream model_backend {
    server model-api:8000;
}
```

## After

```nginx
upstream model_backend {
    server model-api:5000;
}
```

Final nginx configuration:

```nginx
events {}

http {
    upstream model_backend {
        server model-api:5000;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://model_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

---

# Start the Stack

```bash
cd /root/code/serving/production

docker compose down

docker compose up -d --build
```

---

# Verify Containers

Command:

```bash
docker compose ps
```

Final state:

| Container | Status |
|---|---|
| model-api | Running (healthy) |
| prod-redis | Running |
| prod-nginx | Running |
| prod-prometheus | Running |
| prod-grafana | Running |
| prod-traffic | Running |

---

# Verify Flask Metrics

Command:

```bash
curl -s http://localhost:5000/metrics
```

Metrics successfully returned:

```
# HELP flask_http_request_total Total number of HTTP requests
# TYPE flask_http_request_total counter

flask_http_request_total{method="POST",status="200"} 159.0
flask_http_request_total{method="POST",status="429"} 19.0
```

---

# Verify Prediction API Through nginx

Command:

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":500,"hour":12,"num_tx_past_day":2}'
```

Response:

```json
{
  "is_fraud": 0
}
```

Note:

A rate-limit response:

```json
{
  "error": "rate limit exceeded"
}
```

is expected after the traffic generator sends many requests.

---

# Verify Prometheus Target

Command:

```bash
curl http://localhost:9090/api/v1/targets
```

Result:

```json
{
  "job": "model-api",
  "health": "up",
  "scrapeUrl": "http://model-api:5000/metrics"
}
```

Prometheus successfully scraped the Flask API.

---

# Grafana Setup

Grafana credentials:

```
Username: admin
Password: grafana2026
```

Prometheus datasource was already provisioned:

```
http://prometheus:9090
```

---

# Create Grafana Dashboard

Dashboard name:

```
Fraud Model API Request Rate
```

Panel query:

```promql
sum(rate(flask_http_request_total[1m]))
```

This displays API requests per second.

---

# Verify Dashboard API

Command:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/search?type=dash-db
```

Result:

```json
[
  {
    "title": "Fraud Model API Request Rate",
    "type": "dash-db"
  }
]
```

Dashboard successfully created.

---

# Final Architecture

```
              Users
                |
                |
             nginx
          :8085 /predict
                |
                |
          Flask API
          :5000
          |
     +------+------+
     |             |
   Redis       Prometheus
 rate limit      |
                 |
              Grafana
              :3000
```

---

# Key Learnings

- Docker Compose service names work as internal DNS names.
- Container ports must match application listening ports.
- Prometheus targets must point to container ports, not host ports.
- nginx upstream configuration must match backend service ports.
- Prometheus exporters must be initialized before metrics are available.
- Grafana dashboards can visualize Prometheus counters using `rate()` queries.

---

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/349aff86-1822-43ef-b197-c750ab836e79" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/cfbfa3ff-f96f-4ec7-9a81-e4b9152b5e5f" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2b04056a-3678-4e2f-86d4-ebd6c8e94bcd" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c676637e-03ec-46ea-ad85-ff50bc7c4fb3" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1e843e04-fe2e-40b3-b368-4be30ca26750" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/41d0799d-f4d7-4018-b5fa-333800c776be" />

---





