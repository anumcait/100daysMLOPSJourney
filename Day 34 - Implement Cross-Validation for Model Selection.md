# Day 34 - Implement Cross-Validation for Model Selection

The xFusionCorp Industries ML platform team evaluates fraud-detection candidates with k-fold cross-validation so every candidate is measured on multiple folds of an imbalanced dataset. A draft cross-validation scaffold exists at /root/code/fraud-detection/src/models/cross_validate.py, but its report does not match the release checklist and the fold strategy does not preserve the class ratio. Your task is to correct the scaffold so the cross-validation report lands in the expected shape.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to confirm—the dashboard loads with an empty fraud-detection-cv experiment.

The project layout under /root/code/fraud-detection/:

data/train.csv – A pre-generated 200-row synthetic binary-classification dataset with an imbalanced class split (roughly 70 / 30). Do not regenerate it.
src/models/cross_validate.py – The cross-validation scaffold. Every concern other than the splitter and the aggregate schema is correctly wired: fold iteration, per-fold metric computation, nested MLflow runs under a parent, JSON persistence, and artefact logging.
reports/ – Where the cross-validation report must land.
Open src/models/cross_validate.py in the VS Code editor, correct the two problems that keep the report from meeting the release checklist, save, and run the script.

The end state must include:

A file at /root/code/fraud-detection/reports/cv_results.json (absolute path, inside the project's reports directory).
That JSON contains exactly these seven top-level keys: mean_accuracy, std_accuracy, mean_f1, std_f1, mean_roc_auc, std_roc_auc, folds. Every mean_* and std_* value is numeric.
The folds value is a list of five per-fold dictionaries; each carries the keys fold, accuracy, f1, roc_auc.
The cross-validation splitter is stratification-aware – Each fold preserves the dataset's class ratio.
One parent MLflow run in the fraud-detection-cv experiment with five nested children (fold-1 through fold-5), each logging the per-fold metrics.
StratifiedKFold is already imported at the top of the scaffold—no new imports are required. The fix is confined to the CV splitter and the aggregate dict.

## Objective
Improve the model evaluation process by implementing Stratified K-Fold Cross-Validation to ensure robust performance metrics across different data splits while maintaining the class distribution.

## Task
- Update `src/models/cross_validate.py` to use `StratifiedKFold` instead of `KFold`.
- Ensure the final report (`cv_results.json`) contains exactly the required seven top-level keys: `mean_accuracy`, `std_accuracy`, `mean_f1`, `std_f1`, `mean_roc_auc`, `std_roc_auc`, and `folds`.
- Verify that the per-fold metrics are correctly logged and the parent MLflow run shows nested child runs for each fold.

## Solution

### Step 1: Implement Stratified K-Fold
We replaced the standard K-Fold splitter with `StratifiedKFold` to preserve the target class ratio in each fold, which is critical for imbalanced fraud detection datasets.
```python
cv = StratifiedKFold(
    n_splits=N_SPLITS,
    shuffle=True,      
    random_state=42    
)
```

### Step 2: Update Metrics Aggregation
We expanded the `aggregate` dictionary to include the standard deviation of each metric, providing a measure of model stability across folds.
```python
aggregate = {
    "mean_accuracy": round(float(np.mean(acc_vals)), 6),
    "std_accuracy": round(float(np.std(acc_vals)), 6),
    "mean_f1": round(float(np.mean(f1_vals)), 6),
    "std_f1": round(float(np.std(f1_vals)), 6),
    "mean_roc_auc": round(float(np.mean(auc_vals)), 6),
    "std_roc_auc": round(float(np.std(auc_vals)), 6),
    "folds": fold_results,
}
```

### Step 3: Execute and Verify
Ran the cross-validation script and inspected the generated report to confirm it meets the organizational schema requirements.
```bash
cd /root/code/fraud-detection
python src/models/cross_validate.py
cat reports/cv_results.json
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Stratified K-Fold** | A variation of K-Fold that ensures each fold contains approximately the same percentage of samples of each target class as the complete set. |
| **Standard Deviation (Std)** | A statistic that measures the dispersion of a dataset relative to its mean. In CV, it indicates how much the model performance varies across different data splits. |
| **Nested MLflow Runs** | A way to organize experiments where a parent run represents the overall process and child runs represent individual steps like folds in cross-validation. |

## Summary
By switching to Stratified K-Fold and logging standard deviation, we've made our model evaluation significantly more reliable. This approach ensures that our performance metrics are not just high on average, but consistent and representative of how the model will perform on real-world, imbalanced data.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2fdfbc23-68eb-42eb-bdbd-983f30bd63b7" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/ff2c8897-2c44-4741-a7d5-139b754aa0a4" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/43f7caff-79c4-4c53-b37f-51106ebef4d9" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3c3ad2f0-9cc9-456a-a795-05542f1663af" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/e8130045-0ed1-44ed-9265-f0b2e1dbe2c5" />





