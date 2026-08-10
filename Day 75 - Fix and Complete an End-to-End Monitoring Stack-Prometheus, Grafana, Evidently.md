# Day 75: Fix and Complete an End-to-End Monitoring Stack

## Overview

On Day 75, I fixed and completed an end-to-end ML monitoring stack for a fraud-detection platform.

The monitoring stack consisted of:

- **Evidently** — calculates feature drift scores
- **Flask Metric Emitter** — exposes ML serving and drift metrics
- **Prometheus** — scrapes and stores metrics
- **Grafana** — visualizes monitoring signals

The initial stack was running, but the monitoring pipeline was broken:

- Flask `/metrics` returned `404`
- Prometheus reported `metric-emitter` as `DOWN`
- Grafana panels were empty
- Evidently itself was healthy and continued generating drift scores

The goal was to identify and fix three integration bugs and create a tagged Grafana monitoring dashboard.

---

## Architecture

```text
                 ┌─────────────────────┐
                 │      Evidently      │
                 │    Drift Scorer     │
                 └──────────┬──────────┘
                            │
                     drift_scores.json
                            │
                            ▼
                 ┌─────────────────────┐
                 │   Flask Metric      │
                 │      Emitter        │
                 │      :5000          │
                 │                     │
                 │     /metrics        │
                 └──────────┬──────────┘
                            │
                       Prometheus
                         scrape
                            │
                            ▼
                 ┌─────────────────────┐
                 │     Prometheus      │
                 │       :9090         │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │       Grafana       │
                 │        :3000        │
                 └─────────────────────┘
```

---

## Environment

Stack location:

```bash
/root/code/monitoring/
```

Services:

| Service | Port | Purpose |
|---|---:|---|
| `metric-emitter` | `5000` | Flask Prometheus exporter |
| `mon-prometheus` | `9090` | Metrics collection |
| `mon-grafana` | `3000` | Visualization |
| Evidently UI | `8000` | Drift monitoring |

Grafana credentials:

```text
Username: admin
Password: grafana2026
```

---

# 1. Initial Diagnosis

First, I checked the running containers:

```bash
cd ~/code/monitoring
docker compose ps
```

All three containers were running:

```text
metric-emitter
mon-prometheus
mon-grafana
```

However, the monitoring pipeline was still broken.

---

# 2. Bug #1 — Flask Metrics Endpoint

The Flask application contained:

```python
@app.route("/prom-metrics")
def metrics():
    return generate_latest(REGISTRY), 200, {
        "Content-Type": CONTENT_TYPE_LATEST
    }
```

The expected Prometheus endpoint was:

```text
/metrics
```

but the application exposed:

```text
/prom-metrics
```

Therefore:

```bash
curl http://localhost:5000/metrics
```

returned:

```text
404
```

## Fix

Changed:

```python
@app.route("/prom-metrics")
```

to:

```python
@app.route("/metrics")
```

Command:

```bash
sed -i 's#@app.route("/prom-metrics")#@app.route("/metrics")#' \
  app/metric_emitter.py
```

---

# 3. Bug #2 — Prometheus Scrape Target

The original `prometheus.yml` contained:

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: metric-emitter
    static_configs:
      - targets:
          - metric-emitter:8000
```

The metric emitter actually listens on port `5000`.

Docker Compose exposes:

```yaml
ports:
  - "5000:5000"
```

Prometheus was therefore trying to scrape the wrong port.

## Fix

Changed:

```yaml
metric-emitter:8000
```

to:

```yaml
metric-emitter:5000
```

Command:

```bash
sed -i 's/metric-emitter:8000/metric-emitter:5000/' \
  prometheus.yml
```

Final configuration:

```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: metric-emitter
    static_configs:
      - targets:
          - metric-emitter:5000
```

---

# 4. Bug #3 — Grafana Prometheus Datasource

The original Grafana datasource configuration was:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9091
    isDefault: true
    editable: true
```

Prometheus actually listens on port `9090`.

Therefore Grafana was connecting to the wrong port.

## Fix

Changed:

```yaml
url: http://prometheus:9091
```

to:

```yaml
url: http://prometheus:9090
```

Command:

```bash
sed -i 's#http://prometheus:9091#http://prometheus:9090#' \
  grafana/provisioning/datasources/prometheus.yml
```

---

# 5. Reload the Stack

After fixing the configuration files:

```bash
docker compose down
docker compose up -d --build
```

Then:

```bash
sleep 10
docker compose ps
```

All three services were running successfully.

---

# 6. Verify Flask `/metrics`

Command:

```bash
curl -sf -o /dev/null -w "HTTP %{http_code}\n" \
  http://localhost:5000/metrics
```

Result:

```text
HTTP 200
```

The endpoint exposed metrics such as:

```text
flask_http_request_total
prediction_accuracy
data_drift_score
evidently_drift_share
model_inference_duration_seconds
```

---

# 7. Verify Prometheus Target

Command:

```bash
curl -s http://localhost:9090/api/v1/targets | \
  jq '.data.activeTargets[] |
      select(.labels.job=="metric-emitter") |
      {
        job,
        health,
        scrapeUrl,
        lastError
      }'
```

Result:

```json
{
  "job": "metric-emitter",
  "health": "up",
  "scrapeUrl": "http://metric-emitter:5000/metrics",
  "lastError": ""
}
```

This confirmed that Prometheus could successfully scrape the Flask exporter.

---

# 8. Verify Grafana Datasource

Command:

```bash
curl -s -u admin:grafana2026 \
  http://localhost:3000/api/datasources | \
  jq '.[] | {name,type,url}'
```

Result:

```json
{
  "name": "Prometheus",
  "type": "prometheus",
  "url": "http://prometheus:9090"
}
```

Grafana was now correctly connected to Prometheus.

---

# 9. Create Grafana Monitoring Dashboard

I created a dashboard named:

```text
Monitoring Overview
```

The dashboard contains three panels:

1. Request Rate
2. P95 Inference Latency
3. Prediction Accuracy

---

## Panel 1 — Request Rate

PromQL:

```promql
sum(rate(flask_http_request_total[1m]))
```

This displays the application request rate.

---

## Panel 2 — P95 Inference Latency

PromQL:

```promql
histogram_quantile(
  0.95,
  sum(rate(model_inference_duration_seconds_bucket[5m])) by (le)
)
```

This displays the 95th percentile inference latency.

---

## Panel 3 — Prediction Accuracy

PromQL:

```promql
prediction_accuracy
```

This displays the current prediction accuracy.

---

# 10. Dashboard Tags

The dashboard was tagged with:

```text
mlops
monitoring
```

This makes it easier for the operations team to find the dashboard through Grafana search.

---

# 11. Dashboard Verification

The dashboard was successfully created through the Grafana API.

Dashboard UID:

```text
axcs2f
```

Verification command:

```bash
curl -s -u admin:grafana2026 \
  http://localhost:3000/api/dashboards/uid/axcs2f |
  jq '{
    title: .dashboard.title,
    tags: .dashboard.tags,
    panels: (.dashboard.panels | length)
  }'
```

Final result:

```json
{
  "title": "Monitoring Overview",
  "tags": [
    "mlops",
    "monitoring"
  ],
  "panels": 3
}
```

---

# 12. Final Architecture

The completed monitoring flow:

```text
Evidently Drift Scorer
        │
        │ drift_scores.json
        ▼
Metric Emitter
        │
        │ HTTP :5000/metrics
        ▼
Prometheus
        │
        │ PromQL
        ▼
Grafana
        │
        ▼
Monitoring Overview
```

---

# 13. Final Validation Checklist

| Requirement | Status |
|---|---|
| Flask `/metrics` returns HTTP 200 | ✅ |
| Prometheus scrapes metric emitter | ✅ |
| Prometheus target health is `up` | ✅ |
| Correct Prometheus scrape port | ✅ |
| Grafana datasource exists | ✅ |
| Grafana points to Prometheus `:9090` | ✅ |
| Monitoring Overview dashboard created | ✅ |
| Dashboard has 3 panels | ✅ |
| Dashboard has tags | ✅ |
| Evidently scorer left untouched | ✅ |
| Evidently continues producing drift scores | ✅ |

---

# Key Lessons

## 1. Container Ports and Service Ports Matter

Inside a Docker Compose network, services should communicate using the service name and container port:

```text
metric-emitter:5000
prometheus:9090
```

`localhost` inside a container refers to that container itself, not another Compose service.

---

## 2. Verify the Actual Application Route

A running container does not guarantee that the expected HTTP endpoint exists.

The Flask application exposed:

```text
/prom-metrics
```

while Prometheus expected:

```text
/metrics
```

The endpoint mismatch caused the initial `404`.

---

## 3. Follow the Failure Chain

The monitoring failure propagated downstream:

```text
Wrong Flask endpoint
        ↓
Prometheus cannot scrape
        ↓
Prometheus target DOWN
        ↓
No usable Prometheus samples
        ↓
Grafana panels empty
```

Each configuration issue was independent, but together they broke the complete monitoring pipeline.

---

# Conclusion

The end-to-end fraud-model monitoring stack is now operational:

```text
Evidently → Metric Emitter → Prometheus → Grafana
```

The final Grafana dashboard provides visibility into:

- Request rate
- P95 inference latency
- Prediction accuracy

Additional Evidently-derived metrics such as:

```text
data_drift_score
evidently_drift_share
```

are also available through Prometheus.

### Screenshots
