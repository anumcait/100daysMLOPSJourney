# Day 71: Build a 4-Panel Model-Overview Grafana Dashboard

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
