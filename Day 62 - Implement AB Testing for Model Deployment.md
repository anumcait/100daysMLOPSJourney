# Day 62: Implement A/B Testing for Model Deployment
The xFusionCorp Industries ML platform team has deployed a new fraud-detection model into production, utilizing an A/B router to manage traffic. The traffic distribution is set at 80% for the stable MODEL_V1 and 20% for the candidate MODEL_V2. Each response from the server includes a model_version field, enabling downstream monitoring to accurately attribute each prediction to the corresponding model. The ab_server.py scaffold, located at /root/code/serving/, is responsible for loading both models and parsing incoming requests; however, the routing logic has yet to be implemented. Your task is to develop the A/B routing functionality in ab_server.py, ensuring that approximately 80% of traffic is directed to MODEL_V1, 20% to MODEL_V2, and that all responses correctly indicate which model provided the prediction.


Flask is installed at startup (not part of the lab image by default). Two model versions are pre-trained: model_v1.pkl (10-tree RandomForest) and model_v2.pkl (50-tree RandomForest). Both live under /root/code/serving/.

The project layout under /root/code/serving/:

model_v1.pkl + model_v2.pkl – The two model versions the router multiplexes between. Correct.
ab_server.py – Flask app. /health, both model loads, and the request-body parsing in POST /predict are wired; the routing logic (split, model selection, response) is left as a TODO to author.
The end state must include:

ab_server.py splits traffic 80 % to MODEL_V1 and 20 % to MODEL_V2.
Every response to POST /predict carries both is_fraud and model_version; model_version is "v1" or "v2".
Over a batch of 200 requests, roughly 160 land on v1 (±20) and roughly 40 land on v2 (±20).
Flask reads the JSON body via request.get_json(); the scaffold already handles this.

## Overview

Today I implemented A/B testing routing for the xFusionCorp Industries fraud-detection ML serving platform.

The goal was to deploy two versions of a fraud-detection model and distribute incoming prediction traffic between them:

- **MODEL_V1** → Stable production model (80% traffic)
- **MODEL_V2** → Candidate model for testing (20% traffic)

Each prediction response includes the model version used, allowing downstream monitoring to compare model performance.

---

## Project Structure

```
/root/code/serving/
│
├── model_v1.pkl       # Stable RandomForest model (10 trees)
├── model_v2.pkl       # Candidate RandomForest model (50 trees)
└── ab_server.py       # Flask A/B testing server
```

---

## Implementation Details

The Flask server was already configured to:

- Load both model files.
- Expose `/health` endpoint.
- Parse incoming JSON requests using `request.get_json()`.

The missing part was the A/B routing logic inside the `/predict` endpoint.

---

## A/B Routing Logic

Traffic splitting was implemented using `random.random()`:

```python
if random.random() < 0.8:
    model = MODEL_V1
    model_version = "v1"
else:
    model = MODEL_V2
    model_version = "v2"
```

### Traffic Distribution

| Model | Traffic Percentage |
|------|--------------------|
| MODEL_V1 | 80% |
| MODEL_V2 | 20% |

Over 200 requests, the expected distribution is approximately:

- MODEL_V1 → ~160 requests
- MODEL_V2 → ~40 requests

---

## Prediction Response

Every `/predict` response now contains:

```json
{
    "is_fraud": 0,
    "model_version": "v1"
}
```

or

```json
{
    "is_fraud": 1,
    "model_version": "v2"
}
```

The `model_version` field allows monitoring systems to identify which model generated each prediction.

---

## Testing

### Start the Flask Server

```bash
cd /root/code/serving
python3 ab_server.py
```

Server runs on:

```
http://0.0.0.0:8085
```

---

### Health Check

Command:

```bash
curl http://localhost:8085/health
```

Response:

```json
{
    "status": "ok"
}
```

---

### Prediction Test

Command:

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":100.5,"hour":12,"num_tx_past_day":3}'
```

Example response:

```json
{
    "is_fraud": 0,
    "model_version": "v1"
}
```

---

## Final Outcome

✅ Implemented A/B model routing  
✅ Achieved 80/20 traffic split  
✅ Added model attribution using `model_version`  
✅ Enabled downstream monitoring of model predictions  
✅ Completed production-style ML model deployment testing

---

## Key Learning

A/B testing in ML deployment allows teams to safely introduce new models by gradually exposing real traffic to candidate versions while measuring performance against the stable production model.

### Screenshots
