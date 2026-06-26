# Day 36 - Automated Model Selection with MLflow
The xFusionCorp Industries ML platform team is running a three-way bake-off between a RandomForest, a GradientBoosting, and a LogisticRegression candidate for fraud detection, with every candidate tracked as an MLflow run in the bakeoff experiment. Three correct trainer scripts are already in place, but the orchestrator at /root/code/fraud-detection/src/models/bakeoff.py picks the wrong winner and writes an incomplete report. Your task is to correct the orchestrator so the saved winner is the highest-F1 candidate and the report identifies which model family won.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty bakeoff experiment.

The project layout under /root/code/fraud-detection/:

data/train.csv – The same 200-row synthetic binary-classification dataset Day 34 uses (imbalanced roughly 70 / 30).
src/models/train_rf.py, src/models/train_gb.py, src/models/train_lr.py – Three independent trainer scripts. Each one fits its named estimator with 3-fold stratified CV and logs one MLflow run tagged candidate=<model family> with the mean f1_score metric and its hyperparameters. These three files are correct and need no edits.
src/models/bakeoff.py – The orchestrator. It queries the bakeoff experiment with mlflow.search_runs(...) and writes /root/code/fraud-detection/reports/winner.json. Two specific corrections are required.
Run each of the three trainer scripts once so every candidate is logged, open src/models/bakeoff.py in the VS Code editor, correct the two problems that keep the report from meeting the release checklist, save, and run the orchestrator.

The end state must include:

Three runs exist in the bakeoff MLflow experiment, one per candidate, each with tags.candidate, the candidate's hyperparameters, and metrics.f1_score.
A JSON file at /root/code/fraud-detection/reports/winner.json with exactly three keys: model_type (one of random_forest, gradient_boosting, logistic_regression), run_id, and f1_score.
The model_type, run_id, and f1_score stored in winner.json correspond to the candidate with the highest f1_score in the bakeoff experiment.
The MLflow Compare view—select all three runs in the experiment's run list and click Compare—is the fastest way to eyeball which candidate won and spot-check the report.

## Task in brief

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

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3a026d82-f722-4ea2-b69f-3cbbb385215c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3cdfa179-9286-4fef-963d-37821abcbebd" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/26b66d1e-a043-4324-af31-6e9c9b7b3b51" />



