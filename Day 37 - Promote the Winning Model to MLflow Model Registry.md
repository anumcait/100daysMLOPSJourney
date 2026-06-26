# Day 37 - Promote the Winning Model to MLflow Model Registry

The xFusionCorp Industries ML platform team has a `reports/winner.json` from the Day 36 bake-off that identifies the best fraud-detection candidate. The next step in the pipeline is to register that winning model in the MLflow Model Registry so it can be versioned, annotated, and promoted through the staging → production lifecycle. A registration script exists at `/root/code/fraud-detection/src/models/register.py`, but it reads the wrong key from the report and does not set a model alias after registration. Your task is to fix the script so the correct run is registered under the `fraud-detector` model name and the new version is assigned the `champion` alias.

The MLflow tracking server is already running on port 5000. The `reports/winner.json` produced in Day 36 contains the keys `model_type`, `run_id`, and `f1_score`.

The project layout under `/root/code/fraud-detection/`:
- `reports/winner.json` – Output of the Day 36 bake-off containing the winning run's metadata.
- `src/models/register.py` – The registration script. Loading and alias-assignment logic is scaffolded but contains two bugs.

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
