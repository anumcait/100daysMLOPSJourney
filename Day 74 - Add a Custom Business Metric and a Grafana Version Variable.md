# Day 74: Add a Custom Business Metric and a Grafana Version Variable

## Objective

Add a custom Prometheus business metric to track the total fraudulent transaction amount in USD and make the metric filterable by model version in Grafana.

## Requirements

- Add `fraud_amount_usd_total` as a Prometheus `Counter`.
- Add a `version` label to the counter.
- Increment the counter inside the existing `_nudge_metrics` loop.
- Verify Prometheus exposes non-empty samples for each model version.
- Create a Grafana dashboard variable named `version`.
- Source the variable from Prometheus using `label_values(...)`.
- Add a Grafana panel that queries `fraud_amount_usd_total` and filters using `$version`.

---

## 1. Update the Metric Emitter

The metric emitter is located at:

`/root/code/monitoring/app/metric_emitter.py`

A new Prometheus counter was added:

    FRAUD_AMOUNT_USD_TOTAL = Counter(
        "fraud_amount_usd_total",
        "Total fraudulent transaction amount in USD, labelled by model version.",
        labelnames=["version"],
        registry=REGISTRY,
    )

Inside `_nudge_metrics()`, the counter is incremented for each simulated request:

    for version in ("v1", "v1", "v1", "v2"):
        REQUEST_TOTAL.labels(
            version=version,
            endpoint="/predict",
            method="POST",
        ).inc()

        INFERENCE_LATENCY.observe(random.uniform(0.005, 0.15))

        fraud_amount = random.uniform(25.0, 500.0)
        FRAUD_AMOUNT_USD_TOTAL.labels(version=version).inc(fraud_amount)

This creates a separate cumulative series for each model version.

---

## 2. Restart the Metric Emitter

The monitoring containers were:

- `mon-grafana`
- `mon-prometheus`
- `metric-emitter`

Restart the metric emitter:

    docker restart metric-emitter

Check the container:

    docker ps

Check the logs:

    docker logs --tail 50 metric-emitter

The Flask application should start successfully without Python errors.

Prometheus should continue scraping the `/metrics` endpoint every few seconds.

---

## 3. Verify the Metric Emitter

Check the new metric directly:

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

Expected output:

    # HELP fraud_amount_usd_total Total fraudulent transaction amount in USD, labelled by model version.
    # TYPE fraud_amount_usd_total counter
    fraud_amount_usd_total{version="v1"} 18876.64438329985
    fraud_amount_usd_total{version="v2"} 6047.097498702644

The exact values will be different because the fraudulent amounts are simulated.

The important part is that both series exist:

    fraud_amount_usd_total{version="v1"}
    fraud_amount_usd_total{version="v2"}

This confirms:

- The metric exists.
- It is a Prometheus counter.
- It has a `version` label.
- Both `v1` and `v2` series contain values.

---

## 4. Verify Prometheus

Prometheus is exposed on port `9090`.

Query the metric:

    curl -s 'http://localhost:9090/api/v1/query?query=fraud_amount_usd_total'

Prometheus returned two series:

    fraud_amount_usd_total
    instance="metric-emitter:5000"
    job="metric-emitter"
    version="v1"

and:

    fraud_amount_usd_total
    instance="metric-emitter:5000"
    job="metric-emitter"
    version="v2"

This confirms that Prometheus successfully scraped and stored the new metric.

A successful response contains:

    {
      "status": "success",
      "data": {
        "resultType": "vector",
        "result": [
          {
            "metric": {
              "__name__": "fraud_amount_usd_total",
              "instance": "metric-emitter:5000",
              "job": "metric-emitter",
              "version": "v1"
            }
          },
          {
            "metric": {
              "__name__": "fraud_amount_usd_total",
              "instance": "metric-emitter:5000",
              "job": "metric-emitter",
              "version": "v2"
            }
          }
        ]
      }
    }

---

## 5. Create the Grafana Dashboard

Grafana is available on port `3000`.

Login credentials:

    Username: admin
    Password: grafana2026

A new dashboard was created:

`ML Fraud Monitoring`

---

## 6. Create the Grafana Version Variable

Open:

`Dashboard Settings → Variables → Add variable`

Configure the variable:

    Name: version
    Type: Query
    Data source: Prometheus
    Query type: Classic query

Use this query:

    label_values(fraud_amount_usd_total, version)

The variable configuration is:

    Name:
    version

    Target data source:
    Prometheus

    Query type:
    Classic query

    Query:
    label_values(fraud_amount_usd_total, version)

The preview returned:

    v1
    v2

Enable:

    Include All value

This allows the dashboard to select:

    v1
    v2
    All

Save or apply the variable.

---

## 7. Create the Fraud Amount Panel

Create a panel named:

`Fraud Amount USD`

Use the Prometheus datasource.

The panel query is:

    fraud_amount_usd_total{version=~"$version"}

A Time series visualization can be used.

The `=~` matcher allows the query to work with individual versions as well as the `All` selection.

---

## 8. Test the Grafana Variable

At the top of the dashboard, the `version` variable should appear.

### Select v1

Select:

    version = v1

The panel should display the fraudulent transaction amount for model version `v1`.

### Select v2

Select:

    version = v2

The panel should display the fraudulent transaction amount for model version `v2`.

### Select All

Select:

    version = All

The panel should display both model versions.

---

## 9. Final Validation

### Prometheus

- [x] `fraud_amount_usd_total` exists.
- [x] Metric is a Counter.
- [x] Metric has a `version` label.
- [x] `v1` has a non-empty value.
- [x] `v2` has a non-empty value.
- [x] Prometheus successfully scrapes the metric emitter.

### Grafana

- [x] Dashboard created.
- [x] Variable named `version`.
- [x] Variable uses the Prometheus datasource.
- [x] Variable query uses `label_values(...)`.
- [x] Variable returns `v1` and `v2`.
- [x] Include All option enabled.
- [x] Panel references `fraud_amount_usd_total`.
- [x] Panel uses `$version`.
- [x] Dashboard saved.

---

## Architecture

The completed monitoring flow is:

    Metric Emitter
          |
          | fraud_amount_usd_total{version="v1"}
          | fraud_amount_usd_total{version="v2"}
          v
    Prometheus
          |
          | label_values(fraud_amount_usd_total, version)
          v
    Grafana Variable
          |
          | $version
          v
    Grafana Panel
          |
          | fraud_amount_usd_total{version=~"$version"}
          v
    Fraud Amount USD Visualization

---

## Key Takeaway

Grafana template variables decouple dashboard panels from the number of model versions.

The general pattern is:

    Counter
      ↓
    Labelled Prometheus series
      ↓
    label_values(...)
      ↓
    Grafana template variable
      ↓
    $version
      ↓
    Parameterized panel query

This pattern can be reused for multi-version, multi-tenant, and other label-based ML monitoring dashboards.

---

## Useful Commands

### Check the metric emitter container

    docker ps

### Restart the metric emitter

    docker restart metric-emitter

### View metric emitter logs

    docker logs --tail 50 metric-emitter

### Check the new metric

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

### Query Prometheus

    curl -s 'http://localhost:9090/api/v1/query?query=fraud_amount_usd_total'

### Prometheus query

    fraud_amount_usd_total

### Grafana variable query

    label_values(fraud_amount_usd_total, version)

### Grafana panel query

    fraud_amount_usd_total{version=~"$version"}

---

## Result

The monitoring stack now tracks total fraudulent transaction amounts in USD by model version.

The final setup supports:

- `v1` fraud amount monitoring
- `v2` fraud amount monitoring
- All-version monitoring
- Dynamic Grafana filtering using the `version` template variable
- Prometheus-backed business metrics for ML model monitoring

### Screenshots
<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/0fa35368-64c9-44f1-8926-9bc76289e7ae" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/36a20edc-21fd-4f31-b093-8f463a21145e" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/d6010560-b5bc-4c68-921c-2327d97c65e4" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/2da49cce-d814-4414-a4b0-89af3f8cb7be" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/0f9193ff-6fab-4523-b187-564acb9de68c" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/8525d8d3-71c1-4f22-9872-f6d866323fe4" />

<img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/f28aa054-1298-4cac-a953-161a33b7dc6b" />


