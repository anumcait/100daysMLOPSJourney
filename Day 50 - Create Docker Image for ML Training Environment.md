# Day 50: Create Docker Image for ML Training Environment

## Objective

Create a Docker image for the ML fraud-detection training environment.

The Dockerfile must:

- Use `python:3.11-slim` as the base image.
- Set the working directory to `/app`.
- Install the required Python packages:
  - scikit-learn
  - pandas
  - numpy
  - joblib
- Copy `train.py` into the image.
- Run `python3 train.py` by default.

---

## Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir scikit-learn pandas numpy joblib

COPY train.py /app/train.py

CMD ["python3", "train.py"]
```

---

## Build the Image

```bash
cd /root/code/ml-docker

docker build -t ml-trainer:v1 .
```

---

## Verify the Image

```bash
docker images ml-trainer:v1
```

Expected output should list:

```
REPOSITORY    TAG    IMAGE ID    CREATED    SIZE
ml-trainer    v1     <image-id>  <time>     <size>
```

---

## Verify Required Python Imports

```bash
docker run --rm ml-trainer:v1 python3 -c "import sklearn, pandas, numpy, joblib; print('OK')"
```

Expected output:

```text
OK
```

---

## Verify Training Execution

```bash
docker run --rm ml-trainer:v1
```

The container should execute `train.py`, train the sample model, print the evaluation metrics, and save the model as:

```
/app/model.pkl
```

---

## Result

Successfully created the Docker image **ml-trainer:v1** with:

- Python 3.11 Slim base image
- `/app` working directory
- Required ML dependencies installed
- `train.py` copied into the image
- Default command configured to run the training script
- All required Python imports resolved successfully

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/cbb0b380-2c88-4628-958b-5b037ad3da61" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/42d261b5-e56b-497e-b720-5280543751d1" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/092caf9c-def5-401c-baff-055a56a38618" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2e1f87e7-c532-4355-af5a-762c0683c21f" />




