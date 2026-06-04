# Day 19: Build Complete DVC ML Pipeline with Remote Storage and Experiments
Complete the xFusionCorp Industries fraud-detection production DVC pipeline. Three stages are already wired in dvc.yaml, two remain, and the pipeline must finish as a reproducible, SeaweedFS-backed, v1.0-tagged release.


A project exists at /root/code/ml-pipeline/ with Git and DVC initialised. The params.yaml is in place and the .dvc/config is pre-configured to push to the SeaweedFS bucket dvc-storage at http://localhost:8333.

The ingest, validate, and preprocess stages are already declared in dvc.yaml, but one of them contains an incorrect output path that prevents dvc repro from completing. Find and fix it.

The remaining two stages need to be added:

train – Depends on the preprocessed dataset and scripts/train.py; reads n_estimators, max_depth, test_size, and random_seed from params.yaml; outputs models/model.pkl and data/processed/test_split.csv; declares metrics.json as a DVC metric with cache: false.
evaluate – Depends on models/model.pkl, data/processed/test_split.csv, and scripts/evaluate.py; outputs reports/evaluation.json declared with cache: false.
The two scripts you need are pre-staged at /root/code/ml-pipeline/scripts-staging/train.py and scripts-staging/evaluate.py. Copy them into scripts/ before adding the stages.

Run the full pipeline with dvc repro, push the cache to the SeaweedFS remote with dvc push, and tag the current state as v1.0.

Commit every change to Git so the release is fully captured.

Open the SeaweedFS Filer button at the top of the lab and navigate to /buckets/dvc-storage/ to confirm that the bucket holds the pushed artefacts under the files/md5/... layout.

## Objective
Complete the xFusionCorp Industries fraud-detection production DVC pipeline by adding the remaining stages, fixing existing configuration errors, and ensuring a reproducible, SeaweedFS-backed, v1.0-tagged release.

## Task Overview
- **Project Structure**: A project at `ml-pipeline/` with Git and DVC initialized.
- **Remote Storage**: SeaweedFS bucket `dvc-storage` at `http://localhost:8333`.
- **Existing Stages**: `ingest`, `validate`, and `preprocess` (one has an incorrect output path).
- **New Stages**: `train` and `evaluate`.

## Steps to Complete

### 1. Fix Existing Pipeline
Identify and fix the incorrect output path in `dvc.yaml` for the `ingest`, `validate`, or `preprocess` stages to allow `dvc repro` to run correctly.

### 2. Prepare Scripts
Copy the pre-staged scripts from `scripts-staging/` to `scripts/`:
- `scripts-staging/train.py` -> `scripts/train.py`
- `scripts-staging/evaluate.py` -> `scripts/evaluate.py`

### 3. Add Train Stage
Add the `train` stage to `dvc.yaml`:
- **Dependencies**: Preprocessed dataset and `scripts/train.py`.
- **Parameters**: `n_estimators`, `max_depth`, `test_size`, and `random_seed` from `params.yaml`.
- **Outputs**: `models/model.pkl` and `data/processed/test_split.csv`.
- **Metrics**: `metrics.json` (with `cache: false`).

### 4. Add Evaluate Stage
Add the `evaluate` stage to `dvc.yaml`:
- **Dependencies**: `models/model.pkl`, `data/processed/test_split.csv`, and `scripts/evaluate.py`.
- **Outputs**: `reports/evaluation.json` (with `cache: false`).

### 5. Execute and Push
- Run the full pipeline: `dvc repro`.
- Push the cache to SeaweedFS: `dvc push`.
- Tag the release: `git tag -a v1.0 -m "Release v1.0: Complete ML Pipeline"`.

### 6. Verify Remote Storage
Confirm the artefacts are pushed to the SeaweedFS Filer under `/buckets/dvc-storage/` in the `files/md5/...` layout.

## Summary
By the end of this day, the fraud-detection pipeline will be fully automated, tracked, and versioned, providing a solid foundation for further experiments and production deployment.
