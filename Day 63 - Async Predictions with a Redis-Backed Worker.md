# Day 63: Async Predictions with a Redis-Backed Worker
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

## Objective

Implemented Redis result persistence for an asynchronous Flask ML prediction service.

## Task

The application accepts prediction requests asynchronously:

- `POST /predict-async` returns a `task_id` immediately.
- A background worker performs the model inference.
- The worker stores the prediction result in Redis.
- `GET /result/<task_id>` retrieves the stored prediction.

## Changes Made

### 1. Configured Redis

```python
REDIS = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)
```

### 2. Stored Prediction Results

Implemented Redis persistence in the background worker.

```python
def _run_prediction(task_id: str, features) -> None:
    time.sleep(0.3)
    is_fraud = int(MODEL.predict(np.array([features]))[0])

    REDIS.set(
        RESULT_KEY.format(task_id=task_id),
        is_fraud,
        ex=RESULT_TTL_SECONDS,
    )
```

- Redis key format: `result:<task_id>`
- TTL: **600 seconds**

### 3. Implemented Result Lookup

Completed the `GET /result/<task_id>` endpoint.

```python
@app.route("/result/<task_id>")
def result(task_id):
    value = REDIS.get(RESULT_KEY.format(task_id=task_id))

    if value is None:
        return jsonify({
            "task_id": task_id,
            "status": "pending",
        }), 202

    return jsonify({
        "task_id": task_id,
        "is_fraud": int(value),
    }), 200
```

## API Workflow

### Submit Prediction

**Request**

```http
POST /predict-async
```

```json
{
  "amount": 100,
  "hour": 12,
  "num_tx_past_day": 3
}
```

**Response**

```json
{
  "task_id": "a1b2c3d4e5..."
}
```

---

### Poll for Result

**Request**

```http
GET /result/<task_id>
```

**While Processing**

```json
{
  "task_id": "a1b2c3d4e5...",
  "status": "pending"
}
```

**After Completion**

```json
{
  "task_id": "a1b2c3d4e5...",
  "is_fraud": 0
}
```

or

```json
{
  "task_id": "a1b2c3d4e5...",
  "is_fraud": 1
}
```

## Technologies Used

- Python
- Flask
- Redis
- redis-py
- NumPy
- Joblib
- Background Threads

## Key Learnings

- Implemented asynchronous request handling using background threads.
- Stored prediction results in Redis with expiration.
- Used Redis as a lightweight task result backend.
- Implemented polling for asynchronous task completion.
- Improved API responsiveness by decoupling request handling from model inference.

## Outcome

Successfully implemented an asynchronous prediction service where:

- Clients receive a `task_id` immediately.
- Predictions are processed in the background.
- Results are cached in Redis with a 600-second TTL.
- Clients can poll for prediction results using the provided `task_id`.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/80096eb9-2ffd-43cf-a0f0-62006d71d713" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e6f8c127-be87-4010-8c84-dc55d5ffc976" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/0de890fc-d53e-4f2e-a255-c46434284399" />

---


