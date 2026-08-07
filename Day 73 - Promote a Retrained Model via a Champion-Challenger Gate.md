# Day 73: Promote a Retrained Model via a Champion/Challenger Gate
The xFusionCorp Industries ML platform team operates a drift-triggered retraining pipeline. When data drift exceeds the alert threshold, the pipeline automatically retrains the fraud-detection model and registers it as a new version of fraud-detector. Currently, there is an incumbent version 1 serving production traffic under the production alias, along with a newly retrained challenger version 2, which has not yet been promoted. It is essential to avoid promoting the retrained model to production without evaluation, as this could result in a subpar model reaching users. Your task is to implement the promotion gate in promote.py, ensuring that the challenger is only promoted to production if it surpasses the incumbent's performance on the designated evaluation metric. After completing this task, proceed to execute the promotion.


The MLflow tracking server is on port 5000; the MLflow UI button opens it. Under Models → fraud-detector you can see version 1 (alias production, f1_score 0.71) and version 2 (no alias yet, f1_score 0.82).

The project is at /root/code/monitoring/:

retrain_pipeline.py – the startup setup that registered v1 and v2. Correct; do not edit.
promote.py – the promotion-gate scaffold. Its f1_of(version) helper (reads a version's logged f1_score) is wired; the gate logic is a # TODO naming the MLflow client calls to use.
The gate must read the version the production alias currently points at (the champion) and its f1_score, read the challenger's f1_score (version 2), and re-point production at the challenger only if the challenger's is strictly greater—otherwise leave the alias unchanged.

The end state must include:

The production alias on fraud-detector resolves to version 2 (the challenger).
The version now serving production has a higher f1_score than the incumbent v1 — the gate promoted the better model, not a blind version bump.
MLflow 3.x replaced stage-based promotion (Staging/Production) with aliases — named pointers like production that decouple a label from a version, so downstream code loads models:/fraud-detector@production and promotion is just re-pointing the alias. A champion/challenger gate is the guardrail: never re-point production at a retrained model without first confirming it actually beats what is live.

## Objective

Implement a **champion/challenger promotion gate** for an MLflow Model Registry workflow. The goal is to ensure that a newly retrained model is promoted to production **only if it outperforms the currently deployed model** based on the `f1_score` evaluation metric.

---

## Scenario

The retraining pipeline automatically registers a new version of the `fraud-detector` model whenever data drift exceeds the configured threshold.

Current model registry state:

| Version | Alias | F1 Score |
|---------|-------|----------|
| v1 | `production` | **0.71** |
| v2 | *(none)* | **0.82** |

Instead of blindly promoting the latest version, a **champion/challenger gate** compares the performance of the currently deployed model (champion) against the newly retrained model (challenger).

If:

```text
challenger_f1 > champion_f1
```

then the `production` alias is moved to the challenger.

Otherwise, the existing production model continues serving traffic.

---

## Project Structure

```text
/root/code/monitoring/
├── retrain_pipeline.py
└── promote.py
```

- `retrain_pipeline.py` registers model versions.
- `promote.py` implements the promotion gate.

---

## Implementation

```python
from mlflow import MlflowClient

TRACKING_URI = "http://localhost:5000"
MODEL = "fraud-detector"
PROD_ALIAS = "production"
CHALLENGER_VERSION = "2"

client = MlflowClient(tracking_uri=TRACKING_URI)


def f1_of(version: str) -> float:
    mv = client.get_model_version(MODEL, version)
    return client.get_run(mv.run_id).data.metrics["f1_score"]


def main() -> None:
    # Get current production model
    champion = client.get_model_version_by_alias(MODEL, PROD_ALIAS)
    champion_version = str(champion.version)

    champion_f1 = f1_of(champion_version)
    challenger_f1 = f1_of(CHALLENGER_VERSION)

    print(
        f"Champion: v{champion_version} (f1_score={champion_f1}), "
        f"Challenger: v{CHALLENGER_VERSION} (f1_score={challenger_f1})"
    )

    # Promote only if challenger performs better
    if challenger_f1 > champion_f1:
        client.set_registered_model_alias(
            MODEL,
            PROD_ALIAS,
            CHALLENGER_VERSION,
        )
        print(f"Promoted version {CHALLENGER_VERSION} to '{PROD_ALIAS}'.")
    else:
        print("Challenger rejected; production alias unchanged.")


if __name__ == "__main__":
    main()
```

---

## Execute

```bash
cd /root/code/monitoring
python3 promote.py
```

Output:

```text
Champion: v1 (f1_score=0.71), Challenger: v2 (f1_score=0.82)
Promoted version 2 to 'production'.
```

---

## Verification

Open the **MLflow Model Registry**.

Expected state:

- `fraud-detector`
- Version **2** has the **@production** alias.
- Version **1** no longer has the production alias.

Final registry:

| Version | Alias |
|---------|-------|
| v2 | `@production` |
| v1 | — |

---

## Key MLflow APIs Used

| API | Purpose |
|------|---------|
| `get_model_version_by_alias()` | Retrieves the current production (champion) model |
| `get_model_version()` | Gets model version metadata |
| `get_run()` | Reads evaluation metrics from the associated MLflow run |
| `set_registered_model_alias()` | Repoints the production alias to the better model |

---

## Champion/Challenger Workflow

```text
                 Retraining Pipeline
                        │
                        ▼
              Register Version 2
                        │
                        ▼
         Read Champion (production alias)
                        │
                        ▼
        Compare F1 Scores (Champion vs Challenger)
                        │
          ┌─────────────┴─────────────┐
          │                           │
 challenger > champion          challenger <= champion
          │                           │
          ▼                           ▼
 Move production alias          Keep current production
      to Version 2                alias unchanged
```

---

## Result

- Successfully implemented a **Champion/Challenger promotion gate**.
- Compared model performance before promotion.
- Promoted the challenger only because its **F1 Score (0.82)** exceeded the champion's **F1 Score (0.71)**.
- Updated the MLflow `production` alias to point to **Version 2**, ensuring production always serves the best-performing validated model.


### Screenshots
