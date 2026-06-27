# Day 39 - Parallelize Model Training and Compare Runs in MLflow

## Objective
The xFusionCorp Industries ML platform team runs a parallel-training bake-off on the fraud-detection model — the same estimator is trained twice, once on a single worker and once across every available CPU. The MLflow Compare view is used to surface the wall-time gap. A draft script exists at `src/models/train_parallel.py`, but running it produces near-identical wall times for the `serial` and `parallel` runs, and the Compare view cannot distinguish the two configurations. The task is to correct the script so the second run genuinely runs in parallel and every MLflow run records the number of workers it actually used.

## Task
- **Fix `n_jobs` Logging**: Replace the hard-coded string `"all"` parameter value with the actual integer `n_jobs` value (`1` or `-1`) so the Compare view can distinguish the two runs.
- **Enable True Parallelism**: Ensure the second `RandomForestClassifier` is instantiated with `n_jobs=-1` so it distributes tree-building across all available CPUs.
- **Record Wall-Time**: Log `metrics.training_time_seconds` for each run so the speedup is measurable.
- **Verify the Bake-Off**: Confirm that the `n_jobs=-1` run is measurably faster (at least 10%) and that a pickled model is saved to `models/model.pkl`.

## Solution

### Step 1: Inspect the Broken Script
Open `src/models/train_parallel.py` and identify the two defects.

```python
# Defect 1 — n_jobs logged as a hardcoded string, not the actual value:
mlflow.log_param("n_jobs", "all")   # WRONG — should be the integer -1

# Defect 2 — the classifier is still using the default n_jobs=1 on the parallel run:
clf = RandomForestClassifier(n_estimators=500, random_state=42)  # missing n_jobs=-1
```

### Step 2: Fix the Serial Run Block
Set `n_jobs=1` explicitly and log the integer value as the parameter.

```python
import time
import pickle
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
import pandas as pd

mlflow.set_experiment("parallel-training")

df = pd.read_csv("data/train.csv")
X = df.drop("target", axis=1)
y = df["target"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# --- Serial Run ---
with mlflow.start_run(run_name="serial"):
    n_jobs = 1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)

    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start

    mlflow.log_param("n_jobs", n_jobs)          # Fixed: log the integer, not "all"
    mlflow.log_metric("training_time_seconds", elapsed)
```

### Step 3: Fix the Parallel Run Block
Set `n_jobs=-1` to use all available CPU cores and log the correct integer value.

```python
# --- Parallel Run ---
with mlflow.start_run(run_name="parallel"):
    n_jobs = -1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)  # Fixed

    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start

    mlflow.log_param("n_jobs", n_jobs)          # Fixed: log -1, not "all"
    mlflow.log_metric("training_time_seconds", elapsed)

    # Save the parallel-trained model
    with open("models/model.pkl", "wb") as f:
        pickle.dump(clf, f)
```

### Step 4: The Corrected Full Script
The complete corrected `src/models/train_parallel.py`:

```python
import time
import pickle
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
import pandas as pd
import os

mlflow.set_experiment("parallel-training")

df = pd.read_csv("data/train.csv")
X = df.drop("target", axis=1)
y = df["target"]
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

os.makedirs("models", exist_ok=True)

# --- Serial Run (n_jobs=1) ---
with mlflow.start_run(run_name="serial"):
    n_jobs = 1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)
    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start
    mlflow.log_param("n_jobs", n_jobs)
    mlflow.log_metric("training_time_seconds", elapsed)
    print(f"Serial run: {elapsed:.2f}s  (n_jobs={n_jobs})")

# --- Parallel Run (n_jobs=-1) ---
with mlflow.start_run(run_name="parallel"):
    n_jobs = -1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)
    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start
    mlflow.log_param("n_jobs", n_jobs)
    mlflow.log_metric("training_time_seconds", elapsed)
    print(f"Parallel run: {elapsed:.2f}s  (n_jobs={n_jobs})")

    with open("models/model.pkl", "wb") as f:
        pickle.dump(clf, f)
```

### Step 5: Run the Script
Execute from the project root with the MLflow tracking server already running on port 5000:

```bash
cd /root/code/fraud-detection
export MLFLOW_TRACKING_URI=http://localhost:5000
python src/models/train_parallel.py
```

### Step 6: Verify in the MLflow UI
Open the MLflow UI and navigate to the **parallel-training** experiment. Select both runs and click **Compare**. The table should show:

| Run Name | `n_jobs` | `training_time_seconds` |
|----------|----------|------------------------|
| serial   | 1        | ~4.8 s                 |
| parallel | -1       | ~2.1 s                 |

Confirm that the parallel run is at least 10% faster than the serial run.

### Step 7: Verify the Saved Model
```bash
python3 -c "
import pickle
with open('models/model.pkl', 'rb') as f:
    model = pickle.load(f)
print('Model loaded:', model)
print('n_jobs param:', model.n_jobs)
"
```

Expected output confirms the saved model has `n_jobs=-1`.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **`n_jobs=-1`** | Instructs scikit-learn to use all available CPU cores. `-1` is shorthand for "use every core the OS reports". Setting `n_jobs=1` forces single-threaded execution for a fair serial baseline. |
| **Wall-Time Measurement** | `time.time()` captures a high-resolution timestamp before and after `fit()`. The difference (`elapsed`) gives the real-world (wall-clock) training duration in seconds, regardless of how many threads were used. |
| **MLflow `log_param` vs `log_metric`** | `log_param` stores categorical/configuration values (e.g., `n_jobs=-1`) that describe *how* a run was configured. `log_metric` stores numerical measurements (e.g., `training_time_seconds`) that quantify *how well or how fast* the run performed. |
| **MLflow Compare View** | Selecting two or more runs in the MLflow UI and clicking **Compare** renders a side-by-side diff of all logged params and metrics, making it easy to attribute performance differences to specific configuration changes. |
| **Bake-Off Pattern** | Running the same estimator under two or more configurations (e.g., serial vs. parallel, different `n_estimators`) in a single script is a common MLOps practice for generating reproducible, tracked benchmarks without manual bookkeeping. |
| **`RandomForestClassifier` Parallelism** | Each decision tree in a Random Forest is independent, making the ensemble trivially parallelizable. scikit-learn uses `joblib` under the hood to distribute tree-building across the `n_jobs` processes or threads. |

## Summary
By replacing the hardcoded string `"all"` with the actual integer `n_jobs` variable in every `mlflow.log_param` call, and by explicitly passing `n_jobs=-1` to the second `RandomForestClassifier`, the bake-off script now produces two genuinely distinct runs in the `parallel-training` experiment. The MLflow Compare view can differentiate the configurations by their `params.n_jobs` values (`1` vs. `-1`), and the `metrics.training_time_seconds` delta confirms the real-world speedup from multi-core training. This pattern — running controlled configuration variants and tracking every hyperparameter and metric in MLflow — is the foundation of reproducible, auditable model development in an MLOps pipeline.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/placeholder-day39-01" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/placeholder-day39-02" />
