# Day 100: Capstone (4/4) — Close the Loop with Prometheus + Grafana Observability
The xFusionCorp Industries MLOps team is closing the loop on the fraud-detector with a single observability pane: oncall needs to see request rate and latency at a glance, and be alerted when the service misbehaves. The service is already instrumented, Prometheus is already scraping it, and synthetic traffic is being generated in the background. Grafana was provisioned empty — no data source, no dashboards, no alerts. Your task is to wire Grafana to Prometheus, author a multi-panel fraud-monitor dashboard, and add an alert rule on the service.


The Grafana (port 3000, log in with admin / admin) and Prometheus (port 9090) buttons at the top of the lab open the relevant UIs. The pre-staged state:

The FastAPI app on :8085 exposes Prometheus metrics at /metrics: http_requests_total (request counter) and http_request_duration_seconds (latency histogram).
Prometheus on :9090 scrapes the app every 5 s as job fraud-detector.
A background traffic generator hits /predict and /health continuously, so non-zero samples are already flowing.
Grafana on :3000 has no data source, no dashboards, and no alert rules. It reaches Prometheus at http://prometheus:9090.
The end state must include:

Grafana has a data source of type prometheus pointed at http://prometheus:9090.
Grafana has a dashboard named fraud-monitor with at least two panels — one querying http_requests_total, one querying http_request_duration_seconds.
Grafana has at least one alert rule whose query references a service metric.
The Compose stack lives under /root/code/observability/ (app/, compose.yaml, prometheus.yml, scripts/) for transparency; nothing under that directory needs to be edited. Observability is two halves — a dashboard answers 'what is happening?' and an alert answers 'when should a human care?'.


## Objective

Wire Grafana to the existing Prometheus instance, create a fraud-detector observability dashboard, and configure an alert rule for the service.

## Pre-staged Environment

- FastAPI fraud-detector service runs on port `8085`.
- Prometheus runs on port `9090`.
- Grafana runs on port `3000`.
- Grafana credentials:
  - Username: `admin`
  - Password: `admin`
- Prometheus scrapes the FastAPI application every 5 seconds.
- Prometheus job name: `fraud-detector`.
- Service metrics:
  - `http_requests_total`
  - `http_request_duration_seconds`
- Synthetic traffic continuously calls `/predict` and `/health`.
- Prometheus is reachable by Grafana at:

```text
http://prometheus:9090
```

## 1. Configure Prometheus Data Source

In Grafana:

**Connections → Data sources → Add data source → Prometheus**

Configure:

```text
Name: Prometheus
URL: http://prometheus:9090
```

Click **Save & test** and verify that the connection succeeds.

## 2. Create the Fraud Monitor Dashboard

Create a new dashboard from:

**Dashboards → New → New dashboard**

The dashboard must be named exactly:

```text
fraud-monitor
```

### Panel 1 — Request Rate

Use the Prometheus query:

```promql
sum(rate(http_requests_total{job="fraud-detector"}[1m]))
```

Panel title:

```text
Request rate
```

Use a time-series visualization.

### Panel 2 — Latency

Use the Prometheus histogram to calculate P95 request latency:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket{job="fraud-detector"}[5m])) by (le)
)
```

Panel title:

```text
Latency
```

Use a time-series visualization.

Save the dashboard with the exact title:

```text
fraud-monitor
```

The dashboard must contain at least:

- A request-rate panel querying `http_requests_total`.
- A latency panel querying `http_request_duration_seconds`.

## 3. Create an Alert Rule

Go to:

**Alerting → Alert rules → New alert rule**

Example alert name:

```text
fraud-detector-traffic
```

Use the service request metric:

```promql
sum(rate(http_requests_total{job="fraud-detector"}[1m]))
```

Configure the alert condition:

```text
IS BELOW 0.01
```

Set a pending period such as:

```text
1m
```

## 4. Configure a Contact Point

Grafana requires a contact point before an alert rule can be saved.

Go to:

**Alerting → Notification configuration → Contact points → Create contact point**

Configure:

```text
Name: lab-contact
Integration: Alertmanager
URL: http://localhost:9093
```

Save the contact point.

Return to the alert rule and select:

```text
lab-contact
```

as the contact point.

Save the alert rule.

## 5. Final Verification

The completed Grafana configuration should contain:

```text
Data source
└── Prometheus
    └── http://prometheus:9090

Dashboard
└── fraud-monitor
    ├── Request rate
    │   └── http_requests_total
    └── Latency
        └── http_request_duration_seconds_bucket

Alert rule
└── fraud-detector-traffic
    └── http_requests_total
```

## Result

Grafana now provides a single observability pane for the fraud-detector service.

- **Request rate** shows current service traffic.
- **Latency** shows P95 request latency.
- **Alerting** identifies when service traffic falls below the configured threshold.

No files under `/root/code/observability/` need to be modified.
