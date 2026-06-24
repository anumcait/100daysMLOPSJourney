# Day 36 - Automated Model Selection with MLflow

The xFusionCorp Industries ML platform team runs a "bake-off" experiment between multiple model candidates (Random Forest, Gradient Boosting, and Logistic Regression) to detect fraud. An orchestrator script at `src/models/bakeoff.py` is intended to query the MLflow tracking server, identify the candidate with the highest F1 score, and persist the results. However, the script currently selects the worst-performing model due to an incorrect sorting order and generates an incomplete report missing the model family. Your task is to fix these bugs so the production pipeline identifies the true winner.

## Objective
Automate the selection of the best-performing model from an MLflow experiment and ensure the resulting report contains all necessary metadata for downstream deployment.

## Task
- **Run the Bake-off**: Execute the three training scripts to populate the MLflow experiment with candidate runs.
- **Fix Search Logic**: Correct the `mlflow.search_runs` call to sort candidates by F1 score in descending order.
- **Enhance Reporting**: Update the report generation to include the winning `model_type` along with the `run_id` and `f1_score`.

## Solution

### Step 1: Correct Sorting Direction
The orchestrator was originally sorting F1 scores in ascending order, which caused the script to pick the worst model (the first row in the results). We updated the `search_runs` parameters to use descending order.
```python
runs = mlflow.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.f1_score DESC"],
    max_results=10,
)
```

### Step 2: Include Model Metadata in Report
The final report was missing the `model_type` key, which identifies the model family (e.g., `random_forest`). We modified the report dictionary to extract this from the MLflow run tags.
```python
winner = runs.iloc[0]
report = {
    "model_type": winner["tags.candidate"],
    "run_id": winner["run_id"],
    "f1_score": float(winner["metrics.f1_score"]),
}
```

### Step 3: Run and Verify Orchestrator
After making the corrections, we run the bake-off script and verify that `reports/winner.json` contains the correct winner with the highest F1 score.
```bash
cd /root/code/fraud-detection
python src/models/bakeoff.py
cat reports/winner.json
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Model Bake-off** | A competitive comparison where multiple model architectures are trained on the same data and evaluated using the same metrics. |
| **MLflow Search Runs** | A programmatic way to query and filter experiment results based on parameters, metrics, and tags. |
| **Orchestration** | Using scripts to coordinate different stages of the ML lifecycle, such as selecting the best model after multiple training runs. |

## Summary
By fixing the sorting direction and enriching the metadata in the final report, we have successfully automated the "Model Selection" phase of our MLOps pipeline. The system now reliably identifies the top-performing candidate from an experiment and provides a clear, machine-readable report that can be used by automated deployment scripts to promote the best model to production.
