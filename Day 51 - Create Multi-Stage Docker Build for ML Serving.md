# Day 51: Create Multi-Stage Docker Build for ML Serving
The xFusionCorp Industries ML platform team has deployed a fraud-detection model as a Docker image. However, the current runtime image includes all packages required for the training phase and the training source itself, resulting in an unnecessarily large image. Your objective is to refactor the single-stage Dockerfile located at /root/code/ml-serve/ into a multi-stage build. This should comprise a builder stage that trains the model and generates model.pkl, followed by a runtime stage that installs only the dependencies necessary for serving and copies the trained model from the builder stage.


The Docker daemon is already running. docker version can be run in a VS Code terminal to confirm.

The project layout under /root/code/ml-serve/:

train_model.py – Fits a 10-tree RandomForest on the shared 10-row synthetic fraud set and writes /app/model.pkl via joblib.dump(...). Correct and must remain intact.
serve.py – Flask app loading the model and exposing POST /predict + GET /health on port 8080. Correct and must remain intact.
Dockerfile – A single-stage build that installs scikit-learn, pandas, numpy, joblib, and flask, runs the trainer at build time to bake the model in, and serves. The reader rewrites this file.
The end state must include:

The Dockerfile carries at least two FROM instructions; the first is given a name (e.g. AS builder) so a later stage can reference it.
The builder stage produces /app/model.pkl (the trained model).
The runtime stage contains /app/model.pkl (copied out of the builder stage) and serve.py.
The runtime stage's pip install line installs only the four packages serve.py needs: flask, joblib, numpy, scikit-learn.
docker images ml-serve:v1 lists the built image; docker run --rm -p 8090:8080 ml-serve:v1 exposes /health returning {"status": "ok"} on port 8090.
Multi-stage builds let you ship runtime images that carry only what the serving app needs — training dependencies and source files stay in the builder stage and are discarded. docker build -t ml-serve:v1 . can be re-run as each change lands; Docker re-uses cached layers when only runtime-stage lines change.

## Objective

Refactor a single-stage Dockerfile into a multi-stage Docker build for an ML serving application. The builder stage trains the model and generates `model.pkl`, while the runtime stage contains only the files and dependencies required to serve predictions.

---

## Project Structure

```
ml-serve/
├── Dockerfile
├── train_model.py
└── serve.py
```

- **train_model.py**
  - Trains a 10-tree `RandomForestClassifier`
  - Saves the trained model as `/app/model.pkl`

- **serve.py**
  - Loads `model.pkl`
  - Exposes:
    - `GET /health`
    - `POST /predict`
  - Runs on port **8080**

---

## Original Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir scikit-learn pandas numpy joblib flask

COPY train_model.py /app/train_model.py
COPY serve.py /app/serve.py

RUN python3 /app/train_model.py

EXPOSE 8080

CMD ["python3", "/app/serve.py"]
```

### Problems

- Single-stage build
- Runtime image contains training code
- Runtime installs unnecessary packages (`pandas`)
- Larger image than required

---

## Solution: Multi-Stage Docker Build

```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /app

COPY train_model.py .

RUN pip install --no-cache-dir \
    scikit-learn \
    pandas \
    numpy \
    joblib

RUN python3 train_model.py

FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir \
    flask \
    joblib \
    numpy \
    scikit-learn

COPY serve.py .
COPY --from=builder /app/model.pkl .

EXPOSE 8080

CMD ["python3", "serve.py"]
```

---

## Build the Image

```bash
cd /root/code/ml-serve

docker build -t ml-serve:v1 .
```

Verify:

```bash
docker images ml-serve:v1
```

---

## Run the Container

```bash
docker run --rm -p 8090:8080 ml-serve:v1
```

Expected output:

```
 * Serving Flask app 'serve'
 * Running on http://127.0.0.1:8080
```

---

## Test the Health Endpoint

```bash
curl http://localhost:8090/health
```

Output:

```json
{"status":"ok"}
```

---

## Verify Runtime Image

List application files:

```bash
docker run --rm ml-serve:v1 ls /app
```

Expected:

```
model.pkl
serve.py
```

`train_model.py` should **not** exist in the runtime image.

---

## Validation Checklist

- [x] Uses two `FROM` instructions
- [x] Builder stage named (`AS builder`)
- [x] Model trained during build
- [x] `model.pkl` copied from builder stage
- [x] Runtime contains only:
  - `serve.py`
  - `model.pkl`
- [x] Runtime installs only:
  - `flask`
  - `joblib`
  - `numpy`
  - `scikit-learn`
- [x] Container exposes port `8080`
- [x] `/health` returns:

```json
{"status":"ok"}
```

---

## Key Learnings

- Multi-stage Docker builds separate build-time and runtime environments.
- Builder stages can compile artifacts, train models, or build binaries.
- Final images should include only runtime dependencies.
- Smaller images improve security, reduce storage, and speed up deployments.
- Docker layer caching speeds up rebuilds when only runtime-stage changes occur.

---

## Commands Used

```bash
docker build -t ml-serve:v1 .

docker images ml-serve:v1

docker run --rm -p 8090:8080 ml-serve:v1

curl http://localhost:8090/health

docker ps

docker stop <container_id>
```

---

## Outcome

Successfully converted the ML serving application from a single-stage Docker build to a multi-stage build. The final runtime image contains only the serving application, the trained model, and the minimal dependencies required for inference, resulting in a cleaner and more production-ready container.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1d6e0b9f-5d6e-4210-add4-334bf94550e5" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b96ae32d-a40e-4ca3-bf14-54278996949e" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/bb0f5a16-4644-46d2-b3fd-979e595dd765" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/f7cb25b2-07c3-426e-8010-62368404a82c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d9937509-e3ce-401f-a3b5-f860dde3a05d" />





