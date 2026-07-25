### Day 62: Implement A/B Testing for Model Deployment
The xFusionCorp Industries ML platform team operates an asynchronous fraud-detection scoring system, ensuring that the HTTP entry point responds within single-digit milliseconds while the model processes data in a background worker. The scaffold for this process, located at /root/code/serving/async_app.py, is designed to delegate tasks to a background worker and is intended to persist the results of each task in Redis. However, the implementation for storing results in Redis has not yet been completed.

Your objective is to implement the Redis round-trip within async_app.py. This involves storing each result in Redis after the worker has completed its task. In addition, you must ensure that the GET /result/<task_id> endpoint retrieves the stored results. The expected workflow is for clients to submit a request through POST /predict-async, then to subsequently poll the results using GET /result/<task_id>, which should return an is_fraud flag corresponding to the submitted payload.


Flask + redis-py are installed at startup. A Redis container named async-redis is already running on host port 6379.

The project layout under /root/code/serving/:

model.pkl – Deterministic RandomForest trained at startup.
async_app.py – Flask app. The Redis connection, /health, POST /predict-async (returns a task_id, runs the model on a background thread), and the thread itself are wired. Two things are left as TODOs to author: the worker's result store in Redis, and the GET /result/<task_id> lookup that reads it back.
The end state must include:

redis.Redis(host="localhost", port=6379, ...) in async_app.py.
GET /result/<task_id> reads the stored value back from Redis and returns it as part of the JSON body.
POST /predict-async returns a JSON body carrying a task_id; after a short poll, GET /result/<task_id> returns a JSON body carrying an is_fraud flag of 0 or 1.
The background worker stores results at keys shaped result:<task_id>, with a 600-second TTL.

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

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1e4b7b37-6c9c-48a1-93c1-391a75d27394" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8376233d-ebc2-4705-b7ac-6294ec9a1903" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d03fbb34-0fd4-47e5-929b-8e88278cd85d" />




