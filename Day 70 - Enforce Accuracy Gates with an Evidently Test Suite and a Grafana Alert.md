# Day 70: Enforce Accuracy Gates with Evidently Test Suite and Grafana Alert
The xFusionCorp Industries ML platform team requires the enforcement of quality gates for the fraud-detection model in two key ways: first, by establishing an Evidently test suite that can be executed by any CI job against a production batch, and second, by integrating this suite with Grafana to ensure that on-call personnel are notified immediately if live accuracy declines. The monitoring stack is operational, and the test-suite scaffold is already established, including data loading, classification mapping, report execution, and publication to the Evidently UI.

Your task consists of two parts: (1) Complete the TODO block of the scaffold with the two specified threshold metrics, execute the test suite, and examine the results in the Evidently UI; (2) Create a Grafana alert rule that triggers when avg_over_time(prediction_accuracy[1m]) falls below 0.80.


Evidently test suite. /root/code/monitoring/tests/test_suite.py is pre-wired except for the gates themselves; a TODO block marks where two thresholded metrics must be appended to METRICS:

a missing-values gate that fails the suite when the batch carries 10 or more missing values.

an accuracy gate that fails the suite when batch accuracy is 0.80 or lower.

The batch it runs against is /root/code/monitoring/tests/current.csv (features + is_fraud target + the model's prediction column). The batch carries only a few missing values and its accuracy clears 0.80, so both gates should end up SUCCESS. A successful run writes test_results.json and publishes a snapshot to the Evidently workspace, viewable under the Evidently UI button (port 8000) in the fraud-detector quality gates project (Reports tab).

Grafana alert rule. The Grafana UI is running on port 3000. The Grafana button opens the login page. Admin credentials: admin / grafana2026. The Prometheus datasource is pre-provisioned. Metrics available:

prediction_accuracy – The gauge for this task. It drifts in a random walk around 0.85, so avg_over_time(prediction_accuracy[1m]) is the smoothed signal the alert should watch.
data_drift_score{column}, evidently_drift_share – Per-feature PSI and the drifted-columns share, computed by the Evidently drift scorer at /root/code/monitoring/drift/drift_scorer.py.
flask_http_request_total{version, endpoint, method}, model_inference_duration_seconds – The other signals from the shared metric-emitter.
The alert rule must fire when avg_over_time(prediction_accuracy[1m]) drops below 0.80.

The end state must include:

/root/code/monitoring/tests/test_results.json exists and carries at least two Evidently test entries—a missing-values gate and an accuracy gate—all with status SUCCESS.
The Evidently UI's project carries at least one published run (snapshot).
GET /api/v1/provisioning/alert-rules returns a non-empty array.
At least one rule's PromQL expression references prediction_accuracy.
That rule's threshold evaluator carries 0.80 as a numeric parameter.
The same 0.80 accuracy gate is enforced at two altitudes: the Evidently test suite fails a CI pipeline before a degraded model ships, and the Grafana alert rule pages on-call after live accuracy slips. Evidently's include_tests=True turns each metric into a pass/fail assertion—the same structure a pytest run gives you, but over data and model quality—and the Evidently UI is where a reviewer reads those verdicts without touching code.


## Objective

The xFusionCorp Industries ML platform team requires quality gates for the fraud-detection model in two ways:

1. **Evidently test suite**
   - Validate production batch quality before deployment.
   - Fail CI if data quality or model accuracy falls below the required threshold.

2. **Grafana alerting**
   - Monitor live model accuracy.
   - Notify on-call engineers when accuracy drops below the acceptable limit.

The accuracy threshold is enforced at two levels:

- **CI/CD level:** Evidently test suite
- **Production monitoring level:** Grafana alert rule

---

# Part 1: Evidently Accuracy and Data Quality Gates

## File Location

```
/root/code/monitoring/tests/test_suite.py
```

The scaffold was already configured with:

- CSV loading
- Dataset definition
- Classification mapping
- Evidently report execution
- Workspace publishing

Only the metric gates needed to be added.

---

## Required Gates

### 1. Missing Values Gate

Requirement:

- Fail when the batch contains **10 or more missing values**
- Pass when missing values are below 10

Implementation:

```python
DatasetMissingValueCount(
    tests=[lt(10)]
)
```

---

### 2. Accuracy Gate

Requirement:

- Fail when accuracy is **0.80 or lower**
- Pass when accuracy is greater than 0.80

Implementation:

```python
Accuracy(
    tests=[gt(0.80)]
)
```

---

## Final METRICS Configuration

```python
METRICS = []

METRICS.append(
    DatasetMissingValueCount(
        tests=[lt(10)]
    )
)

METRICS.append(
    Accuracy(
        tests=[gt(0.80)]
    )
)
```

---

## Execute Evidently Test Suite

Run:

```bash
python3 /root/code/monitoring/tests/test_suite.py
```

Expected output:

```text
Tests: 2/2 passed -> /root/code/monitoring/tests/test_results.json
  - DatasetMissingValueCount: SUCCESS
  - Accuracy: SUCCESS
Run published -- refresh the Evidently UI to inspect it.
```

---

## Validation

Generated file:

```
/root/code/monitoring/tests/test_results.json
```

The file contains Evidently test results:

- Missing value gate → SUCCESS
- Accuracy gate → SUCCESS

The report is published to the Evidently workspace.

---

## Evidently UI Verification

Open:

```
http://<host>:8000
```

Navigate:

```
Project:
fraud-detector quality gates

Reports tab
```

Verify that a published snapshot exists.

---

# Part 2: Grafana Accuracy Alert

## Grafana Details

URL:

```
http://<host>:3000
```

Credentials:

```
Username: admin
Password: grafana2026
```

---

# Create Alert Rule

Navigate:

```
Alerting
   └── Alert rules
          └── New alert rule
```

---

## Alert Name

```
Prediction Accuracy Alert
```

---

## Prometheus Query

Datasource:

```
Prometheus
```

Query:

```promql
avg_over_time(prediction_accuracy[1m])
```

This creates a smoothed accuracy signal.

---

## Alert Condition

Configure:

```
WHEN QUERY

Is below

0.80
```

The alert fires when:

```promql
avg_over_time(prediction_accuracy[1m]) < 0.80
```

---

## Folder

Created Grafana folder:

```
ml-alerts
```

---

## Evaluation

Configured:

```
Evaluation interval: 1m
Pending period: None
```

---

## Save Alert Rule

After saving, verify using:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/v1/provisioning/alert-rules
```

Expected:

- Non-empty alert rule array
- Rule contains:

```
prediction_accuracy
```

- Threshold:

```
0.80
```

---

# Final Validation Checklist

## Evidently

✅ `test_results.json` exists

✅ Contains two tests:

```
DatasetMissingValueCount → SUCCESS

Accuracy → SUCCESS
```

✅ Evidently workspace contains published snapshot

---

## Grafana

✅ Alert rule exists

✅ PromQL expression:

```promql
avg_over_time(prediction_accuracy[1m])
```

✅ Threshold:

```
0.80
```

✅ Alert fires when live accuracy drops below 80%

---

# Conclusion

The fraud-detection model quality gate is now enforced at two levels:

## CI/CD Protection

Evidently prevents degraded models from being promoted by failing automated quality checks.

## Production Monitoring

Grafana continuously monitors live accuracy and alerts the on-call team when model performance drops.

The same accuracy threshold (0.80) is enforced before deployment and during production operation.

### Screenshots
