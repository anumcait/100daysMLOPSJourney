# Day 59: Run Batch Predictions on a Dataset

## Objective

Implement the batch scoring process for a pre-trained `RandomForest` fraud detection model. The script should:

- Load the trained model from `model.pkl`
- Read transaction data from `input.csv`
- Generate integer class predictions (`0` or `1`) using `model.predict()`
- Add a `prediction` column to the dataset
- Save the results to `predictions.csv`

---

## Project Structure

```text
/root/code/serving/
├── batch_predict.py
├── input.csv
├── model.pkl
└── predictions.csv   (generated)
```

---

## Implementation

Replaced the `TODO` section in `batch_predict.py` with the following code:

```python
# Load the pre-trained model
model = joblib.load(MODEL_PATH)

# Read the input dataset
df = pd.read_csv(INPUT_CSV)

# Select feature columns
features = df[["amount", "hour", "num_tx_past_day"]]

# Generate integer class predictions
df["prediction"] = model.predict(features).astype(int)

# Save predictions
df.to_csv(OUTPUT_CSV, index=False)

# Print summary
print(f"Wrote {len(df)} rows to {OUTPUT_CSV}")
```

---

## Run

```bash
python3 /root/code/serving/batch_predict.py
```

---

## Output

The generated `predictions.csv` contains:

| amount | hour | num_tx_past_day | prediction |
|--------:|-----:|----------------:|-----------:|
| ... | ... | ... | 0 / 1 |

---

## Validation

- ✅ Model loaded using `joblib.load()`
- ✅ Input dataset read with `pandas.read_csv()`
- ✅ Used feature columns:
  - `amount`
  - `hour`
  - `num_tx_past_day`
- ✅ Predictions generated with `model.predict()`
- ✅ Predictions stored as integer class labels (`0` or `1`)
- ✅ Output written to `predictions.csv`
- ✅ Output row count matches the input row count

---

## Result

Successfully implemented the batch prediction pipeline for the fraud detection model. The script now performs end-to-end batch inference by loading the trained model, scoring every transaction, and exporting the predictions to a new CSV file.

### Screenshots

