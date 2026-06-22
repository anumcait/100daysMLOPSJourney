# Day 34 - Implement Cross-Validation for Model Selection

Proper model evaluation is a cornerstone of MLOps. In this task, we refine our cross-validation strategy to ensure that our performance metrics are both robust and representative, especially when dealing with imbalanced datasets.

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
