# Day 35 - Hyperparameter Tuning with Optuna

The xFusionCorp Industries ML platform team tunes fraud-detection hyperparameters with Optuna and inspects the full search in the MLflow Compare view. A draft tuner exists at `/root/code/fraud-detection/src/models/tune.py`, but the search optimises in the wrong direction and no trials ever land on the tracking server. Your task is to correct the tuner so each of the 20 trials is visible in MLflow and the saved best configuration is actually the highest-F1 candidate.

The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty hyperopt-tuning experiment.

The project layout under `/root/code/fraud-detection/`:
- `data/train.csv` – The same 200-row synthetic binary-classification dataset Day 35 uses (imbalanced roughly 70 / 30).
- `src/models/tune.py` – The Optuna tuner scaffold. Fold iteration, metric averaging, Optuna study creation, and YAML persistence are already wired; corrections are required.
- `configs/` – Where `best_params.yaml` is written after the search completes.

## Objective
Automate the hyperparameter optimization process using Optuna while ensuring complete observability of the search space by logging all trials to MLflow.

## Task
- Update the Optuna study to maximize the F1 score instead of minimizing it.
- Modify the objective function to log each trial as a separate run in MLflow with its corresponding parameters and metrics.
- Verify that the search completes 20 trials and saves the best parameters to a YAML file.

## Solution

### Step 1: Set Search Direction
We updated the Optuna study initialization to use `direction="maximize"`. Since the optimization target is the F1 score, we want the highest possible value.
```python
study = optuna.create_study(
    direction="maximize", 
    study_name=EXPERIMENT_NAME
)
```

### Step 2: Implement Per-Trial Logging
We modified the `objective` function to wrap the model training and evaluation inside an `mlflow.start_run()` block. This ensures that every trial attempted by Optuna is recorded as a nested or individual run in the MLflow tracking server.
```python
def objective(trial, X, y):
    n_estimators = trial.suggest_int("n_estimators", 50, 500)
    max_depth = trial.suggest_int("max_depth", 3, 20)

    model = RandomForestClassifier(
        n_estimators=n_estimators,
        max_depth=max_depth,
        random_state=42,
    )

    scores = cross_val_score(model, X, y, cv=cv, scoring="f1")
    score = float(np.mean(scores))

    with mlflow.start_run():
        mlflow.log_param("n_estimators", n_estimators)
        mlflow.log_param("max_depth", max_depth)
        mlflow.log_metric("f1_score", score)

    return score
```

### Step 3: Run and Verify
Executed the tuner script and verified that 20 runs appear in the MLflow UI under the `hyperopt-tuning` experiment.
```bash
cd /root/code/fraud-detection
python src/models/tune.py
cat configs/best_params.yaml
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Optuna Study** | A session of hyperparameter optimization that manages multiple trials to find the best configuration. |
| **Optimization Direction** | Defines whether Optuna should search for the minimum (e.g., error) or maximum (e.g., accuracy/F1) value of an objective function. |
| **Experiment Tracking** | The process of logging parameters, metrics, and artifacts for every model iteration to enable comparison and reproducibility. |

## Summary
By correcting the optimization direction and implementing per-trial logging, we've transformed a silent, incorrect search into a robust, observable hyperparameter tuning pipeline. This setup allows the team to visually compare the performance of different parameter combinations in MLflow and guarantees that the best configuration is saved for production use.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/83dbb17c-1880-4c82-a059-ce0fec45968f" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c50bb8c0-8607-4b38-90e8-90aff68af37a" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/cb54adcc-0f63-4e68-93f3-bacb1f2be189" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/c82cf7f4-112b-494d-85b6-02fa93d19fb2" />





