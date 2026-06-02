# Day 16 — Tracking ML Metrics with DVC

The xFusionCorp Industries ML team needs to monitor the performance of their models to ensure quality and track improvements over time. Instead of manually recording accuracy or loss in spreadsheets, we will integrate metric tracking directly into the DVC pipeline. This allows DVC to treat performance results as first-class citizens, enabling easy comparison between different versions.

## 📋 Task Summary

Integrate metric tracking into the `fraud-detection` project:
- Modify the pipeline to output a `metrics.json` file.
- Update `dvc.yaml` to define a `metrics` section.
- Run the pipeline and verify the metrics.
- Use DVC commands to view and compare metrics.

---

## 🛠️ Step 1: Update the Evaluation Script

The training or evaluation script must save the calculated metrics to a file (JSON or YAML).

### `src/models/evaluate.py`
```python
import json
import pandas as pd
from sklearn.metrics import accuracy_score, precision_score, recall_score
import pickle

# Load model and test data
model = pickle.load(open("models/model.pkl", "rb"))
test_data = pd.read_csv("data/processed/test.csv")

X_test = test_data.drop("target", axis=1)
y_test = test_data["target"]

# Predict and calculate metrics
y_pred = model.predict(X_test)
metrics = {
    "accuracy": accuracy_score(y_test, y_pred),
    "precision": precision_score(y_test, y_pred),
    "recall": recall_score(y_test, y_pred)
}

# Save metrics to json
with open("metrics.json", "w") as f:
    json.dump(metrics, f, indent=4)
```

---

## 🛠️ Step 2: Configure `dvc.yaml`

Update the `dvc.yaml` file to include an `evaluate` stage and mark the output file as `metrics`. Unlike `outs`, metrics are not typically cached in the DVC cache by default (though they can be), allowing them to be easily read by Git and DVC summary commands.

```yaml
stages:
  # ... previous stages (process_data, split_data, train) ...

  evaluate:
    cmd: python src/models/evaluate.py
    deps:
      - data/processed/test.csv
      - models/model.pkl
      - src/models/evaluate.py
    metrics:
      - metrics.json:
          cache: false
```

---

## 🛠️ Step 3: Run the Pipeline

Execute the pipeline to generate the metrics.

```bash
cd /root/code/fraud-detection

# Run the pipeline
dvc repro
```

---

## 🛠️ Step 4: View Tracked Metrics

Use DVC to display the captured performance values.

```bash
# Show current metrics
dvc metrics show
```

### Expected Output
```text
Path          accuracy    precision    recall
metrics.json  0.942       0.915        0.887
```

---

## 🛠️ Step 5: Visualizing Plots (Optional)

DVC also supports tracking data for plots (like Confusion Matrices or Precision-Recall curves).

1.  **Generate Plot Data**: Save a CSV file with predicted vs actual values.
2.  **Update `dvc.yaml`**:
    ```yaml
    evaluate:
      ...
      plots:
        - plots.csv:
            cache: false
    ```
3.  **View Plots**:
    ```bash
    dvc plots show
    ```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **Metrics** | Performance measurements (scalars like accuracy, loss) tracked by DVC. |
| **`metrics` field** | A Special field in `dvc.yaml` that tells DVC to treat a file as a performance report. |
| **`dvc metrics show`** | Command to aggregate and display metrics from the current workspace or across Git commits. |
| **`cache: false`** | Often used for metrics to keep the JSON file small and easily viewable without needing `dvc pull`. |
| **Plots** | Visual representations of data (ROC, Precision-Recall) that DVC can render as HTML. |

---

## ⚠️ Common Pitfalls

1.  **Incorrect JSON Structure** — Use a flat dictionary for the simplest `dvc metrics show` output.
2.  **Missing `cache: false`** — If you don't set this, the metrics file itself is moved to the DVC cache, which might make it harder to see the raw values in the file system without DVC.
3.  **Dependency Tracking** — Ensure `metrics.json` is listed under `metrics` in `dvc.yaml`, not just as a regular output.

### Screenshots
