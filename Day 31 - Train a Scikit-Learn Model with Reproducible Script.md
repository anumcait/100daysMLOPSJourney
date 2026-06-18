# Day 31 - Train a Scikit-Learn Model with Reproducible Script
The xFusionCorp Industries ML platform team maintains a config-driven training pipeline so hyperparameters can be swapped without editing Python code. A training scaffold exists at /root/code/fraud-detection/ with the trainer already in place, but the YAML config has been left in a broken state and the pipeline will not run cleanly. Your task is to correct the config so one successful training run lands on the MLflow tracking server and the trained model ends up inside the project tree.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty fraud-detection experiment already in place.

The project layout under /root/code/fraud-detection/:

data/train.csv – A pre-generated 200-row synthetic binary classification dataset (columns: amount, hour, num_tx_past_day, is_fraud).
src/models/train.py – The config-driven trainer. This file is correct and must not be modified.
configs/train_config.yaml – The project's training configuration. This file has bugs.
models/ – Where the serialised model must land.
Running python3 /root/code/fraud-detection/src/models/train.py currently either fails with an estimator or column error, or succeeds while dropping the model file outside the project tree. Open configs/train_config.yaml in the VS Code editor, identify every setting that prevents a clean run, and correct it.

The end state must include:

A successful training run printed to stdout.
Exactly one new MLflow run in the fraud-detection experiment, with the estimator's hyperparameters logged as run parameters.
A serialised model at /root/code/fraud-detection/models/model.pkl (absolute path, inside the project tree).
The trainer uses a small registry of supported estimators—RandomForestClassifier, GradientBoostingClassifier, LogisticRegression. Only these exact class names resolve.
==============================================================================================
The xFusionCorp Industries ML platform team maintains a config-driven training pipeline so hyperparameters can be swapped without editing Python code. A training scaffold exists at `/root/code/fraud-detection/` with the trainer already in place, but the YAML config has been left in a broken state and the pipeline will not run cleanly. Your task is to correct the config so one successful training run lands on the MLflow tracking server and the trained model ends up inside the project tree.

The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm — the dashboard loads with an empty `fraud-detection` experiment already in place.

**Project layout under `/root/code/fraud-detection/`:**

- `data/train.csv` — A pre-generated 200-row synthetic binary classification dataset (columns: `amount`, `hour`, `num_tx_past_day`, `is_fraud`).
- `src/models/train.py` — The config-driven trainer. This file is correct and **must not be modified**.
- `configs/train_config.yaml` — The project's training configuration. **This file has bugs.**
- `models/` — Where the serialised model must land.

## Objective
Fix the broken `configs/train_config.yaml` so that one successful training run is produced, the model is saved inside the project tree, and the run's hyperparameters are logged to the MLflow tracking server — all without touching `train.py`.

## End State Requirements
- A successful training run printed to stdout.
- Exactly one new MLflow run in the `fraud-detection` experiment, with the estimator's hyperparameters logged as run parameters.
- A serialised model at `/root/code/fraud-detection/models/model.pkl` (absolute path, inside the project tree).

## Solution

### Step 1: Inspect the broken config

```bash
cat /root/code/fraud-detection/configs/train_config.yaml
```

**Broken config (as found in the lab):**
```yaml
model:
  type: RandomForest        # ❌ unsupported name
  n_estimators: 100
  max_depth: 5
  random_state: 42
data:
  train_path: /root/code/fraud-detection/data/train.csv
  target_column: target     # ❌ wrong column name
output:
  model_path: /root/code/model.pkl   # ❌ path outside project tree
mlflow:
  tracking_uri: http://localhost:5000
  experiment_name: fraud-detection
```

**Three bugs identified:**
| # | Field | Broken Value | Correct Value |
|---|-------|-------------|---------------|
| 1 | `model.type` | `RandomForest` | `RandomForestClassifier` |
| 2 | `data.target_column` | `target` | `is_fraud` |
| 3 | `output.model_path` | `/root/code/model.pkl` | `/root/code/fraud-detection/models/model.pkl` |

### Step 2: Fix the config file

Open `configs/train_config.yaml` in the VS Code editor and replace its contents with the corrected version:

```yaml
model:
  type: RandomForestClassifier
  n_estimators: 100
  max_depth: 5
  random_state: 42

data:
  train_path: /root/code/fraud-detection/data/train.csv
  target_column: is_fraud

output:
  model_path: /root/code/fraud-detection/models/model.pkl

mlflow:
  tracking_uri: http://localhost:5000
  experiment_name: fraud-detection
```

### Step 3: Run the trainer

```bash
python3 /root/code/fraud-detection/src/models/train.py
```

**Expected output:**
```
accuracy=0.8000, f1_score=0.8261
model saved to /root/code/fraud-detection/models/model.pkl
🏃 View run puzzled-cub-515 at: http://localhost:5000/#/experiments/1/runs/...
🧪 View experiment at: http://localhost:5000/#/experiments/1
```

### Step 4: Verify the model file exists

```bash
ls -l /root/code/fraud-detection/models/model.pkl
```

**Expected output:**
```
-rw-r--r-- 1 root root 318521 Jun 18 00:07 /root/code/fraud-detection/models/model.pkl
```

### Step 5: Verify the MLflow run

Open the MLflow UI and confirm:
- Experiment: `fraud-detection`
- Exactly one new run exists
- Parameters logged: `model_type`, `n_estimators`, `max_depth`, `random_state`
- Metrics logged: `accuracy`, `f1_score`

## How `train.py` Resolves the Estimator

The trainer uses a **registry pattern** — only exact class names from scikit-learn are accepted:

```python
ESTIMATOR_REGISTRY = {
    "RandomForestClassifier": RandomForestClassifier,
    "GradientBoostingClassifier": GradientBoostingClassifier,
    "LogisticRegression": LogisticRegression,
}
```

If `model.type` doesn't match a key exactly, the script exits with a clear error rather than a confusing sklearn traceback. This is a deliberate design choice that makes config bugs immediately visible.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Config-Driven Training** | Hyperparameters live in YAML, not in code — swap models without touching Python |
| **Estimator Registry** | A dict mapping class-name strings to actual sklearn classes; unknown names fail fast |
| **`mlflow.log_params()`** | Logs all remaining YAML keys (after removing `type`) as MLflow parameters automatically |
| **`os.makedirs(..., exist_ok=True)`** | Creates the `models/` directory if it doesn't exist before saving |
| **Reproducibility** | Same config → same run → same model; anyone with the YAML can reproduce the exact experiment |

## Common Pitfalls

1. **Wrong estimator name** — `RandomForest` ≠ `RandomForestClassifier`; the registry match is case-sensitive and exact.
2. **Wrong target column** — The CSV has `is_fraud`, not `target`; mismatching causes an immediate `KeyError`.
3. **Model path outside the project** — `/root/code/model.pkl` writes the file one level above the project, making it invisible to DVC and other pipeline tools.
4. **Modifying `train.py`** — The scaffold is authoritative; the fix must live entirely in the YAML config.

## Summary

This task reinforces the principle that ML training code and ML configuration should be completely separated. By keeping all hyperparameters and paths in a YAML file, the pipeline becomes reproducible and auditable — different team members can run different experiments by simply swapping config files, and every experiment is automatically logged to MLflow with its full parameter set.

### Screenshots
<img width="1575" height="778" alt="MLflow run showing fraud-detection experiment with logged parameters" src="https://github.com/user-attachments/assets/328b292b-da8a-47ae-b4c9-5180f9f60e9b" />
