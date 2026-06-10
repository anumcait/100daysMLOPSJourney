# Day 22: Creating Metadata-Rich Experiments in MLflow via UI

## Task Description
The xFusionCorp Industries team needs to organize their experiments in MLflow with proper metadata. To ensure better discoverability and management, the goal is to create separate experiments for different projects with specific descriptions and team tags, performing all actions directly through the MLflow User Interface.

### Requirements:
1. Access the MLflow Tracking UI.
2. Create an experiment named `fraud-detection` using the UI:
   - Add a description: `Production fraud detection models`.
   - Add an experiment tag: `team: ml-platform`.
3. Create another experiment named `churn-prediction` using the UI:
   - Add an experiment tag: `team: analytics`.
4. Verify that:
   - Both experiments appear in the experiment list.
   - The metadata (description and tags) are correctly displayed for each.

## Solution

### Steps performed in the MLflow UI:

**1. Create the `fraud-detection` Experiment:**
- Click the **+ Create Experiment** button in the top left of the Experiments sidebar.
- Enter `fraud-detection` in the **Experiment Name** field.
- In the **Artifact Location** (if applicable), leave the default or specify paths.
- Under the **Tags** section, click **Add Tag**:
  - Key: `team`, Value: `ml-platform`.
- Once created, navigate to the experiment page and click on the **Notes** or **Description** icon/section to add: `Production fraud detection models`.

**2. Create the `churn-prediction` Experiment:**
- Click the **+ Create Experiment** button again.
- Enter `churn-prediction` in the **Experiment Name** field.
- Under the **Tags** section, click **Add Tag**:
  - Key: `team`, Value: `analytics`.
- Click **Create**.

## Verification

1. **Experiment List**: Confirm `fraud-detection` and `churn-prediction` are visible in the left-hand sidebar.
2. **Metadata Check (Fraud Detection)**:
   - Click on `fraud-detection`.
   - Confirm the description "Production fraud detection models" is visible in the experiment overview.
   - Expand the **Tags** section to verify `team: ml-platform`.
3. **Metadata Check (Churn Prediction)**:
   - Click on `churn-prediction`.
   - Expand the **Tags** section to verify `team: analytics`.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/01facc17-c52a-474d-8d05-7d1660a275fa" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b0530208-b2b2-48c2-87f7-e5a22c495ea6" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d77c3850-9c50-4f33-ba5f-1c1d3462730d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/1b5b519e-b2c5-4d3c-b9e3-ff6a2dc2787d" />



