# Day 57: Serve an ML Model with Flask

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





