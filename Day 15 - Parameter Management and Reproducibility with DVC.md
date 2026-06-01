# Day 15 — Parameter Management with DVC

The xFusionCorp Industries ML team manages model hyperparameters through `params.yaml` so experiments can vary without code changes. The `fraud-detection` project's train stage already wires `params.yaml` for `n_estimators`, but `dvc repro` currently fails. The goal is to correct the parameter wiring and demonstrate that DVC re-runs the train stage when the parameter changes.

## 📋 Task Summary

Correct the `dvc.yaml` and `params.yaml` wiring to ensure:
- The `train` stage correctly consumes parameters from `params.yaml`.
- `dvc repro` completes successfully.
- Changing `n_estimators` triggers a re-run of only the `train` stage.
- The new parameter value is recorded in `dvc.lock`.

---

## 🛠️ Step 1: Review and Correct Parameter Wiring

First, ensure `params.yaml` contains the correct keys. The `train` stage in `dvc.yaml` expects a key that must exist in `params.yaml`.

### `params.yaml`
```yaml
train:
  n_estimators: 100
  learning_rate: 0.01
```

### Update `dvc.yaml`
Update the `train` stage to reference the parameters correctly.

```yaml
stages:
  process_data:
    cmd: python src/data/process_data.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean_transactions.csv

  split_data:
    cmd: python src/data/split_data.py
    deps:
      - data/processed/clean_transactions.csv
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv

  train:
    cmd: python src/models/train.py
    deps:
      - data/processed/train.csv
      - src/models/train.py
    params:
      - train.n_estimators
    outs:
      - models/model.pkl
```

---

## 🛠️ Step 2: Run the Pipeline

Execute the pipeline to ensure the wiring is correct and the model is trained.

```bash
cd /root/code/fraud-detection

# Run the pipeline
dvc repro
```

---

## 🛠️ Step 3: Demonstrate Reproducibility

Change the `n_estimators` value in `params.yaml` and observe that DVC only re-runs the `train` stage.

1.  **Modify `params.yaml`**:
    Change `n_estimators` from `100` to `200`.

2.  **Run `dvc repro` again**:
    ```bash
    dvc repro
    ```

### Expected Output
DVC will detect that only `params.yaml` has changed and will skip the `process_data` and `split_data` stages, re-running only `train`:

```text
Stage 'process_data' didn't change, skipping
Stage 'split_data' didn't change, skipping
Running stage 'train':
> python src/models/train.py
Updating lock file 'dvc.lock'
```

---

## 🛠️ Step 4: Verify the Lock File

Check `dvc.lock` to see that the new value is recorded.

```yaml
# dvc.lock (fragment)
train:
  cmd: python src/models/train.py
  deps:
  - path: data/processed/train.csv
    md5: ...
  - path: src/models/train.py
    md5: ...
  params:
    params.yaml:
      train.n_estimators: 200
  outs:
  - path: models/model.pkl
    md5: ...
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **`params.yaml`** | A dedicated file for storing hyperparameters and configuration settings. |
| **Parameter Wiring** | The mechanism by which DVC stages reference specific keys in `params.yaml` to track changes. |
| **Granular Reproducibility** | DVC only re-runs stages whose dependencies or parameters have changed, saving time and compute. |
| **`dvc.lock`** | Records the exact state (hashes and parameter values) of the last successful pipeline run. |

---

## ⚠️ Common Pitfalls

1.  **Missing Keys** — Every name listed under `params:` in `dvc.yaml` must exist as a key in `params.yaml`.
2.  **Incorrect Nesting** — If `params.yaml` uses nested keys (e.g., `train: n_estimators: 100`), they must be referenced using dot notation in `dvc.yaml` (e.g., `train.n_estimators`).
3.  **Hardcoded Hyperparameters** — Changing hyperparameters in the Python script without updating `params.yaml` breaks DVC's ability to track changes.
