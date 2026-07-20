# Day 58: Serve an ML Model with FastAPI
The xFusionCorp Industries ML platform team has developed a fraud-detection model that is accessible via a FastAPI service. This service features an auto-generated Swagger UI available at /docs. In the draft app.py file located at /root/code/serving/, the pre-trained model is loaded and the /health endpoint is implemented. However, the PredictRequest schema and the POST /predict handler have not yet been completed.

Your task is to finalize the app.py file by defining the typed request model and implementing the /predict handler. Additionally, ensure the server operates on port 8085, and verify that it successfully scores transactions through the Swagger UI while rejecting any invalid input.


FastAPI + uvicorn are baked into the lab image. The FastAPI Swagger UI button opens /docs once the server is running on port 8085.

The project layout under /root/code/serving/:

model.pkl – Deterministic RandomForest trained at startup on the shared amount / hour / num_tx_past_day → is_fraud synthetic dataset.
train.csv – The 10-row source used to fit the model.
app.py – FastAPI app. /health, the root→/docs redirect, the response models, and GET /last-predictions are wired. Two things are left as TODOs to author:
TODO 1 – the PredictRequest model's three typed fields (amount, hour, num_tx_past_day), with types and range constraints that drive FastAPI's request validation + Swagger schema.
TODO 2 – the POST /predict handler body (which returns 501 until authored): build the feature row, score it with MODEL.predict(...), record it, and return PredictResponse.
The end state must include:

The Swagger UI at /docs is reachable (the FastAPI Swagger UI button loads it).
POST /predict with a valid payload returns {"is_fraud": 0} or {"is_fraud": 1} (HTTP 200).
Two distinct payloads return different is_fraud values – The handler reads the posted features.
POST /predict with an out-of-range field (e.g. hour: 25) returns HTTP 422 – The typed model rejects invalid input before the handler runs.
Suggested payloads: {"amount": 3200, "hour": 23, "num_tx_past_day": 5} (high-value, late-night—expected to flag fraud); {"amount": 25.5, "hour": 10, "num_tx_past_day": 1} (low-value, daytime—expected to pass).

## Objective

Built and completed a FastAPI-based machine learning inference service for a fraud detection model by implementing request validation, prediction logic, and model serving with Swagger UI support.

## Tasks Completed

- Loaded the pre-trained RandomForest model from `model.pkl`.
- Implemented the `PredictRequest` schema using Pydantic with type validation.
- Added field constraints using `Field()`:
  - `amount` → `float`, non-negative (`ge=0`)
  - `hour` → `int`, range `0-23`
  - `num_tx_past_day` → `int`, non-negative (`ge=0`)
- Implemented the `POST /predict` endpoint.
- Created the feature array using NumPy.
- Performed inference using `MODEL.predict()`.
- Returned prediction as a `PredictResponse`.
- Logged each prediction to the in-memory prediction history.
- Verified the API through the auto-generated Swagger UI.
- Configured the application to run on **port 8085**.

## Tech Stack

- Python
- FastAPI
- Pydantic
- NumPy
- Joblib
- Scikit-learn
- Uvicorn

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| POST | `/predict` | Predict whether a transaction is fraudulent |
| GET | `/last-predictions` | View prediction history |
| GET | `/docs` | Interactive Swagger UI |

## Sample Requests

### Fraudulent Transaction

```json
{
  "amount": 3200,
  "hour": 23,
  "num_tx_past_day": 5
}
```

**Response**

```json
{
  "is_fraud": 1
}
```

### Legitimate Transaction

```json
{
  "amount": 25.5,
  "hour": 10,
  "num_tx_past_day": 1
}
```

**Response**

```json
{
  "is_fraud": 0
}
```

### Invalid Request

```json
{
  "amount": 100,
  "hour": 25,
  "num_tx_past_day": 1
}
```

**Response**

```
HTTP 422 Unprocessable Entity
```

## Key Learning Outcomes

- Building REST APIs with FastAPI
- Request validation using Pydantic models
- Serving machine learning models with FastAPI
- Performing inference with Scikit-learn models
- Maintaining prediction history
- Using Swagger UI for API testing
- Input validation through typed schemas

## Outcome

Successfully deployed a FastAPI inference service capable of validating incoming requests, serving predictions from a trained fraud detection model, recording prediction history, and exposing interactive API documentation through Swagger UI.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/465c1263-94e0-4af2-8d30-6c5094a9ba27" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9c29579f-8d42-46df-a58d-8a10bb92c530" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/722202bb-7eb5-4ccf-b189-afb859f6f84d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ec4181ca-7022-4255-a61b-7f9a76252db8" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8f72e691-0397-42c6-91a0-c8e3b628c46c" />






