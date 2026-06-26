# Day 37 - Promote the Winning Model to MLflow Model Registry

The xFusionCorp Industries ML platform team runs fraud-detection training as a four-stage pipeline—preprocess, featurize, train, evaluate—orchestrated by a single script that logs the end-to-end run to MLflow. A pre-staged pipeline is already in place, but the stage-chain invariant is broken: the pipeline currently produces a feature matrix that does not reflect the upstream drop-and-clean work. Your task is to correct the stage wiring so every stage reads from its immediate predecessor and one MLflow run captures the full pipeline.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty training-pipeline experiment.

The project layout under /root/code/fraud-detection/:

data/raw/train.csv – The same 200-row synthetic binary-classification dataset the rest of the Training section uses (imbalanced roughly 70 / 30).
configs/pipeline_config.yaml – Declares the data paths, model hyperparameters, output paths, and MLflow settings every stage consumes. Correct and must remain intact.
src/preprocess.py, src/featurize.py, src/train.py, src/evaluate.py – The four pipeline stages. preprocess.py drops negligible-amount rows (amount < 50) and duplicates before writing the processed CSV. The four stages are wired through the config's data: paths.
run_pipeline.py – The orchestrator that executes the four stages in order and logs one MLflow run with the config-driven parameters and the final evaluation metrics. Correct and requires no edits.
Identify the stage whose input path breaks the chain, correct the wiring in the VS Code editor, save, and run python3 run_pipeline.py once from the project root.

The end state must include:

The row count of data/features/features.csv equals the row count of data/processed/train_clean.csv and is strictly less than the 200-row raw CSV.
models/model.pkl and reports/evaluation.json are written and the report carries accuracy, f1, and roc_auc as numeric values.
Exactly one MLflow run exists in the training-pipeline experiment, carrying params.model_type, params.n_estimators, params.max_depth, and the three evaluation metrics.

## Objective
Register the winning model artifact from the bake-off experiment into the MLflow Model Registry and assign it a production-ready alias (`champion`) so downstream deployment pipelines can reference a stable, named pointer instead of a raw `run_id`.

## Task
- **Read the Winner Report**: Load `reports/winner.json` and extract the correct `run_id` of the winning model.
- **Register the Model**: Use the MLflow client to register the model logged in that run under the name `fraud-detector`.
- **Assign the Champion Alias**: Set the `champion` alias on the newly registered version so deployment scripts can always resolve the latest best model by name.

## Solution

### Step 1: Fix the Report Key Lookup
The original script was reading `winner["model"]` which does not exist in the JSON. The correct key, as written by the bake-off orchestrator, is `run_id`.
```python
import json
import mlflow
from mlflow import MlflowClient

with open("reports/winner.json") as f:
    winner = json.load(f)

run_id = winner["run_id"]          # Fixed: was winner["model"]
model_name = "fraud-detector"
```

### Step 2: Register the Model Version
Use `mlflow.register_model` to create a new version under the `fraud-detector` registered model name. The `runs:/` URI links the registry entry directly to the logged artifact.
```python
model_uri = f"runs:/{run_id}/model"

mv = mlflow.register_model(
    model_uri=model_uri,
    name=model_name,
)

print(f"Registered model '{model_name}' version {mv.version} from run {run_id}")
```

### Step 3: Assign the Champion Alias
The original script called `client.set_model_version_tag(...)` instead of `client.set_registered_model_alias(...)`. Aliases are stable, human-readable pointers that always resolve to a specific version.
```python
client = MlflowClient()

client.set_registered_model_alias(
    name=model_name,
    alias="champion",
    version=mv.version,
)

print(f"Alias 'champion' → version {mv.version}")
```

### Step 4: Run the Registration Script and Verify
Execute the corrected script and confirm the alias is assigned via the MLflow UI or the client API.
```bash
cd /root/code/fraud-detection
python src/models/register.py
```

Verify programmatically:
```python
client = MlflowClient()
champion_mv = client.get_model_version_by_alias("fraud-detector", "champion")
print(champion_mv.version, champion_mv.run_id)
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **MLflow Model Registry** | A centralized, versioned store for ML models that supports lifecycle stages and annotations, decoupling model identity from a raw `run_id`. |
| **`runs:/` URI** | A portable URI scheme (`runs:/<run_id>/artifact_path`) that links a Registry version directly to a specific experiment artifact. |
| **Model Version** | An immutable snapshot of a model artifact created every time `mlflow.register_model` is called, automatically assigned an incrementing integer. |
| **Alias** | A mutable, human-readable pointer (e.g., `champion`) that can be re-assigned to any version, allowing deployment scripts to reference a stable name rather than a hard-coded version number. |
| **MlflowClient** | The low-level Python API for performing administrative Registry operations such as creating aliases, adding descriptions, and transitioning lifecycle stages. |

## Summary
By correcting the key lookup and switching from a tag to an alias, the registration pipeline now reliably promotes the bake-off winner into the MLflow Model Registry with a `champion` alias. Downstream deployment automation (CI/CD pipelines, batch scoring jobs, or REST serving endpoints) can always load the current best model using `models:/fraud-detector@champion` without ever needing to know a numeric version or a raw `run_id`. This is the foundational step toward a fully automated MLOps promotion workflow.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8cf218f9-9d3a-451b-9bf6-c23488793830" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/53d40416-0327-4775-a558-7f0991294e1b" />
