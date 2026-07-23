# Day 60: Package a Model as a BentoML Service
The xFusionCorp Industries ML platform team has deployed the fraud-detection model using BentoML. The model is registered in BentoML's local store and is served over HTTP with the command bentoml serve, which automatically generates a Swagger UI at the server's root. Within the scaffold located at /root/code/serving/service.py, the modern @bentoml.service class API is utilized. This script loads the pre-registered fraud_detector:latest model from the store and defines the APIs, although the POST /predict handler remains unimplemented. Your objective is to implement the /predict handler to score a transaction using the loaded model. Additionally, you need to start the server on port 3000 and verify that it returns predictions.


The fraud_detector model is registered in BentoML's local store. The BentoML server is NOT pre-started — the BentoML UI button opens the Swagger surface once the server is running on port 3000.

The project layout under /root/code/serving/:

service.py – BentoML service (@bentoml.service class FraudService). The model-store load (bentoml.models.BentoModel + bentoml.sklearn.load_model in __init__) and the last_predictions API are wired. The predict handler body is left as a TODO — it returns an error until authored. The handler takes the amount/hour/num_tx_past_day parameters and returns {"is_fraud": <int>}.
train.csv – The 10-row source used at startup to train and register the model with bentoml.sklearn.save_model("fraud_detector", model).
The end state must include:

bentoml models list lists fraud_detector in the store.
curl http://localhost:3000/ returns HTTP 200 – The Swagger UI is reachable once the server is running.
POST /predict with a valid payload returns {"is_fraud": 0} or {"is_fraud": 1}.
Two distinct payloads return different is_fraud values – The handler scores the posted features.
Suggested payloads: {"amount": 3200, "hour": 23, "num_tx_past_day": 5} (high-value, late-night—expected to flag fraud); {"amount": 25.5, "hour": 10, "num_tx_past_day": 1} (low-value, daytime).

## Objective

Package and serve a pre-trained fraud detection model using **BentoML**. Implement the missing prediction handler, start the BentoML server, and verify that the API successfully returns predictions.

---

## Project Structure

```
/root/code/serving/
├── service.py
├── train.csv
```

- **service.py** – Defines the BentoML service using the modern `@bentoml.service` API.
- **train.csv** – Dataset used to train and register the `fraud_detector` model.

---

## Implementation

The `predict()` API was implemented to:

1. Create a feature vector from the request.
2. Run inference using the loaded model.
3. Store prediction history.
4. Return the prediction as JSON.

```python
@bentoml.api
def predict(
    self, amount: float, hour: int, num_tx_past_day: int
) -> Dict[str, Any]:
    # Build feature row
    features = np.array([[amount, hour, num_tx_past_day]])

    # Predict
    prediction = int(self.model.predict(features)[0])

    # Save request history
    self._history.append(
        {
            "amount": amount,
            "hour": hour,
            "num_tx_past_day": num_tx_past_day,
            "is_fraud": prediction,
        }
    )

    # Return response
    return {"is_fraud": prediction}
```

---

## Verify Model Registration

```bash
bentoml models list
```

Expected output:

```
Tag
fraud_detector:latest
```

---

## Start the BentoML Server

```bash
cd /root/code/serving

bentoml serve service:FraudService --port 3000
```

The server starts on **http://localhost:3000**.

---

## Verify Swagger UI

```bash
curl -I http://localhost:3000/
```

Expected:

```
HTTP/1.1 200 OK
```

---

## Test the Prediction API

### High-Risk Transaction

```bash
curl -X POST http://localhost:3000/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

Example response:

```json
{
  "is_fraud": 1
}
```

---

### Low-Risk Transaction

```bash
curl -X POST http://localhost:3000/predict \
-H "Content-Type: application/json" \
-d '{"amount":25.5,"hour":10,"num_tx_past_day":1}'
```

Example response:

```json
{
  "is_fraud": 0
}
```

---

## View Prediction History

```bash
curl -X POST http://localhost:3000/last_predictions
```

Example response:

```json
{
  "count": 2,
  "predictions": [
    {
      "amount": 3200,
      "hour": 23,
      "num_tx_past_day": 5,
      "is_fraud": 1
    },
    {
      "amount": 25.5,
      "hour": 10,
      "num_tx_past_day": 1,
      "is_fraud": 0
    }
  ]
}
```

---

## Outcome

- Successfully loaded the `fraud_detector:latest` model from the BentoML model store.
- Implemented the `POST /predict` endpoint.
- Served the model using BentoML on **port 3000**.
- Verified the Swagger UI was accessible.
- Confirmed that different inputs produced different fraud predictions.
- Recorded prediction history through the `last_predictions` endpoint.

---

## Skills Learned

- BentoML model loading
- BentoML Service API
- Model inference with scikit-learn
- Serving ML models over HTTP
- Building REST prediction endpoints
- Maintaining inference history
- API testing with `curl`

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/97c3c27b-d852-4e49-894e-9a6d7c543c61" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e0fe7dd0-7f01-44cf-8ac5-6a3e8a561717" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/198b7f47-74d7-42ae-8245-dc57784ad82d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/80b08d5b-57cf-4824-8adb-04607daf8cc0" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/92fc8b97-4a7d-4f5d-9513-d86363120655" />





