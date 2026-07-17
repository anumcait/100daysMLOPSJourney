# Day 56: Fix a Docker CI Pipeline with Git-SHA Tagging
The xFusionCorp Industries ML platform team operates a shell-based Docker CI pipeline for the fraud-detection Flask service. In this process, tests are executed, the image is built, a short git SHA is applied as the tag, and the tagged image is subsequently pushed to the local private registry. However, the pre-staged build.sh located at /root/code/ci/ does not currently execute cleanly from start to finish. Your objective is to rectify the configuration so that ./build.sh completes its execution without errors and that the registry catalog displays ml-ci-app tagged with the current git short SHA.


The Docker daemon is already running and a registry:2 container named local-registry is already up on host port 5555.

The repository layout under /root/code/ci/:

app/app.py – Flask service exposing /health + /predict on port 8086. Correct.
app/test_app.py – Three pytest cases covering /health, the fraud-flag flow, and the pass-through flow. Correct.
app/Dockerfile – python:3.11-slim, installs flask, COPYs app.py, exposes 8086, runs the Flask app. Correct.
app/.git/ – A local git repository initialised at startup with a single "Initial CI baseline" commit. Correct.
build.sh – Executable shell script with four stages (test → build → tag → push). Needs attention.
The end state must include:

./build.sh runs end-to-end without non-zero exit.
docker images ml-ci-app:latest lists the locally-built image.
curl http://localhost:5555/v2/_catalog lists ml-ci-app in the repositories array.
curl http://localhost:5555/v2/ml-ci-app/tags/list lists the current git -C app rev-parse --short HEAD value in the tags array.
Run ./build.sh against the scaffold as-is; each re-run surfaces the next blocker. All fixes live inside build.sh.

## Objective

The goal of this lab was to fix a shell-based Docker CI pipeline so that it executes successfully from start to finish. The pipeline performs the following stages:

1. Run application tests
2. Build the Docker image
3. Tag the image using the current Git short SHA
4. Push the tagged image to a local Docker registry

---

## Problem

The provided `build.sh` script failed due to multiple configuration issues:

- Incorrect pytest path
- Incorrect Docker registry port
- Incorrect variable name used for the Git SHA tag

---

## Root Cause

### 1. Incorrect Test Path

**Before**

```bash
python3 -m pytest app/tests/
```

The repository contains:

```text
app/test_app.py
```

**Fix**

```bash
python3 -m pytest app/test_app.py
```

---

### 2. Incorrect Registry Port

**Before**

```bash
REGISTRY="localhost:5000"
```

The local registry was already running on **port 5555**.

**Fix**

```bash
REGISTRY="localhost:5555"
```

---

### 3. Incorrect Variable Name

The script stored the Git SHA in:

```bash
SHA=$(git -C app rev-parse --short HEAD)
```

but later referenced a variable that didn't exist:

```bash
TAGGED="$REGISTRY/$IMAGE:$GIT_SHA"
```

**Fix**

```bash
TAGGED="$REGISTRY/$IMAGE:$SHA"
```

---

## Corrected `build.sh`

```bash
#!/bin/bash
set -euo pipefail

cd "$(dirname "$0")"

IMAGE="ml-ci-app"
REGISTRY="localhost:5555"

echo "[ci] stage 1/4 — running tests"
python3 -m pytest app/test_app.py

echo "[ci] stage 2/4 — building image"
docker build -t "$IMAGE:latest" app/

echo "[ci] stage 3/4 — tagging"
SHA=$(git -C app rev-parse --short HEAD)
TAGGED="$REGISTRY/$IMAGE:$SHA"
docker tag "$IMAGE:latest" "$TAGGED"

echo "[ci] stage 4/4 — pushing"
docker push "$TAGGED"

echo "[ci] complete: $TAGGED"
```

---

## Run the Pipeline

```bash
cd /root/code/ci
./build.sh
```

---

## Verification

### Verify Local Image

```bash
docker images ml-ci-app:latest
```

### Verify Registry Catalog

```bash
curl http://localhost:5555/v2/_catalog
```

Expected:

```json
{"repositories":["ml-ci-app"]}
```

### Verify Image Tags

```bash
curl http://localhost:5555/v2/ml-ci-app/tags/list
```

Expected:

```json
{"name":"ml-ci-app","tags":["<git-short-sha>"]}
```

### Verify Git Short SHA

```bash
git -C app rev-parse --short HEAD
```

The Git SHA returned by this command should match the tag shown in the registry.

---

## Files Modified

- `build.sh`

---

## Key Takeaways

- Always verify file paths used in CI scripts.
- Ensure registry endpoints match the running environment.
- Be consistent with variable names in shell scripts.
- Use `set -euo pipefail` to catch scripting errors early.
- Tagging Docker images with Git SHAs improves traceability in CI/CD pipelines.

### Screenshots

---
