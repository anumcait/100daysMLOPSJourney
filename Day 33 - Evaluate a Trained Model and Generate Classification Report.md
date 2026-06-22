# Day 33 - Evaluate a Trained Model and Generate Classification Report

Proper model evaluation is a cornerstone of MLOps. In this task, we refine an evaluation script to comply with organizational reporting standards, ensuring that all key performance metrics and visual artifacts are correctly generated and tracked.

## Objective
Correct the evaluation script (`src/models/evaluate.py`) to generate a comprehensive five-metric JSON report and a confusion matrix, and log these artifacts to MLflow.

## Task
- Update the `METRICS_JSON` path to point to the project's reports directory.
- Expand the metrics calculation to include precision, recall, and AUC-ROC.
- Ensure the final report (`metrics.json`) contains exactly the required five keys: `accuracy`, `precision`, `recall`, `f1_score`, and `auc_roc`.
- Verify that both the metrics file and the confusion matrix image are logged as MLflow artifacts.

## Solution

### Step 1: Fix the Metrics Output Path
The original script pointed to a temporary directory. We updated it to use the project's standardized `reports/` folder.
```python
REPORTS_DIR = "/root/code/fraud-detection/reports"
METRICS_JSON = os.path.join(REPORTS_DIR, "metrics.json")
```

### Step 2: Implement Comprehensive Metrics
We expanded the `metrics` dictionary to calculate all five required performance indicators, ensuring consistent rounding for readability.
```python
metrics = {
    "accuracy": round(accuracy_score(y, preds), 6),
    "precision": round(precision_score(y, preds), 6),
    "recall": round(recall_score(y, preds), 6),
    "f1_score": round(f1_score(y, preds), 6),
    "auc_roc": round(roc_auc_score(y, proba), 6),
}
```

### Step 3: Execute and Validate
Ran the evaluator to generate the local reports and sync with the MLflow tracking server.
```bash
cd /root/code/fraud-detection
python src/models/evaluate.py
```

**Expected Artifacts:**
- `reports/metrics.json`
- `reports/confusion_matrix.png`

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Classification Report** | A summary of key metrics (precision, recall, F1) that provides a deeper look into model performance than accuracy alone. |
| **Confusion Matrix** | A table used to describe the performance of a classification model by showing the count of true/false positives and negatives. |
| **Artifact Logging** | Storing non-scalar outputs (images, CSVs, JSONs) alongside experiment runs for full auditability. |
| **AUC-ROC** | A performance measurement for classification problems at various threshold settings; it tells how much the model is capable of distinguishing between classes. |

## Summary
By standardizing the evaluation output, we ensured that every model candidate is held to the same rigorous performance checklist. Logging these as artifacts in MLflow transforms individual one-off evaluations into a searchable, comparable, and permanent record of the project's progress.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/8a5c973c-adfa-45c6-9111-67755d21d556" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/06c01d1c-f4a6-4a81-96d0-3d7385da9b3c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ca5fc58b-de2d-479a-8fdd-35d3970d62a3" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ac55a5a0-e482-4bec-b2c4-54392f55c1e7" />




