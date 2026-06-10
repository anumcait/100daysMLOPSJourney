# Day 24: Enable MLflow Autologging
The xFusionCorp Industries ML team wants to replace the manual log_param / log_metric boilerplate in their training scripts with MLflow's autologging feature, so every training run captures its constructor parameters, training metrics, and model artefact automatically. A training scaffold has been pre-staged at /root/code/autolog_experiment.py—it configures MLflow, fits a small synthetic sklearn model, and prints a confirmation message. Two # TODO blocks remain empty. Your task is to complete them so the end state below holds.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to view the dashboard; only the Default experiment is present on first load.

Open /root/code/autolog_experiment.py in the VS Code editor and complete the two TODO blocks—both are one-line additions—so that, after the script is executed, the following end state holds:

An experiment named autolog-demo exists on the MLflow server.
At least one run exists in the autolog-demo experiment.
The run's Parameters panel lists every sklearn constructor parameter that the LogisticRegression in the scaffold implicitly carries (for example C, max_iter, solver, tol, penalty) – Not only the three explicit keyword arguments the scaffold passes.
The Artifacts panel on the run contains a model directory with an MLmodel descriptor and a pickled estimator.
Once the TODOs are in place, execute the script:

   python3 /root/code/autolog_experiment.py

Confirm the result in the MLflow UI.

No real dataset is loaded by the scaffold—the training step is a deterministic toy that gives MLflow a .fit() call to observe. The focus of the lab is autolog configuration, not model quality.
## Task Description
The xFusionCorp Industries team wants to simplify their training scripts by removing manual logging calls. The goal is to use MLflow's autologging feature to automatically capture model parameters, training metrics, and artifacts during the training process.

### Requirements:
1. Connect to the MLflow Tracking Server at `http://localhost:5000`.
2. Enable autologging for the scikit-learn flavor.
3. Set the active experiment to `autolog-demo`.
4. Train a `LogisticRegression` model on synthetic data.
5. Verify that:
   - All constructor parameters (including defaults) are logged.
   - Training metrics and model artifacts (pickled model, MLmodel file) are captured automatically.

## Solution

### Completed Training Script (`autolog_experiment.py`):

```python
import numpy as np
import mlflow
import mlflow.sklearn
from sklearn.linear_model import LogisticRegression

# Configure tracking server
mlflow.set_tracking_uri("http://localhost:5000")

# TODO 1: Enable autologging for sklearn
mlflow.sklearn.autolog()

# TODO 2: Set the active experiment
mlflow.set_experiment("autolog-demo")

# Synthetic XOR-like dataset
X = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
y = np.array([0, 0, 1, 1])

# Initialize and fit the model
# Autologging hooks into the .fit() call
model = LogisticRegression(C=1.0, max_iter=100, random_state=42)
model.fit(X, y)

print("Autolog run complete — check the MLflow UI")
```

### To execute:
```bash
python3 /root/code/autolog_experiment.py
```

## Verification

1. **Experiment Creation**: An experiment named `autolog-demo` should appear in the MLflow UI.
2. **Automatic Parameters**: Open the run and check the **Parameters** panel. You should see not just `C` and `max_iter`, but also default parameters like `solver`, `tol`, `penalty`, etc.
3. **Artifacts**: The **Artifacts** panel should contain a `model` directory with:
   - `MLmodel`: Metadata about the model.
   - `model.pkl`: The serialized scikit-learn estimator.
   - Dependency files (`conda.yaml`, `requirements.txt`).

### Screenshots
<img width="1603" height="781" alt="MLflow Autologged Parameters" src="https://github.com/user-attachments/assets/placeholder-autolog-params" />
<img width="1603" height="781" alt="MLflow Autologged Artifacts" src="https://github.com/user-attachments/assets/placeholder-autolog-artifacts" />
