````
# Day 97: Capstone (1/4) — End-to-End MLOps System - Train, Register, Serve

The xFusionCorp Industries MLOps team is standing up the serving path of their fraud-detector platform — the route a model takes from a training run to a live prediction endpoint. The backing stack is already running: a SeaweedFS object store (with a seeded training dataset) and an MLflow tracking server. Your task is to drive that path end to end: run the training script to produce a run, complete register.py so it registers the run as the fraud-detector model and assigns the production alias, then start the FastAPI inference server so it serves live predictions from the aliased model.


The MLflow UI and SeaweedFS Filer buttons at the top of the lab open those UIs. Pre-staged state:

SeaweedFS bucket data holds transactions.csv; the mlflow-artifacts bucket is empty until a run logs to it.
MLflow experiment fraud-detection does not exist yet and the Model Registry is empty.
Under /root/code/: train.py, serve.py, and config.yaml are reference scripts (no edits needed); register.py ships with its register + promote step as an unfinished TODO for you to complete. train.py reads the dataset from SeaweedFS and logs the run + artefacts to MLflow; the FastAPI server's background loader polls the registry for models:/fraud-detector@production and loads the model once the alias exists.
The end state must include:

MLflow experiment fraud-detection has at least one run; run artefacts are in the mlflow-artifacts SeaweedFS bucket.
A Registered Model named fraud-detector exists with the production alias assigned to the version sourced from the run, set in code by register.py.
POST http://localhost:8085/predict with {"features": [100.5, 12, 3]} returns {"prediction": 0} or {"prediction": 1} (tests poll up to 60 s).
This task builds the serving path of the platform: training logs a run + artefacts to MLflow (backed by SeaweedFS object storage), the registry production alias is the stable handle production code targets (models:/fraud-detector@production), and the serving process resolves that alias at load time — so promoting a new model later is an alias move, not a redeploy.

## Objective

Build the complete fraud-detector serving path:

**Train → Track artifacts → Register model → Assign production alias → Serve → Predict**

The platform uses:

- SeaweedFS for object storage
- MLflow for experiment tracking and model registry
- RandomForestClassifier for fraud detection
- FastAPI for model serving
- `models:/fraud-detector@production` as the stable production model reference

## Pre-staged Environment

The lab provides:

- SeaweedFS with a `data` bucket containing `transactions.csv`
- An `mlflow-artifacts` bucket for MLflow artifacts
- MLflow tracking server at `http://localhost:5000`
- `/root/code/train.py`
- `/root/code/register.py`
- `/root/code/serve.py`
- `/root/code/config.yaml`

The MLflow experiment and Model Registry start empty.

## 1. Inspect the Code

Move into the project directory:

```bash
cd /root/code
```

Inspect the supplied files:

```bash
sed -n '1,240p' train.py
sed -n '1,240p' register.py
sed -n '1,240p' serve.py
cat config.yaml
```

`train.py`, `serve.py`, and `config.yaml` do not require modification.

Only the TODO in `register.py` needs to be completed.

## 2. Training

`train.py` performs the following:

1. Loads `config.yaml`.
2. Configures SeaweedFS as the S3 endpoint.
3. Downloads `transactions.csv` from the `data` bucket.
4. Splits the dataset into training and test portions.
5. Trains a `RandomForestClassifier`.
6. Enables `mlflow.sklearn.autolog()`.
7. Creates/uses the `fraud-detection` MLflow experiment.
8. Logs the training run and model artifacts to MLflow.

Run:

```bash
cd /root/code
python3 train.py
```

Successful output:

```text
2026/09/07 00:08:17 INFO mlflow.tracking.fluent:
Experiment with name 'fraud-detection' does not exist. Creating a new experiment.

Logged MLflow run id=47341459c7704903a725fccd604f4436
```

The training run ID was:

```text
47341459c7704903a725fccd604f4436
```

The warnings about integer columns and scikit-learn model serialization were non-fatal. Training completed successfully.

## 3. Complete register.py

The purpose of `register.py` is to take the latest run from the `fraud-detection` experiment and register its model as `fraud-detector`.

Replace the TODO section with:

```python
model_uri = f"runs:/{run_id}/model"

registered = mlflow.register_model(
    model_uri=model_uri,
    name=MODEL_NAME,
)

client.set_registered_model_alias(
    name=MODEL_NAME,
    alias=ALIAS,
    version=registered.version,
)

print(f"[register] registered version={registered.version}")
print(f"[register] promoted {MODEL_NAME} version {registered.version} to @{ALIAS}")
```

The logic:

- Builds `runs:/<run_id>/model`.
- Registers the model as `fraud-detector`.
- Assigns the `production` alias.
- Prints the promoted model version.

## 4. Register and Promote

Run:

```bash
cd /root/code
python3 register.py
```

Successful output:

```text
[register] latest run_id=47341459c7704903a725fccd604f4436
Successfully registered model 'fraud-detector'.
Created version '1' of model 'fraud-detector'.
[register] registered version=1
[register] promoted fraud-detector version 1 to @production
```

Final registry state:

```text
Model: fraud-detector
Version: 1
Alias: production
```

The serving URI is:

```text
models:/fraud-detector@production
```

### MLflow Registration Warning

MLflow produced:

```text
Run with id 47341459c7704903a725fccd604f4436 has no artifacts at artifact path 'model',
registering model based on models:/m-9b9cb3e56adb4a3ebacdf45d93ff5878 instead
```

This was not an error.

MLflow autologging had logged the model using its newer model-ID mechanism. MLflow therefore resolved the logged model through:

```text
models:/m-9b9cb3e56adb4a3ebacdf45d93ff5878
```

The registration completed successfully and version `1` was assigned the `production` alias.

## 5. FastAPI Serving

`serve.py` is already complete and requires no changes.

It reads the MLflow configuration from `config.yaml` and constructs:

```text
models:/fraud-detector@production
```

The background loader repeatedly attempts to load the model:

```python
while _state["model"] is None:
    try:
        _state["model"] = mlflow.pyfunc.load_model(model_uri)
        print(f"[serve] loaded {model_uri}")
        return
    except Exception:
        time.sleep(5)
```

The loader retries every 5 seconds.

Therefore, the server can be started before the model exists or before the alias is assigned.

## 6. Start the API Server

Run:

```bash
cd /root/code
python3 serve.py
```

Expected startup output:

```text
INFO:     Started server process [...]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8085
```

After the production alias becomes available, the loader should print:

```text
[serve] loaded models:/fraud-detector@production
```

Keep this terminal running.

## 7. Check Server Health

Open another terminal and run:

```bash
curl http://localhost:8085/health
```

While the model is still loading:

```json
{
  "status": "loading",
  "model_uri": "models:/fraud-detector@production"
}
```

After successful loading:

```json
{
  "status": "healthy",
  "model_uri": "models:/fraud-detector@production"
}
```

## 8. Test Prediction

Run the required prediction request:

```bash
curl -X POST http://localhost:8085/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [100.5, 12, 3]}'
```

Expected response:

```json
{"prediction":0}
```

or:

```json
{"prediction":1}
```

Both values are valid for the lab.

## 9. Troubleshooting

### HTTP 503 from `/predict`

Example:

```json
{
  "detail": "model models:/fraud-detector@production is not yet available. Register the fraud-detection run as `fraud-detector` in the MLflow UI and assign the `production` alias."
}
```

This means the FastAPI server is running but the model has not loaded yet.

Check the registry:

```bash
python3 register.py
```

Then check:

```bash
curl http://localhost:8085/health
```

The serving process automatically retries every 5 seconds.

### Experiment does not exist

Run:

```bash
python3 train.py
```

Then:

```bash
python3 register.py
```

### Model artifact warning during registration

If MLflow reports:

```text
Created version '1' of model 'fraud-detector'.
```

and:

```text
[register] promoted fraud-detector version 1 to @production
```

then registration succeeded even if MLflow also reports that the run has no artifact at the legacy `model` path.

## 10. End-to-End Architecture

```text
SeaweedFS
    |
    | transactions.csv
    v
train.py
    |
    | RandomForest training
    v
MLflow Experiment: fraud-detection
    |
    | logged model + artifacts
    v
MLflow Model Registry
    |
    | fraud-detector version 1
    | production alias
    v
models:/fraud-detector@production
    |
    v
serve.py / FastAPI
    |
    | POST /predict
    v
{"prediction": 0 or 1}
```

## 11. Why the Production Alias Matters

The serving application does not directly reference a fixed version such as:

```text
models:/fraud-detector/1
```

Instead, it uses:

```text
models:/fraud-detector@production
```

This creates a stable production handle.

When a new model is trained, the process can:

1. Train the new model.
2. Register a new model version.
3. Move the `production` alias to the new version.

The serving application continues using:

```text
models:/fraud-detector@production
```

Therefore, promoting a new model is an alias change rather than a serving redeployment.

## 12. Final Verification Checklist

- [x] `fraud-detection` MLflow experiment created.
- [x] Training run completed.
- [x] Training run ID: `47341459c7704903a725fccd604f4436`
- [x] Model registered as `fraud-detector`.
- [x] Registered model version: `1`
- [x] `production` alias assigned to version `1`.
- [x] Serving URI: `models:/fraud-detector@production`
- [x] FastAPI server listening on port `8085`.
- [x] Background loader resolves the production alias.
- [x] `/health` reports `healthy`.
- [x] `/predict` accepts `{"features": [100.5, 12, 3]}`.
- [x] Prediction returns `0` or `1`.

## Final Commands

### Train

```bash
cd /root/code
python3 train.py
```

### Register

```bash
python3 register.py
```

### Serve

```bash
python3 serve.py
```

### Health Check

```bash
curl http://localhost:8085/health
```

### Prediction

```bash
curl -X POST http://localhost:8085/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [100.5, 12, 3]}'
```

## Result

The Day 97 capstone establishes the complete MLOps serving path:

**SeaweedFS dataset → MLflow training run → MLflow Model Registry → `production` alias → FastAPI model loader → live prediction endpoint.**

The production alias provides a stable handoff between model lifecycle management and the serving layer, allowing future model versions to be promoted without changing the serving application's model URI.

### Screenshots

````