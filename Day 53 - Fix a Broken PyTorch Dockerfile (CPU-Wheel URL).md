# Day 53: Fix a Broken PyTorch Dockerfile (CPU-Wheel URL)
The xFusionCorp Industries ML platform team has provided the PyTorch deep-learning images under the tag dl-trainer:v1. Each lab host is equipped with CPU-only capabilities, necessitating that the Dockerfile targets the CPU wheel index and that the container's default command operates effectively on hardware without a GPU. The current draft of the Dockerfile, located at /root/code/dl-docker/, does not fulfill these specifications; attempts to execute docker build are unsuccessful, and once an image is generated, the container fails to run upon startup. Your objective is to revise the Dockerfile to ensure that the command docker build -t dl-trainer:v1 . executes successfully and that the command docker run --rm dl-trainer:v1 outputs the installed torch version alongside the message cuda? False.


The Docker daemon is already running. docker version can be run in a VS Code terminal to confirm. The lab host does not expose a GPU—nvidia-smi returns command not found and torch.cuda.is_available() returns False inside any CPU-only container. Run docker build -t dl-trainer:v1 . in /root/code/dl-docker/ to see the build fail against the draft.

The project layout under /root/code/dl-docker/:

Dockerfile – A FROM, a WORKDIR, a RUN pip install line targeting torch, and a CMD that probes torch.cuda.
The end state must include:

docker images dl-trainer:v1 lists the built image.
docker run --rm dl-trainer:v1 exits 0 and prints the installed torch version alongside the CUDA flag (e.g. 2.5.0+cpu cuda? False).

## Objective

Fixed the Dockerfile for a PyTorch CPU-only container by:

- Replacing the incorrect GPU wheel index with the CPU wheel index.
- Updating the container's default command so it works correctly on systems without a GPU.
- Successfully building and running the Docker image.

---

## Problem

The provided Dockerfile had two issues:

1. It attempted to install PyTorch using the GPU wheel index:
   `https://download.pytorch.org/whl/gpu`
2. It asserted that CUDA must be available, causing the container to fail on CPU-only systems.

---

## Original Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir \
    --index-url https://download.pytorch.org/whl/gpu \
    torch

CMD ["python3", "-c", "import torch; assert torch.cuda.is_available(), 'CUDA required'"]
```

---

## Fixed Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir \
    --index-url https://download.pytorch.org/whl/cpu \
    torch

CMD ["python3", "-c", "import torch; print(torch.__version__, 'cuda?', torch.cuda.is_available())"]
```

---

## Build the Image

```bash
cd /root/code/dl-docker
docker build -t dl-trainer:v1 .
```

---

## Verify the Image

```bash
docker images dl-trainer:v1
```

---

## Run the Container

```bash
docker run --rm dl-trainer:v1
```

---

## Expected Output

```text
2.x.x+cpu cuda? False
```

> The exact PyTorch version may differ depending on the latest CPU wheel available.

---

## Key Learnings

- Use the PyTorch CPU wheel index for CPU-only environments:
  - `https://download.pytorch.org/whl/cpu`
- Avoid hardcoding CUDA requirements in containers intended for CPU systems.
- `torch.cuda.is_available()` returns `False` when CUDA is unavailable.
- A simple startup command can be used to verify both the installed PyTorch version and CUDA availability.

---

## Result

- ✅ Docker image built successfully.
- ✅ Image available as `dl-trainer:v1`.
- ✅ Container executed successfully.
- ✅ Output displayed the installed PyTorch version and `cuda? False`.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2ee9c34d-61db-412b-9cbc-8f601767211a" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d5c64b9e-f23b-426c-b759-f243f3da667d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8644f1a9-3ac3-49d3-86a6-0ecebc2955ea" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/9f625111-3ab6-4a8e-a593-9fd091187d15" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d57fb2ca-0c26-4e46-9a3c-400bdef5eedb" />





