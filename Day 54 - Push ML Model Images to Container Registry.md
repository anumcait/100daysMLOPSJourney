# Day 54: Push ML Model Images to Container Registry
The xFusionCorp Industries ML platform team has implemented a fraud-detection image and stored it in a private Docker registry for downstream clusters to access by tag. The registry is currently operational on host port 5555, and there exists a push.sh script located at /root/code/ml-registry/ that builds the image. However, the script does not yet include the steps necessary for publishing the image to the registry, which is marked as a TODO. Your objective is to complete the publishing flow within the script. Specifically, ensure that when you run ./push.sh, it successfully builds the image labeled fraud-detector:v1, tags it appropriately for the local registry, pushes the image, and confirms that the registry's HTTP catalogue responds with {"repositories":["fraud-detector"]}.


The Docker daemon is already running and a registry:2 container named local-registry is already up on host port 5555 (→ container port 5000).

The project layout under /root/code/ml-registry/:

train.py – Fits a tiny RandomForest and writes /app/model.pkl. Correct.
Dockerfile – python:3.11-slim base, installs sklearn + numpy + joblib, runs train.py at build time so the model is baked into the image. Correct.
push.sh – Executable shell script that docker builds fraud-detector:v1; the registry publish flow (naming the image for the registry and pushing it) is left as a TODO to author.
The end state must include:

docker images fraud-detector:v1 lists the locally-built image.
curl http://localhost:5555/v2/_catalog returns a JSON body with fraud-detector in the repositories array.
curl http://localhost:5555/v2/fraud-detector/tags/list returns a JSON body carrying v1 in the tags array.
docker tag only writes local metadata; nothing reaches the registry until docker push runs.

## Objective

Complete the publishing flow for an ML model Docker image by updating the `push.sh` script so that it:

- Builds the Docker image `fraud-detector:v1`
- Tags the image for the local private registry
- Pushes the image to the registry
- Verifies that the image is available in the registry

---

## Problem Statement

The ML platform team at **xFusionCorp Industries** has created a Docker image for a fraud detection model. A private Docker registry is already running on **localhost:5555**, but the `push.sh` script only builds the image and is missing the steps required to publish it.

The task is to update the script so that running:

```bash
./push.sh
```

successfully publishes the image to the registry.

---

## Existing `push.sh`

```bash
#!/bin/bash
set -euo pipefail

cd "$(dirname "$0")"

IMAGE="fraud-detector:v1"

docker build -t "$IMAGE" .

# TODO:
# Tag the image for the local registry
# Push the image to the registry
```

---

## Solution

Add the following commands after the `docker build` command:

```bash
docker tag "$IMAGE" localhost:5555/fraud-detector:v1
docker push localhost:5555/fraud-detector:v1
```

### Final `push.sh`

```bash
#!/bin/bash
set -euo pipefail

cd "$(dirname "$0")"

IMAGE="fraud-detector:v1"

docker build -t "$IMAGE" .

docker tag "$IMAGE" localhost:5555/fraud-detector:v1
docker push localhost:5555/fraud-detector:v1
```

---

## Verification

### Check the local image

```bash
docker images fraud-detector:v1
```

Expected output should include:

```text
fraud-detector    v1
```

---

### Verify the registry catalog

```bash
curl http://localhost:5555/v2/_catalog
```

Expected:

```json
{"repositories":["fraud-detector"]}
```

---

### Verify available tags

```bash
curl http://localhost:5555/v2/fraud-detector/tags/list
```

Expected:

```json
{"name":"fraud-detector","tags":["v1"]}
```

---

## Key Takeaways

- `docker build` creates the image locally.
- `docker tag` creates a registry-qualified image name.
- `docker push` uploads the image to the registry.
- An image does **not** appear in a registry until `docker push` is executed.
- The Docker Registry HTTP API can be used to verify repositories and tags.

---

## Commands Used

```bash
docker build -t fraud-detector:v1 .
docker tag fraud-detector:v1 localhost:5555/fraud-detector:v1
docker push localhost:5555/fraud-detector:v1

docker images fraud-detector:v1

curl http://localhost:5555/v2/_catalog
curl http://localhost:5555/v2/fraud-detector/tags/list
```

---

## Outcome

✅ Successfully built the Docker image.

✅ Tagged it for the local private registry.

✅ Pushed the image to the registry.

✅ Verified the repository and tag using the Docker Registry HTTP API.

### Screenshots
