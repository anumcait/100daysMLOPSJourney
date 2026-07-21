# Day 59: Run Batch Predictions on a Dataset
The xFusionCorp Industries ML platform team processes the overnight batch of transactions using a fraud-detection model. This process is executed with a standalone script that reads from input.csv, applies the pre-trained RandomForest model to each row, and generates predictions.csv. A scaffold for the script, located at /root/code/serving/batch_predict.py, includes the paths for the model, input, and output; however, the scoring flow is currently marked as TODO.

Your objective is to implement the batch scoring process within the script. Specifically, you need to ensure that it reads input.csv, utilizes the pre-trained model to score each row, and outputs predictions.csv, which should include a column for integer prediction class labels. After implementing the flow, run the script.


The project layout under /root/code/serving/:

model.pkl – Deterministic RandomForest trained at startup on the shared amount / hour / num_tx_past_day → is_fraud synthetic dataset.
input.csv – The 10-row batch input: three feature columns, no label column.
batch_predict.py – The scorer scaffold. The MODEL_PATH / INPUT_CSV / OUTPUT_CSV constants are set; the scoring flow (load the model, read the input, add an integer prediction column via model.predict(...), write the output) is left as a TODO to author.
The end state must include:

/root/code/serving/predictions.csv exists.
The output carries the three input columns plus a prediction column.
Every value in prediction is 0 or 1 (integer class label), not a float probability.
The output row count matches the input row count.

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
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/d09059a2-649a-4530-9e0a-89da507da136" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4d568269-1885-48e9-b401-5daa6be5a23c" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4a6ac81e-8b8c-42dd-950b-2abef3c17a8d" />



