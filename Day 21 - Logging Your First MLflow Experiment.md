# Day 21: Log an ML Experiment to MLflow

A xFusionCorp Industries data scientist needs a training run recorded in MLflow so the team has a baseline record on the tracking dashboard. The non-MLflow scaffolding has already been written at /root/code/log_experiment.py; the MLflow logging calls are left as TODO blocks. Your task is to complete the script so that every element of the run is captured by the MLflow tracking server.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to view the dashboard; the Default experiment is present on first load.

/root/code/log_experiment.py can be opened in the VS Code editor. The script prepares a params dictionary, fits a trivial sklearn model, and advertises a pair of synthetic evaluation scores (accuracy and f1). Three blocks marked # TODO inside the mlflow.start_run() context are the only edits required.

Execute the script once (python3 /root/code/log_experiment.py) after the TODOs are completed. The end state must include:

Exactly one new run in the Default experiment.
Every hyperparameter in the params dict (n_estimators=100, max_depth=5, random_state=42) recorded as a run parameter.
Both advertised scores (accuracy, f1_score) recorded as run metrics.
The sklearn model captured as an MLflow model artefact on the run.
The result can be confirmed in the MLflow UI—once the run is opened, the Parameters, Metrics, and Artifacts panels each show the expected content.

## Task Description
The xFusionCorp Industries data science team needs to record their first baseline experiment using MLflow. The goal is to log parameters, metrics, and a model artifact for a simple scikit-learn model to the MLflow tracking server to establish an experiment tracking baseline.

### Requirements:
1. Create a Python script named `log_experiment.py` under `/root/code/`.
2. Configure the script to log:
   - Parameters: `n_estimators=100`, `max_depth=5`.
   - Metrics: `accuracy=0.85`, `f1_score=0.82`.
   - Artifact: A scikit-learn dummy model.
3. Execute the script and verify the run in the MLflow UI.

## Solution

### Run these commands on the controlplane host:

```bash
# Create the logging script
cat > /root/code/log_experiment.py <<EOF
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris
import os

if __name__ == "__main__":
    # Start an MLflow run
    with mlflow.start_run():
        # Log parameters
        mlflow.log_param("n_estimators", 100)
        mlflow.log_param("max_depth", 5)

        # Log metrics
        mlflow.log_metric("accuracy", 0.85)
        mlflow.log_metric("f1_score", 0.82)

        # Create and log a dummy model artifact
        iris = load_iris()
        model = RandomForestClassifier(n_estimators=100, max_depth=5)
        model.fit(iris.data, iris.target)
        
        # Log the model
        mlflow.sklearn.log_model(model, "model")
        
        print("Experiment run logged successfully!")
EOF

# Execute the script
python3 /root/code/log_experiment.py
```

### To verify:

1. Open the MLflow UI (usually at `http://localhost:5000` or the configured server address).
2. Open the **Default** experiment.
3. Confirm a new run was created.
4. Check that:
   - Parameters `n_estimators` and `max_depth` are recorded.
   - Metrics `accuracy` and `f1_score` are present.
   - The model artifact is available under the **Artifacts** section.

### Screenshots
<img width="1600" height="800" alt="MLflow UI Experiment Run" src="https://github.com/user-attachments/assets/placeholder-run-list" />
<img width="1600" height="800" alt="MLflow Run Details" src="https://github.com/user-attachments/assets/placeholder-run-details" />
