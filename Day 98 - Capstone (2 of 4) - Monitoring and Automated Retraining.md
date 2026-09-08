# Day 98: Capstone (2/4): Monitoring and Automated Retraining

## Objective

Build an automated MLOps retraining loop for the production `fraud-detector` model.

The loop should:

1. Detect data drift using the provided `drift.py`.
2. Retrain **only when drift is detected**.
3. Retrain using the combined data through the provided `retrain.py`.
4. Register the retrained model as a new `fraud-detector` version.
5. Move the `production` alias to the newly registered version.
6. Confirm that version 2 is live.

---

## Pre-staged Environment

The lab provides:

- MLflow tracking server on port `5000`.
- SeaweedFS artifacts at port `8333`.
- SeaweedFS Filer at port `8888`.
- MLflow registered model:
  - Name: `fraud-detector`
  - Version: `1`
  - Alias: `production`
- Reference training data:
  - `/root/code/data/reference.csv`
- Current shifted transaction data:
  - `/root/code/data/current.csv`
- Drift report server on port `8086`.
- Drift report directory:
  - `/root/code/reports/`
- Provided scripts:
  - `/root/code/drift.py`
  - `/root/code/retrain.py`
- Scaffold:
  - `/root/code/retrain_if_drift.py`

The provided `drift.py` and `retrain.py` scripts do not need to be modified.

---

## Starting Point

The scaffold already contains the MLflow configuration, drift execution, retraining execution, and logic for finding the latest `retrain` run.

The two TODOs are:

1. Gate retraining based on the drift result.
2. Register and promote the retrained model.

---

## TODO 1: Gate Retraining on Drift

After running `drift.py`, the script loads:

```python
summary = json.loads((HERE / "reports" / "drift-summary.json").read_text())
drifted = bool(summary.get("dataset_drift"))
```

If the data has **not** drifted, the automation must stop immediately.

Add:

```python
if not drifted:
    print("[loop] no drift detected; no retraining needed")
    raise SystemExit(0)
```

This ensures that:

- No retraining occurs when there is no drift.
- No new model version is registered.
- The production alias is not changed.
- The script exits successfully.

---

## TODO 2: Register and Promote the Retrained Model

After `retrain.py` runs, the scaffold finds the latest retraining run:

```python
run_id = _latest_retrain_run_id()
```

The retrained MLflow model is stored under:

```text
runs:/<run_id>/model
```

Construct the model URI:

```python
model_uri = f"runs:/{run_id}/model"
```

Register it as a new version of `fraud-detector`:

```python
model_version = mlflow.register_model(model_uri, MODEL_NAME)
```

Then move the `production` alias to the newly created version:

```python
client.set_registered_model_alias(
    MODEL_NAME,
    ALIAS,
    model_version.version,
)
```

A useful confirmation message is:

```python
print(
    f"[loop] promoted: {MODEL_NAME} "
    f"version={model_version.version} alias={ALIAS}"
)
```

---

## Completed TODO Sections

The two TODO sections should look like this:

```python
# TODO 1: Gate retraining on drift. If the model has NOT drifted, print
# that no retraining is needed and exit 0 -- do not retrain or promote.
# (Only reach the steps below when drifted is True.)
if not drifted:
    print("[loop] no drift detected; no retraining needed")
    raise SystemExit(0)


# --- 2. Retrain on the combined data (logs a `retrain` run to MLflow).
_run("retrain.py")
run_id = _latest_retrain_run_id()
print(f"[loop] retrained: run_id={run_id}")

# TODO 2: Promote the retrained model automatically. Register
# runs:/<run_id>/model as a new version of MODEL_NAME, then move the
# ALIAS (`production`) onto that new version so the serving layer picks
# it up -- use mlflow.register_model(...) and
# client.set_registered_model_alias(...).
model_uri = f"runs:/{run_id}/model"
model_version = mlflow.register_model(model_uri, MODEL_NAME)

client.set_registered_model_alias(
    MODEL_NAME,
    ALIAS,
    model_version.version,
)

print(
    f"[loop] promoted: {MODEL_NAME} "
    f"version={model_version.version} alias={ALIAS}"
)
```

---

## Full `retrain_if_drift.py`

```python
"""Drift-triggered retraining for fraud-detector.

One command closes the loop: detect drift -> (only if drifted) retrain
on the combined data -> register the new run as a `fraud-detector`
version -> move the `production` alias to it. This is the "automated
retraining" the lab is about -- no manual clicking in the MLflow UI.

The plumbing to run the provided drift/retrain scripts and to find the
new run id is written for you. Author the two TODOs.
"""
from __future__ import annotations

import json
import subprocess
import sys
from pathlib import Path

import mlflow

HERE = Path(__file__).resolve().parent
TRACKING_URI = "http://localhost:5000"
MODEL_NAME = "fraud-detector"
ALIAS = "production"

mlflow.set_tracking_uri(TRACKING_URI)
client = mlflow.tracking.MlflowClient()


def _run(script: str) -> None:
    """Run one of the provided scripts (drift.py / retrain.py)."""
    subprocess.check_call([sys.executable, str(HERE / script)])


def _latest_retrain_run_id() -> str:
    exp = client.get_experiment_by_name("fraud-detection")
    runs = client.search_runs(
        [exp.experiment_id],
        filter_string="tags.mlflow.runName = 'retrain'",
        order_by=["attributes.start_time DESC"],
        max_results=1,
    )
    if not runs:
        raise SystemExit("no `retrain` run found after retraining")
    return runs[0].info.run_id


# --- 1. Detect drift (writes reports/drift-summary.json + drift.html).
_run("drift.py")
summary = json.loads((HERE / "reports" / "drift-summary.json").read_text())
drifted = bool(summary.get("dataset_drift"))
print(f"[loop] dataset_drift={drifted}")

# Gate retraining on drift.
if not drifted:
    print("[loop] no drift detected; no retraining needed")
    raise SystemExit(0)


# --- 2. Retrain on the combined data (logs a `retrain` run to MLflow).
_run("retrain.py")
run_id = _latest_retrain_run_id()
print(f"[loop] retrained: run_id={run_id}")

# Register the retrained model as a new version.
model_uri = f"runs:/{run_id}/model"
model_version = mlflow.register_model(model_uri, MODEL_NAME)

# Move the production alias to the newly registered version.
client.set_registered_model_alias(
    MODEL_NAME,
    ALIAS,
    model_version.version,
)

print(
    f"[loop] promoted: {MODEL_NAME} "
    f"version={model_version.version} alias={ALIAS}"
)
```

---

## Run the Automated Loop

Execute:

```bash
cd /root/code
python3 retrain_if_drift.py
```

Because `current.csv` contains shifted data, the expected result is:

```text
[loop] dataset_drift=True
[loop] retrained: run_id=<run-id>
[loop] promoted: fraud-detector version=2 alias=production
```

The exact run ID will depend on the MLflow environment.

---

## Verify the Drift Report

Check that the report exists:

```bash
test -f /root/code/reports/drift.html && echo "drift.html OK"
```

Check the summary:

```bash
cat /root/code/reports/drift-summary.json
```

The summary must contain:

```json
"dataset_drift": true
```

The generated `drift.html` can also be viewed through the lab's **Drift Report** button on port `8086`.

---

## Verify the Retraining Run

Use MLflow to confirm that the `fraud-detection` experiment contains a run named `retrain`:

```bash
python3 - <<'PY'
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")

exp = mlflow.get_experiment_by_name("fraud-detection")

runs = mlflow.search_runs(
    experiment_ids=[exp.experiment_id],
    filter_string="tags.mlflow.runName = 'retrain'"
)

print(runs[["run_id", "tags.mlflow.runName"]].to_string(index=False))
PY
```

A run named `retrain` should be present.

---

## Verify the Registered Model

Check all versions of `fraud-detector`:

```bash
python3 - <<'PY'
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")
client = mlflow.tracking.MlflowClient()

versions = client.search_model_versions("name='fraud-detector'")

for v in versions:
    print(
        f"version={v.version}, "
        f"run_id={v.run_id}, "
        f"aliases={v.aliases}"
    )
PY
```

The output should show at least version 2.

Version 2 should be associated with the retraining run.

---

## Verify the Production Alias

Run:

```bash
python3 - <<'PY'
import mlflow

mlflow.set_tracking_uri("http://localhost:5000")
client = mlflow.tracking.MlflowClient()

v = client.get_model_version_by_alias(
    "fraud-detector",
    "production",
)

print("production version:", v.version)
print("run_id:", v.run_id)
PY
```

Expected result:

```text
production version: 2
run_id: <retrain-run-id>
```

The important condition is:

```text
production -> version 2
```

or a higher version if the automation has been run multiple times.

Version 1 must no longer have the `production` alias.

---

## Final State

The lab is complete when all of the following are true:

- `/root/code/reports/drift.html` exists.
- `/root/code/reports/drift-summary.json` reports `dataset_drift=True`.
- The `fraud-detection` MLflow experiment contains a run named `retrain`.
- `fraud-detector` has version 2 or higher.
- The new version is sourced from the retraining run.
- The `production` alias points to version 2 or higher.
- Version 1 is no longer the production version.
- `retrain_if_drift.py` performs registration and promotion automatically.

---

## MLOps Concept

This exercise demonstrates a complete drift-triggered retraining loop:

```text
Production Data
      |
      v
Drift Detection
      |
      v
dataset_drift?
   /       \
 No         Yes
 |           |
 v           v
Stop       Retrain
             |
             v
       Register Version
             |
             v
     Move production Alias
             |
             v
        New Model Live
```

The key principle is that retraining is **conditional**.

If the data has not drifted, the system should do nothing.

If the data has drifted, the system automatically:

1. Detects the shift.
2. Retrains the model.
3. Registers a new model version.
4. Promotes that version.
5. Updates the production serving alias.

This turns model maintenance from a manual process into an automated production MLOps feedback loop.

### Screenshots
