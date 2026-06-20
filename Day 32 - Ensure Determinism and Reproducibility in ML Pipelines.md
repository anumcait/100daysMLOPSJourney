# Day 32 - Ensure Determinism and Reproducibility in ML Pipelines

Ensuring that machine learning experiments produce identical results across different runs is critical for debugging, auditing, and collaborative research. In this task, we address non-determinism in a Scikit-Learn training script by explicitly seeding all random processes.

## Objective
Fix non-determinism in the model training script (`src/models/train.py`) to ensure that multiple training runs produce byte-identical metrics and feature importances.

## Task
- Identify unseeded random processes in the training script.
- Apply a fixed `RANDOM_STATE` to data partitioning and model initialization.
- Verify that `check_determinism.sh` exits with status 0, indicating perfectly reproducible runs.
- Confirm that accuracy, f1_score, and feature importances remain constant across three consecutive executions.

## Solution

### Step 1: Identify Sources of Randomness
In the original `src/models/train.py`, both the data splitting and the model training lacked a fixed random seed:
- `train_test_split()` used a different random shuffle every time it was called.
- `RandomForestClassifier()` used a different internal seed for individual tree construction (feature sampling and row bootstrapping).

### Step 2: Define a Global Random State
Added a constant `RANDOM_STATE` at the top of the script (after imports) to encourage consistency across all components.
```python
RANDOM_STATE = 42
```

### Step 3: Seed the Data Split
Modified the `train_test_split` call to include the `random_state` parameter. This ensures that the same rows land in the training and test sets every time the script is run.
```python
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.2,
    stratify=y,
    random_state=RANDOM_STATE,
)
```

### Step 4: Seed the ML Model
Updated the `RandomForestClassifier` instantiation to use the global seed. This forces the algorithm to make the same stochastic decisions (which features to split on, which subsets of data to use for each tree) during training.
```python
model = RandomForestClassifier(
    n_estimators=100,
    max_depth=5,
    random_state=RANDOM_STATE,
)
```

### Step 5: Verification with `check_determinism.sh`
Ran the provided verification script which executes the trainer three times and compares the resulting metrics files.
```bash
./check_determinism.sh
```

**Expected Result:**
```text
OK: all three runs produced byte-identical metrics.
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Determinism** | A system where no randomness is involved in the development of future states of the system. |
| **Reproducibility** | The ability to get the exact same results (metrics, weights, predictions) when running the same code on the same data. |
| **Random Seed / State** | A starting point for a pseudo-random number generator; using the same seed ensures the "random" sequence is identical. |
| **Stochastic Algorithms** | Algorithms like Random Forest or Gradient Boosting that use randomness to improve performance or stability; they must be seeded for determinism. |

## Summary
By explicitly seeding every component that introduces randomness — from data partitioning to model initialization — we transformed a stochastic training process into a fully deterministic one. This is a foundational MLOps requirement, ensuring that any result reported to MLflow can be perfectly recreated by any teammate using the same code and data version.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c6ebcaed-720c-4bbf-8aa0-9ac3002ab451" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d93ddc7a-47ef-4b2d-9027-97cd7188f869" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/786d4981-03f4-4f93-9c4f-f60a1bc8ac8f" />



