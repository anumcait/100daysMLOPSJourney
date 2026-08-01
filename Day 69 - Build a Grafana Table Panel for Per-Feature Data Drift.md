# Day 69: Build a Grafana Table Panel for Per-Feature Data Drift
The xFusionCorp Industries ML platform team monitors per-feature data drift for the fraud-detection model using a Grafana table. This table presents one row for each feature and one column for each drift score, allowing reviewers to easily scan the entire feature set. The monitoring stack is operational; the Flask metric-emitter exposes data_drift_score as a labeled gauge (with one time-series per feature), Prometheus is actively scraping the data, and Grafana has the Prometheus datasource pre-provisioned. Your objective is to create a Grafana dashboard featuring a Table panel that displays the data_drift_score values for each column.


The Grafana UI is running on port 3000. The Grafana button opens the login page. Admin credentials: admin / grafana2026. The Prometheus datasource is pre-provisioned.

Metrics available on the Prometheus datasource:

data_drift_score{column="amount"}, data_drift_score{column="hour"}, data_drift_score{column="num_tx_past_day"} – The gauge for this task, one series per feature column.
prediction_accuracy, flask_http_request_total{version, endpoint, method}, model_inference_duration_seconds – The other signals from the shared metric-emitter.
The end state must include:

GET /api/search?type=dash-db returns at least one user-created dashboard.
At least one panel across your dashboards has type: table.
At least one of those panel's Prometheus targets references data_drift_score.
Querying data_drift_score through Grafana's datasource proxy returns non-empty Prometheus series whose labels include column – Confirming the table has per-feature rows to render.
A per-feature drift view answers 'which input shifted?' at a glance — a Table panel is the natural shape for it, one row per feature carrying that feature's latest drift score.

## Objective

Create a Grafana dashboard containing a **Table** panel that displays the latest `data_drift_score` for each feature (column) of the fraud detection model.

## Environment

- **Grafana URL:** `http://<host>:3000`
- **Username:** `admin`
- **Password:** `grafana2026`
- **Datasource:** Prometheus (pre-configured)

## Available Metrics

- `data_drift_score{column="amount"}`
- `data_drift_score{column="hour"}`
- `data_drift_score{column="num_tx_past_day"}`
- `prediction_accuracy`
- `flask_http_request_total`
- `model_inference_duration_seconds`

## Steps

### 1. Log in to Grafana

Open Grafana and log in using:

- Username: `admin`
- Password: `grafana2026`

---

### 2. Create a Dashboard

1. Navigate to **Dashboards**.
2. Click **New Dashboard**.
3. Select **Add Visualization**.
4. Choose the **Prometheus** datasource.

---

### 3. Configure the Query

Use the following Prometheus metric:

```prometheus
data_drift_score
```

Change the query settings:

- **Type:** `Instant`
- **Format:** `Table` (or keep **Time series** and use a transformation if required)

> **Note:** Using an **Instant** query ensures only the latest value for each feature is displayed instead of historical timestamps.

---

### 4. Configure the Visualization

Set the visualization type to:

- **Table**

If Grafana does not automatically display one row per feature:

1. Open the **Transformations** tab.
2. Click **Add transformation**.
3. Select **Labels to fields**.
   - If unavailable, use **Series to rows**.

This converts the Prometheus label (`column`) into a table column.

---

### 5. Save the Dashboard

Rename the panel, for example:

```
Feature Data Drift
```

Click **Apply**, then **Save Dashboard**.

---

## Expected Output

| column | Value |
|---------|------:|
| amount | 0.391 |
| hour | 0.xxx |
| num_tx_past_day | 0.xxx |

Each row represents one feature, and the **Value** column shows its latest drift score.

---

## Validation

The task is considered complete when:

- ✅ `GET /api/search?type=dash-db` returns at least one user-created dashboard.
- ✅ At least one panel is of type **Table**.
- ✅ The panel queries **`data_drift_score`**.
- ✅ The Prometheus query returns multiple series with the `column` label.
- ✅ The table displays one row per feature with its latest drift score.

## Outcome

A Grafana dashboard now provides a concise per-feature data drift view, making it easy to identify which input features have drifted at a glance.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6eba061d-b5d1-416c-beb7-ad94f9fe44ba" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/bdf93f85-4bff-4185-949e-e1b71285d3cd" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8d898fe4-b43f-4b31-8221-c92470b626ba" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1c0a8619-a004-4db0-9311-86f459ae6fdf" />




