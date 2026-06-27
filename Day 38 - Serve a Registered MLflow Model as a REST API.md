# Day 38 - Serve a Registered MLflow Model as a REST API
The xFusionCorp Industries ML platform team runs a parallel-training bake-off on the fraud-detection model—the same estimator is trained twice, once on a single worker and once across every available CPU, and the MLflow Compare view surfaces the wall-time gap. A draft script exists at /root/code/fraud-detection/src/models/train_parallel.py, but running it currently produces near-identical wall times for the 'serial' and 'parallel' runs, and the Compare view cannot distinguish the two configurations. Your task is to correct the script so the second run genuinely runs in parallel and every MLflow run records the number of workers it actually used.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty parallel-training experiment.

The project layout under /root/code/fraud-detection/:

data/train.csv – A 5000-row synthetic binary-classification dataset (imbalanced roughly 70 / 30). Larger than the 200-row dataset used earlier in the section because the n_jobs speedup is only visible once there is enough work per tree.
src/models/train_parallel.py – The bake-off script. Data loading, MLflow experiment setup, wall-time measurement, metrics.training_time_seconds logging, and model persistence to models/model.pkl are already wired; corrections are required.
Open src/models/train_parallel.py in the VS Code editor, correct every issue that keeps the bake-off from meeting the release checklist, save, and run the script once.

The end state must include:

At least two runs exist in the parallel-training experiment on MLflow. Across the two runs, params.n_jobs takes the values 1 and -1 (no run still carries the hardcoded "all").
Every run carries metrics.training_time_seconds, and the n_jobs = -1 run is measurably faster than the n_jobs = 1 run (at least 10 %).
A pickled model at /root/code/fraud-detection/models/model.pkl.

### Objective
The xFusionCorp Industries ML platform team has successfully registered the fraud-detection champion model in the MLflow Model Registry (Day 37). The next step in the deployment pipeline is to expose that champion model as a live REST API so that upstream services can submit transaction features and receive fraud predictions in real time. A serving script at `src/serve/predict.py` exists but is broken — it loads the model using a hard-coded `run_id` instead of the stable `models:/fraud-detector@champion` alias URI, and the Flask route returns raw NumPy types that are not JSON-serializable. Your task is to correct both defects so the endpoint accepts a JSON payload of features and returns a valid prediction response.

The MLflow tracking server is already running on port 5000. The `fraud-detector` model with the `champion` alias is already registered (from Day 37).

The project layout under /root/code/fraud-detection/:

data/raw/train.csv – The same 200-row synthetic binary-classification dataset.
reports/winner.json – Contains `model_type`, `run_id`, and `f1_score` of the winning candidate.
src/serve/predict.py – The Flask serving script. Two specific corrections are required.

The end state must include:

The server starts without errors on port 8080.
A POST to `/predict` with a valid JSON feature payload returns `{"prediction": 0}` or `{"prediction": 1}`.
The model is loaded using the Registry alias URI (`models:/fraud-detector@champion`), not a raw `run_id`.

## Objective
Load the registered champion model from the MLflow Model Registry by its stable alias URI and expose it through a Flask REST endpoint that accepts JSON feature payloads and returns JSON-serializable fraud predictions.

## Task
- **Fix Model Loading**: Replace the hard-coded `run_id` URI with the `models:/fraud-detector@champion` alias URI so the serving layer is decoupled from specific version numbers.
- **Fix JSON Serialization**: Convert the NumPy prediction result to a native Python `int` before returning it in the JSON response.
- **Start and Verify the Server**: Launch the Flask application and validate it with a `curl` test request.

## Solution

### Step 1: Fix the Model Loading URI
The original script used `runs:/<run_id>/model` which breaks whenever the champion is re-promoted to a new version. Switch to the Registry alias URI so it always resolves to the current champion without code changes.
```python
import mlflow.sklearn

# Before (broken — hard-coded to a specific run):
# model = mlflow.sklearn.load_model("runs:/abc123def456/model")

# After (correct — resolves via Registry alias):
model = mlflow.sklearn.load_model("models:/fraud-detector@champion")
```

### Step 2: Fix the Flask Prediction Route
The original route returned a NumPy `int64` value directly, which `flask.jsonify` cannot serialize. Cast it to a native Python `int`.
```python
from flask import Flask, request, jsonify
import pandas as pd
import mlflow.sklearn

app = Flask(__name__)

# Load model once at startup using the stable alias URI
model = mlflow.sklearn.load_model("models:/fraud-detector@champion")

@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json(force=True)
    features = pd.DataFrame([data["features"]])
    prediction = model.predict(features)
    # Fixed: cast numpy int64 → native Python int for JSON serialization
    return jsonify({"prediction": int(prediction[0])})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

### Step 3: Start the Serving Endpoint
Launch the Flask server in the background from the project root:
```bash
cd /root/code/fraud-detection
python src/serve/predict.py &
```

### Step 4: Validate with a Test Request
Send a sample feature payload and confirm a valid JSON prediction is returned:
```bash
curl -s -X POST http://localhost:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [1200.50, 1, 0, 3, 0.85, 22, 1, 0, 1, 0]}' | python3 -m json.tool
```

Expected response:
```json
{
    "prediction": 0
}
```

### Step 5: Verify the Correct Model Version Was Loaded
Confirm the alias resolves to the expected version programmatically:
```python
from mlflow import MlflowClient

client = MlflowClient()
mv = client.get_model_version_by_alias("fraud-detector", "champion")
print(f"Serving model version {mv.version} from run {mv.run_id}")
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **`models:/` URI Scheme** | A Registry-aware URI (`models:/<name>@<alias>` or `models:/<name>/<version>`) that MLflow resolves to the correct artifact at load time, decoupling the serving layer from raw `run_id` values. |
| **Model Alias as Stable Pointer** | The `champion` alias is a mutable pointer that can be re-assigned to any new version during a future promotion. Serving code referencing the alias requires zero changes after a model update. |
| **Flask REST Endpoint** | A lightweight HTTP server exposing an ML model as a `/predict` route; accepts JSON feature arrays and returns JSON predictions, making the model consumable by any HTTP client. |
| **NumPy ↔ JSON Serialization** | NumPy scalar types (e.g., `int64`, `float32`) are not natively JSON-serializable. Always cast predictions to Python builtins (`int`, `float`) before passing them to `jsonify`. |
| **Startup Loading Pattern** | Loading the model once at server startup (outside the request handler) avoids repeated deserialization on every request, significantly reducing per-request latency. |

## Summary
By switching from a raw `run_id` URI to the `models:/fraud-detector@champion` alias URI and casting the NumPy prediction to a native Python `int`, the serving script now correctly loads the current production champion from the MLflow Registry and returns well-formed JSON responses. This pattern fully decouples the deployment layer from specific run or version identifiers — future promotions only require reassigning the `champion` alias, with no code changes to the serving infrastructure. This is the natural next step after model registration: moving from a named artifact in the Registry to a live, queryable prediction service.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8b283161-bf07-4b91-a617-1afd806b57ae" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b3dde43d-8c9b-4b1b-ab90-1724a5030f56" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8a0abd38-c48f-4040-b639-e7c57c7be43c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9d78a615-c91e-4383-a4bf-12b05a2d6de0" />





