# Day 57: Serve an ML Model with Flask
The xFusionCorp Industries ML platform team serves the fraud-detection model over HTTP with a Flask app so downstream services can score transactions synchronously. A draft app.py at /root/code/serving/ loads the pre-trained model and declares /health + /predict, but the server binds to the wrong port and the /predict handler is left unimplemented. Your task is to author the /predict handler and bind the server to the exposed port, start the server, and confirm two distinct payloads produce two distinct responses.


Flask is installed at startup (it is not part of the lab image by default).

The project layout under /root/code/serving/:

model.pkl – Deterministic RandomForest trained at startup on the shared amount / hour / num_tx_past_day → is_fraud synthetic set. Correct and must remain intact.
train.csv – The 10-row training source used to fit model.pkl.
app.py – Flask app loading model.pkl and declaring GET /health + POST /predict. The /predict handler body is left as a TODO to author, and the app.run(...) bind port needs attention.
The end state must include:

curl http://localhost:8085/health returns {"status":"ok"} with HTTP 200.
curl -X POST http://localhost:8085/predict -H 'Content-Type: application/json' -d '{"amount":3200,"hour":23,"num_tx_past_day":5}' returns a JSON body with an is_fraud key.
The same POST with a low-amount daytime payload (e.g. {"amount":25.5,"hour":10,"num_tx_past_day":1}) returns a different is_fraud value – Confirming the endpoint actually reads the posted body rather than falling back to zeros.
The lab's port forwarding targets 8085; the running server must bind there to be reachable.

## 📌 Objective

Implement and serve a pre-trained fraud detection machine learning model using **Flask**.

---

## 🛠️ Tasks Completed

- Loaded the pre-trained `RandomForest` model (`model.pkl`).
- Implemented the `POST /predict` endpoint.
- Parsed incoming JSON requests.
- Created the feature array using:
  ```python
  [[amount, hour, num_tx_past_day]]
  ```
- Generated predictions using `MODEL.predict()`.
- Returned the prediction as a JSON response.
- Updated the Flask application to run on **port 8085**.
- Verified the `/health` endpoint.
- Tested the API using different request payloads.

---

## 📁 Project Structure

```text
/root/code/serving/
├── app.py
├── model.pkl
└── train.csv
```

---

## 🚀 Final `/predict` Endpoint

```python
@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()

    features = np.array([[
        data["amount"],
        data["hour"],
        data["num_tx_past_day"]
    ]])

    prediction = int(MODEL.predict(features)[0])

    return jsonify({"is_fraud": prediction}), 200
```

---

## 🌐 Running the Flask Server

```python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8085)
```

Start the server:

```bash
cd /root/code/serving
python3 app.py
```

---

## 📡 API Endpoints

### Health Check

**Request**

```http
GET /health
```

**Response**

```json
{
  "status": "ok"
}
```

---

### Prediction Endpoint

**Request**

```http
POST /predict
Content-Type: application/json
```

**Sample Request**

```json
{
  "amount": 3200,
  "hour": 23,
  "num_tx_past_day": 5
}
```

**Sample Response**

```json
{
  "is_fraud": 1
}
```

---

## 🧪 Testing the API

### Health Check

```bash
curl http://localhost:8085/health
```

### High-Risk Transaction

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

### Low-Risk Transaction

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":25.5,"hour":10,"num_tx_past_day":1}'
```

The responses should return different `is_fraud` values, confirming that the model is processing the provided request data.

---

## 📚 Key Learnings

- Building REST APIs with Flask.
- Loading a pre-trained machine learning model using Joblib.
- Processing JSON requests in Flask.
- Preparing model input using NumPy.
- Returning predictions as JSON.
- Running Flask on a custom port.
- Testing APIs using `curl`.

---

## ✅ Outcome

Successfully deployed a Flask-based prediction API for a fraud detection model, exposed health and prediction endpoints, verified model inference with multiple payloads, and configured the application to run on the required **8085** port.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/fbbc20a6-8b6e-4760-a0c6-c7086fc8a617" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2f475f5f-e677-42e2-9fcd-c232678b28ac" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/13051ebc-c442-4199-9ff2-a2555aa8728c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e20a5eed-b5a0-412e-9435-9e84e9350e7f" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b5dc00ac-8d50-461c-ae51-35a6078ee26c" />





