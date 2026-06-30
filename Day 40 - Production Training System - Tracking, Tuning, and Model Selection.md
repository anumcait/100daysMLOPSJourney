# Day 40 - Production Training System: Tracking, Tuning, and Model Selection

## Objective
The xFusionCorp Industries ML platform team has a five-stage production training pipeline (`validate_data → tune → select_model → register → report`) orchestrated by a `Makefile`. Running `make train-pipeline` surfaces three wiring bugs — the stages execute in the wrong order, the model-selection stage searches for a metric the tuner never logs, and the registration stage assigns the wrong alias while failing to clean up stale ones. The task is to identify each defect in turn, fix it with a one-line edit, and re-run until the full pipeline completes without a non-zero exit.

The MLflow tracking server is already running on port 5000. The `fraud-detection-tuning` experiment is pre-created.

The project layout under `/root/code/fraud-detection/`:

- `data/train.csv` — The 200-row synthetic binary-classification dataset.
- `src/validate_data.py` — Schema + null-check gate. Writes `reports/validation_status.json`. ✅ Correct.
- `src/tune.py` — Runs 10 Optuna trials across RandomForest and GradientBoosting, each logged as an MLflow run with `model_type`, params (`n_estimators`, `max_depth`), `metrics.f1_score`, and the fitted model artifact. ✅ Correct.
- `src/select_model.py` — Picks the winning run by training metric and writes `reports/selection.json`. ⚠️ Needs attention.
- `src/register.py` — Registers the selected run's model as `fraud-detector` and assigns the release-lane alias. ⚠️ Needs attention.
- `src/report.py` — Aggregates every upstream artifact into `reports/training_report.json`. ✅ Correct.
- `Makefile` — `train-pipeline` target runs the five stages in order. ⚠️ Needs attention.

The end state must include:

- `make train-pipeline` completes without non-zero exit.
- The `fraud-detection-tuning` MLflow experiment carries at least five trial runs, each with `metrics.f1_score`.
- `reports/selection.json`, `reports/validation_status.json`, and `reports/training_report.json` are all present.
- The training report carries `best_model`, `best_params`, `metrics`, `total_trials`, and `validation_status` keys; `validation_status` is `"ok"` and `total_trials` is an integer ≥ 5.
- The MLflow Model Registry shows a `fraud-detector` registered model with at least one version carrying the `staging` alias and **no** `production` alias.

## Task
- **Fix the Makefile**: Correct the stage execution order so `tune` runs before `select_model`.
- **Fix `select_model.py`**: Replace the `metrics.accuracy` sort key with `metrics.f1_score` to match what the tuner actually logs.
- **Fix `register.py`**: Set `RELEASE_ALIAS = "staging"` and delete any pre-existing `production` / `staging` aliases before assigning the new one.

## Solution

### Step 1: Run the Pipeline to Surface the First Bug
Execute the pipeline as-is so the first failure is visible:

```bash
cd /root/code/fraud-detection
make train-pipeline
```

The pipeline exits non-zero immediately — `select_model.py` ran **before** `tune.py`, so no MLflow runs exist yet and it finds nothing to select.

### Step 2: Fix the Makefile — Stage Execution Order

**Before (broken order):**
```makefile
train-pipeline:
	python src/validate_data.py
	python src/select_model.py
	python src/tune.py
	python src/register.py
	python src/report.py
```

**After (correct order):**
```makefile
train-pipeline:
	python src/validate_data.py
	python src/tune.py
	python src/select_model.py
	python src/register.py
	python src/report.py
```

### Step 3: Re-Run and Surface the Second Bug
```bash
make train-pipeline
```

Now `tune.py` runs first and logs `metrics.f1_score` for every Optuna trial. But `select_model.py` then fails — it tries to sort runs by `metrics.accuracy`, a metric that was never logged.

### Step 4: Fix `select_model.py` — Wrong Sort Metric

**Before (broken):**
```python
runs = client.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.accuracy DESC"]   # ← metric never logged by tune.py
)
best_score = best.data.metrics["metrics.accuracy"]   # ← KeyError
```

**After (correct):**
```python
runs = client.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.f1_score DESC"]   # ← matches tune.py
)
best_score = best.data.metrics["f1_score"]
```

### Step 5: Re-Run and Surface the Third Bug
```bash
make train-pipeline
```

Selection succeeds and writes `reports/selection.json`. But `register.py` fails — it assigns a `production` alias instead of `staging`, and never cleans up stale aliases from previous runs.

### Step 6: Fix `register.py` — Wrong Alias and No Cleanup

**Before (broken):**
```python
RELEASE_ALIAS = "production"   # ← wrong alias

client.set_registered_model_alias(
    REGISTERED_MODEL_NAME,
    RELEASE_ALIAS,
    version.version
)
```

**After (correct):**
```python
RELEASE_ALIAS = "staging"

# Remove any stale aliases before assigning the new one
for alias in ["production", "staging"]:
    try:
        client.delete_registered_model_alias(
            REGISTERED_MODEL_NAME,
            alias
        )
    except Exception:
        pass

client.set_registered_model_alias(
    REGISTERED_MODEL_NAME,
    RELEASE_ALIAS,
    version.version
)
```

### Step 7: Final Clean Run
```bash
make train-pipeline
```

Expected terminal output:

```
python src/validate_data.py
✅ Validation passed — writing reports/validation_status.json
python src/tune.py
[I 0] Trial 0 finished with value: 0.8421 ...
...
[I 9] Trial 9 finished with value: 0.8912 ...
python src/select_model.py
Best run: <run_id>  f1_score=0.8912
Writing reports/selection.json ...
python src/register.py
Registered model: fraud-detector  version=1  alias=staging
python src/report.py
Writing reports/training_report.json ...
✅ Pipeline complete.
```

### Step 8: Verify the Final Outputs

**Check the report files:**
```bash
cat reports/validation_status.json
cat reports/selection.json
cat reports/training_report.json
```

`training_report.json` must contain these top-level keys:
```json
{
  "best_model":        "GradientBoosting",
  "best_params":       { "n_estimators": 200, "max_depth": 5 },
  "metrics":           { "f1_score": 0.8912 },
  "total_trials":      10,
  "validation_status": "ok"
}
```

**Check the MLflow Model Registry alias:**
```bash
python3 -c "
from mlflow import MlflowClient
client = MlflowClient()
aliases = client.get_registered_model('fraud-detector').aliases
print('Aliases:', aliases)
"
```

Expected: `Aliases: {'staging': '1'}` — no `production` key present.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Pipeline Stage Ordering** | In a Makefile `train-pipeline` target, each shell command runs sequentially top to bottom. Running `select_model` before `tune` queries an empty MLflow experiment and crashes immediately. The correct order is: validate → tune → select → register → report. |
| **Metric Consistency (tune ↔ select)** | The tuning stage and the selection stage must agree on the exact MLflow metric key. If `tune.py` logs `metrics.f1_score` but `select_model.py` queries `metrics.accuracy`, `search_runs` returns an empty list and the pipeline fails. Always cross-check both files when debugging selection failures. |
| **MLflow Model Aliases** | Aliases (`staging`, `champion`, `production`) are mutable pointers in the Model Registry assigned to specific versions. They persist across pipeline runs — a stale `production` alias survives re-registration unless explicitly deleted. |
| **`client.delete_registered_model_alias`** | The safe, idempotent way to clear an alias before reassigning it. Wrapping in a `try/except` prevents crashes on the first run when no alias exists yet. |
| **`make train-pipeline`** | A Makefile target that chains multiple Python scripts into a single reproducible command. Each script's non-zero exit propagates upward, making `make` ideal for debugging multi-stage bugs one at a time. |
| **`reports/training_report.json`** | A consolidated artifact capturing the full pipeline output: validation status, best model identity, best hyperparameters, metrics, and trial count — serving as both documentation and a machine-readable audit trail. |

## Summary
Three minimal edits were required to bring the five-stage production training pipeline to a clean run:

1. **Makefile** — moved `select_model` after `tune` so MLflow runs exist before the selection stage queries them.
2. **`select_model.py`** — changed the `order_by` sort key from `metrics.accuracy` to `metrics.f1_score` to match what the tuner actually logs.
3. **`register.py`** — changed `RELEASE_ALIAS` from `"production"` to `"staging"` and added a cleanup loop that deletes stale aliases before assigning the new one.

After all three fixes, `make train-pipeline` completes successfully, producing all three report files and registering `fraud-detector` in the MLflow Model Registry with only the `staging` alias — fulfilling the full production training system requirement: tracking, tuning, and model selection.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e0c56db8-e81f-4902-863b-ee215d5192a4" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5e737f95-edd6-41bd-aff5-55cf98603321" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2cdf2a78-139a-4fb1-847c-9d9010104f07" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6985e298-7302-43f7-9f1c-733d37dd919e" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/42208a0a-f72b-4581-b74d-501630541cc1" />






