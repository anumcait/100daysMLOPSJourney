# Day 23: Search and Query MLflow Runs

## Task Description
The xFusionCorp Industries team needs to filter and flag experimental runs in the `fraud-detection` experiment based on their performance. Specifically, the goal is to programmatically identify runs with high and low performance and label them for review.

### Requirements:
1. Connect to the MLflow Tracking Server at `http://localhost:5000`.
2. Access the `fraud-detection` experiment.
3. Search for all runs within this experiment.
4. Apply the following logic to tag runs:
   - If `f1_score` is exactly `0.95`, set a tag `review-status: shortlisted`.
   - If `f1_score` is less than `0.75`, set a tag `review-status: rejected`.
5. Verify the tags have been successfully applied to the correct runs.

## Solution

### Run this Python script to update the tags:

```python
import mlflow
from mlflow import MlflowClient

# Initialize the client to connect to the tracking server
mlflow.set_tracking_uri("http://localhost:5000")
client = MlflowClient()

# Get the experiment by name
exp = client.get_experiment_by_name("fraud-detection")
runs = client.search_runs([exp.experiment_id])

# Update tags based on f1_score logic
for r in runs:
    f1 = r.data.metrics.get("f1_score")

    if f1 == 0.95:
        client.set_tag(r.info.run_id, "review-status", "shortlisted")
        print(f"Run {r.info.run_id} (f1={f1}) -> shortlisted")

    elif f1 is not None and f1 < 0.75:
        client.set_tag(r.info.run_id, "review-status", "rejected")
        print(f"Run {r.info.run_id} (f1={f1}) -> rejected")

print("Done tagging runs.")
```

### To verify:

```python
import mlflow
from mlflow import MlflowClient

mlflow.set_tracking_uri("http://localhost:5000")
client = MlflowClient()

exp = client.get_experiment_by_name("fraud-detection")

print("F1 Score | Review Status")
print("-" * 25)
for r in client.search_runs([exp.experiment_id]):
    f1 = r.data.metrics.get("f1_score")
    status = r.data.tags.get("review-status")
    print(f"{f1:<8} | {status}")
```

### You should see output similar to:

```text
0.95     | shortlisted
0.72     | rejected
0.70     | rejected
...      | None
```

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/a4588a5b-7464-483b-9997-f0194774ce74" />


