# Day 26: Compare Model Runs and Select the Best
A xFusionCorp Industries data scientist has trained three candidate models on the same problem and logged them to the model-comparison experiment. Your task is to review the candidates side by side in the MLflow UI and explicitly mark the winning run so downstream tooling can pick it up.


The MLflow tracking server is already running on port 5000 and the model-comparison experiment has been pre-populated with three runs, each named after its algorithm (RandomForest, GradientBoosting, LogisticRegression) and carrying accuracy and f1_score metrics. The runs can be viewed via the MLflow UI button → model-comparison experiment.

Using the MLflow UI, inspect the three runs side by side and identify the winner by metrics.f1_score.

The run with the highest f1_score must carry a run-level tag: key production-candidate, value true.
Neither of the other two runs may carry a production-candidate tag.
The result can be confirmed in the MLflow UI: the model-comparison experiment lists three runs, and only the top-f1_score run shows the production-candidate tag on its detail page.
## Task Description
A xFusionCorp Industries data scientist has trained three candidate models on the same problem and logged them to the `model-comparison` experiment. Your task is to review the candidates side by side in the MLflow UI and explicitly mark the winning run so downstream tooling can pick it up.

### Requirements:
1. Open the MLflow UI (running on port 5000).
2. Open the `model-comparison` experiment.
3. Review the three runs: `RandomForest`, `GradientBoosting`, and `LogisticRegression`.
4. Identify the run with the highest `f1_score`.
5. Add a run-level tag to the winning run: key `production-candidate`, value `true`.
6. Ensure neither of the other two runs carries a `production-candidate` tag.

## Solution

### Steps to follow:

1. **Access the MLflow UI**
   Open your browser and navigate to `http://localhost:5000`.

2. **Select the Experiment**
   In the left sidebar, click on the **model-comparison** experiment.

3. **Compare Candidate Models**
   Observe the metrics for the three runs:
   - **GradientBoosting**: `f1_score` = 0.91
   - **RandomForest**: `f1_score` = 0.85
   - **LogisticRegression**: `f1_score` = 0.78

   The **GradientBoosting** run is the winner based on the highest `f1_score`.

4. **Tag the Winning Run**
   - Click on the **GradientBoosting** run name to open its detail page.
   - Scroll down to the **Tags** section.
   - Click the **+** (Add) button.
   - Key: `production-candidate`
   - Value: `true`
   - Click **Save**.

5. **Verify Other Runs**
   Ensure that the `RandomForest` and `LogisticRegression` runs do not have any `production-candidate` tags assigned.

### To verify:

In the **model-comparison** experiment view, only the **GradientBoosting** run should show the `production-candidate` tag on its detail page.

### Screenshots
<img width="1575" height="778" alt="MLflow Experiment View" src="https://github.com/user-attachments/assets/cd89aae3-9e5d-4322-a639-db43dc224a3" />
