# Day 25: MLflow Model Registry
The xFusionCorp Industries ML platform team needs two trained candidates promoted through the MLflow Model Registry so the ops side can track which model version is serving production traffic. Both runs already exist in the fraud-detection experiment. Your task is to register both as versions of a new fraud-detector model, add a model-level description, and assign challenger and champion aliases—all through the MLflow UI.


The MLflow tracking server is already running on port 5000 and two runs are pre-populated in the fraud-detection experiment: a baseline run (n_estimators=100, max_depth=5, f1_score=0.80) and an improved run (n_estimators=200, max_depth=10, f1_score=0.89). Both runs can be opened via the MLflow UI button → fraud-detection experiment.

Using the MLflow UI, reach the end state below. The order (baseline first, improved second) matters because MLflow assigns version numbers sequentially within a registered model.

A registered model named fraud-detector exists in the Model Registry.
The registered model carries a non-empty description that references the word fraud (any phrasing; for example Fraud detection model for xFusionCorp transactions).
Version 1 of fraud-detector is the baseline run and carries the alias challenger.
Version 2 of fraud-detector is the improved run and carries the alias champion.
The result can be confirmed by opening Model registry → fraud-detector in the MLflow UI. Two versions are listed, the description is shown at the top of the model page, and the alias column (or the Aliases field on each version) indicates challenger on v1 and champion on v2.
## Task Description
The xFusionCorp Industries data science team wants to organize their models using the MLflow Model Registry. After running multiple experiments, it's time to register the best versions of the fraud detection model, add descriptions, and use aliases to manage model stages.

### Requirements:
1. Register the model from the `baseline` run as the first version of a new model named `fraud-detector`.
2. Register the model from the `improved` run as the second version of the same `fraud-detector` model.
3. Add a model-level description: "Fraud detection model for xFusionCorp transactions". The description must contain the word "fraud".
4. Assign the alias `challenger` to Version 1.
5. Assign the alias `champion` to Version 2.

## Solution

### 1. Register Version 1 (Baseline)
- Open the MLflow Tracking UI.
- Navigate to the **Experiments** tab and select the experiment containing your runs.
- Click on the `baseline` run to open इसकी details.
- Scroll down to the **Artifacts** section.
- Click on the `model` directory/artifact.
- Click the **Register Model** button.
- In the dialog:
  - **Model**: Select `Create New Model`.
  - **Model Name**: type `fraud-detector`.
- Click **Register**.

### 2. Register Version 2 (Improved)
- Navigate back to the Experiments list and click on the `improved` run.
- Go to the **Artifacts** section.
- Click on the `model` artifact.
- Click the **Register Model** button.
- In the dialog:
  - **Model**: Select `Existing Model`.
  - **Model Name**: Select `fraud-detector` from the list.
- Click **Register**. This creates **Version 2**.

### 3. Add Model-Level Description
- Click on the **Models** tab in the top navigation bar.
- Click on the `fraud-detector` model.
- At the top of the model details page, find the **Description** box.
- Click the **Edit** (pencil) icon.
- Enter: `Fraud detection model for xFusionCorp transactions`
- Click **Save**.

### 4. Assign Alias to Version 1
- On the `fraud-detector` model page, click on **Version 1**.
- Locate the **Aliases** section.
- Click **Add Alias**.
- Enter `challenger`.
- Click **Save**.

### 5. Assign Alias to Version 2
- Go back to the `fraud-detector` model page or use the navigation to select **Version 2**.
- Locate the **Aliases** section.
- Click **Add Alias**.
- Enter `champion`.
- Click **Save**.

### Final Verification Check
Return to the **Model Registry** -> `fraud-detector` main page. You should see:
- **Description**: "Fraud detection model for xFusionCorp transactions" (contains "fraud").
- **Versions Table**:
  - **v1**: Alias `challenger`
  - **v2**: Alias `champion`

## Screenshots
<img width="1575" height="778" alt="image" src="https://github.com/user-attachments/assets/328b292b-da8a-47ae-b4c9-5180f9f60e9b" />
