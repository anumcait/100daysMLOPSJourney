# Day 67: Add Prometheus as a Grafana Data Source
The xFusionCorp Industries ML platform team is in the process of implementing Grafana-based monitoring for their fraud-detection model. Although Prometheus and Grafana are already running in Docker, alongside a Flask metric emitter that provides live ML signals, Grafana currently lacks a configured data source, preventing the UI from accessing any metrics.

Your objective is to access the Grafana interface, configure the running Prometheus container as a data source through the Grafana UI, and create an initial dashboard panel that queries a live metric. This will ensure that the connection functions correctly end to end.


The Grafana UI is already running on port 3000. The Grafana button at the top of the lab opens the login page. Admin credentials: admin / grafana2026.

The stack running under /root/code/monitoring/ (via docker compose):

metric-emitter – Flask app exposing /metrics with flask_http_request_total{version}, prediction_accuracy, data_drift_score{column}, and model_inference_duration_seconds metrics. A background thread nudges the values every 5 seconds so panels built on top see real motion.
mon-prometheus – Prometheus, scraping metric-emitter:5000 every 5 seconds. Reachable inside the compose network as http://prometheus:9090.
mon-grafana – Grafana, no data sources configured.
The end state must include:

A data source of type prometheus exists in Grafana's configuration.
Its URL is http://prometheus:9090 (the compose service name – localhost:9090 does NOT work from inside the Grafana container).
Grafana's /api/datasources/uid/<uid>/health check reports status: OK.
At least one saved dashboard exists carrying a panel whose query targets a metric (a non-empty PromQL expression) — proof the data source actually returns data.
Grafana and Prometheus share a Docker network. Inside the Grafana container, localhost refers to Grafana itself, not to Prometheus.

## Objective

Configured Grafana to use the running Prometheus instance as a data source and verified end-to-end connectivity by creating a dashboard with a live Prometheus metric.

## Environment

- **Grafana:** Running on port `3000`
- **Prometheus:** Running as Docker service `prometheus:9090`
- **Metric Emitter:** Flask application exposing `/metrics`
- **Docker Compose Network:** Shared by Grafana and Prometheus

## Steps Performed

### 1. Logged into Grafana

Credentials:

- **Username:** `admin`
- **Password:** `grafana2026`

---

### 2. Added Prometheus Data Source

Navigated to:

```text
Connections → Data Sources → Add Data Source
```

Selected **Prometheus** and configured:

| Setting | Value |
|---------|-------|
| Name | prometheus |
| URL | `http://prometheus:9090` |

> **Note:** `localhost:9090` does not work because Grafana runs inside its own Docker container. The Docker service name (`prometheus`) must be used.

Saved the configuration and verified connectivity using **Save & Test**.

---

### 3. Verified Data Source Using API

List configured data sources:

```bash
curl -s -u admin:grafana2026 \
http://localhost:3000/api/datasources
```

Output:

```json
[
  {
    "id": 1,
    "uid": "dftmvdvfdrcaoc",
    "name": "prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090"
  }
]
```

---

### 4. Verified Health Check

```bash
curl -s -u admin:grafana2026 \
http://localhost:3000/api/datasources/uid/dftmvdvfdrcaoc/health
```

Response:

```json
{
  "status": "OK",
  "message": "Successfully queried the Prometheus API."
}
```

---

### 5. Created Dashboard

Created a new dashboard named:

```
ML Monitoring
```

Added a visualization using the Prometheus data source.

Example PromQL query:

```promql
prediction_accuracy
```

Other valid metrics:

```promql
flask_http_request_total
```

```promql
data_drift_score
```

```promql
model_inference_duration_seconds
```

Saved the dashboard.

---

### 6. Verified Dashboard via API

```bash
curl -s -u admin:grafana2026 \
http://localhost:3000/api/search
```

Output:

```json
[
  {
    "title": "ML Monitoring",
    "uid": "ad92hnb",
    "type": "dash-db"
  }
]
```

---

## Verification Checklist

- ✅ Prometheus data source created
- ✅ URL configured as `http://prometheus:9090`
- ✅ Data source health status is **OK**
- ✅ Dashboard saved successfully
- ✅ Dashboard contains a panel querying live Prometheus metrics

---

## Key Learning

When Grafana and Prometheus run inside Docker containers, **`localhost` refers to the current container**, not the host machine or another container.

Always use the Docker Compose **service name** for inter-container communication.

Example:

❌ Incorrect

```text
http://localhost:9090
```

✅ Correct

```text
http://prometheus:9090
```

Docker's internal DNS automatically resolves the service name to the correct container.

---

## Commands Used

```bash
# List data sources
curl -s -u admin:grafana2026 \
http://localhost:3000/api/datasources

# Verify health
curl -s -u admin:grafana2026 \
http://localhost:3000/api/datasources/uid/dftmvdvfdrcaoc/health

# List dashboards
curl -s -u admin:grafana2026 \
http://localhost:3000/api/search
```

---

## Outcome

Successfully configured Grafana with Prometheus as the data source, verified API connectivity, and created a dashboard displaying live metrics from the Flask metric emitter, completing the monitoring pipeline.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2bf844cb-da1f-468e-aa89-3c1a816c95ae" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/dc14d5c8-1086-4b99-8608-ce201defdbf0" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d2feda11-e01a-44a8-948c-d592cc3f83b0" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/538761b8-1703-449e-a6b8-9b327169229f" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/0560affb-1c4b-4aff-9ac4-18403c562d8d" />





