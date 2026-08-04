# Day 71: Build a 4-Panel Model-Overview Grafana Dashboard
The xFusionCorp Industries ML platform team requires the creation of a model-overview dashboard in Grafana. This dashboard should display the following metrics side-by-side: request rate, p95 inference latency, current prediction accuracy, and Evidently-computed per-feature drift. This setup will facilitate a quick assessment of the model's health by the on-call engineer.

The existing monitoring stack includes the following components: an Evidently drift scorer that recalculates per-feature drift (PSI) against a reference window every 15 seconds; a Flask metric-emitter that republishes these scores alongside its serving signals; Prometheus, which is actively scraping data; and Grafana, which has the Prometheus datasource pre-provisioned.

Your task is to construct a 4-panel Grafana dashboard that integrates multiple visualization types effectively.


The Grafana UI is running on port 3000. The Grafana button opens the login page. Admin credentials: admin / grafana2026. The Prometheus datasource is pre-provisioned.

The dashboard must contain one panel for each of the four signals below, and mix at least three distinct visualization types across them:

Request rate – built from the flask_http_request_total counter (labels version, endpoint, method).
p95 inference latency – built from the model_inference_duration_seconds histogram (its _bucket series carry the le label).
Prediction accuracy – the prediction_accuracy gauge.
Drift by column – the data_drift_score gauge, one series per column label. This is the Evidently signal: each value is the PSI the drift scorer computed for that feature in the latest window.
The same drift data is also visible from Evidently's side for cross-checking: the Evidently UI button (port 8000) opens the fraud-detector drift monitoring project, whose Dashboard tab plots the drifted-columns share and per-column PSI over time (a new scoring run lands roughly every minute) and whose Reports tab lists the underlying runs. Nothing to configure there—it's the same drift data, seen from Evidently's side.

The end state must include:

GET /api/search?type=dash-db returns at least one user-created dashboard.
The dashboard carries 4 or more non-row panels.
The panel targets collectively reference flask_http_request_total, model_inference_duration_seconds_bucket, prediction_accuracy, and data_drift_score.
The panels use at least 3 distinct visualization types (e.g. timeseries, stat, bargauge).
The Evidently UI's project keeps accumulating scoring runs (pre-wired—nothing to change).

## Objective

Created a Grafana dashboard that provides a quick overview of the ML model's health by displaying four key monitoring metrics.

## Dashboard Name

**Model Overview**

## Panels

### 1. Request Rate
- **Visualization:** Time Series
- **Metric:** `flask_http_request_total`
- **Query:**
```promql
sum(rate(flask_http_request_total[5m]))
```

---

### 2. p95 Inference Latency
- **Visualization:** Stat
- **Metric:** `model_inference_duration_seconds_bucket`
- **Query:**
```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(model_inference_duration_seconds_bucket[5m])
  )
)
```

---

### 3. Prediction Accuracy
- **Visualization:** Gauge
- **Metric:** `prediction_accuracy`
- **Query:**
```promql
prediction_accuracy
```

---

### 4. Drift by Column
- **Visualization:** Bar Gauge
- **Metric:** `data_drift_score`
- **Query:**
```promql
data_drift_score
```

---

## Visualization Types Used

- Time Series
- Stat
- Gauge
- Bar Gauge

This satisfies the requirement of using at least three different visualization types.

## Outcome

Successfully created a Grafana dashboard containing:

- ✅ Request Rate
- ✅ p95 Inference Latency
- ✅ Prediction Accuracy
- ✅ Per-feature Drift (PSI)

The dashboard references all required Prometheus metrics:

- `flask_http_request_total`
- `model_inference_duration_seconds_bucket`
- `prediction_accuracy`
- `data_drift_score`

The dashboard is saved and available through the Grafana dashboard search API (`/api/search?type=dash-db`).

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/51198b98-4a8d-44db-88cd-6930f8819c9b" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/13c76664-1974-42d6-99be-4c8b71ea0a7b" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/250fef13-2da1-4fce-8c05-0a5a0192885b" />



