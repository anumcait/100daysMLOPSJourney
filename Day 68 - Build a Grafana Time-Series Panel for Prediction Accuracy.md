# Day 68: Build a Grafana Time-Series Panel for Prediction Accuracy
The xFusionCorp Industries ML platform team monitors the rolling prediction accuracy of the fraud-detection model using a Grafana dashboard. The monitoring stack is operational, with the Flask metric-emitter exposing prediction_accuracy as a gauge. Prometheus is configured to scrape this metric, and the Prometheus datasource is already pre-provisioned in Grafana. Your objective is to create a Grafana dashboard featuring a time-series panel that visualizes prediction_accuracy over time.


The Grafana UI is running on port 3000. The Grafana button opens the login page. Admin credentials: admin / grafana2026. The Prometheus datasource is already wired (pre-provisioned at startup—no datasource form to fill).

Available metrics on the Prometheus datasource include:

prediction_accuracy – The gauge for this task.
flask_http_request_total{version, endpoint, method} – Counter.
data_drift_score{column} – Per-column drift gauge.
model_inference_duration_seconds – Latency histogram.
The end state must include:

Grafana's Data sources list shows a provisioned Prometheus datasource (pre-staged).
GET /api/search?type=dash-db returns at least one user-created dashboard.
That dashboard carries at least one panel whose type is timeseries.
At least one panel across your dashboards has a Prometheus query that references prediction_accuracy.

## Objective

Create a Grafana dashboard that visualizes the `prediction_accuracy` Prometheus metric using a **Time series** panel.

---

## Environment

- **Grafana URL:** `http://<SERVER-IP>:3000`
- **Username:** `admin`
- **Password:** `grafana2026`

The Prometheus datasource is already provisioned.

---

## Available Metrics

- `prediction_accuracy` *(Gauge)*
- `flask_http_request_total{version, endpoint, method}` *(Counter)*
- `data_drift_score{column}` *(Gauge)*
- `model_inference_duration_seconds` *(Histogram)*

---

## Steps

### 1. Login to Grafana

Open Grafana:

```
http://<SERVER-IP>:3000
```

Login with:

```
Username: admin
Password: grafana2026
```

---

### 2. Verify Prometheus Datasource

Navigate to:

```
Connections → Data sources
```

Verify that a **Prometheus** datasource already exists.

---

### 3. Create a Dashboard

Go to:

```
Dashboards → New → New Dashboard
```

Click:

```
Add visualization
```

Select the existing **Prometheus** datasource.

---

### 4. Create the Time Series Panel

Ensure the visualization type is:

```
Time series
```

Use the following Prometheus query:

```promql
prediction_accuracy
```

(Optional) Panel title:

```
Prediction Accuracy
```

Click **Apply**.

---

### 5. Save the Dashboard

Click **Save dashboard**.

Dashboard name:

```
ML Monitoring
```

Click **Save**.

---

## Verification

List all dashboards:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/search?type=dash-db
```

Expected output:

```json
[
  {
    "title": "ML Monitoring",
    "type": "dash-db",
    "uid": "xxxxxxxx"
  }
]
```

Retrieve the dashboard:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/dashboards/uid/<UID>
```

Verify the dashboard JSON contains:

```json
"type": "timeseries"
```

and

```json
"expr": "prediction_accuracy"
```

---

## Validation Checklist

- ✅ Prometheus datasource is provisioned.
- ✅ At least one user-created dashboard exists.
- ✅ Dashboard contains a **Time series** panel.
- ✅ The panel queries:

```promql
prediction_accuracy
```

---

## Result

The Grafana dashboard now displays the rolling prediction accuracy of the fraud-detection model using a **Time series** visualization backed by the pre-configured Prometheus datasource.

### Screenshots
