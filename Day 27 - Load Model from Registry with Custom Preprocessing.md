# Day 27: Load Model from Registry with Custom Preprocessing

The xFusionCorp Industries deployment team requires a batch-prediction wrapper around the registered `fraud-detector` champion model. This wrapper includes custom preprocessing logic to ensure the model receives data in the expected format before being exposed to downstream services.

## Task Description

Complete the MLflow-side implementation in the pre-written script to load the champion model from the registry and execute a batch prediction on a set of input data.

### Requirements:

1. **Load the Champion Model**: Load the `champion` version of the `fraud-detector` model from the MLflow Model Registry into a variable named `inner_model`.
2. **Model URI**: Use the predefined `MODEL_URI`: `models:/fraud-detector@champion`.
3. **Execute Prediction**: Run the batch prediction over the pre-staged inputs.
4. **Augment Data**: Attach the resulting predictions as a new column named `prediction` to the inputs DataFrame.
5. **Save Output**: Write the final DataFrame to `OUTPUT_CSV` (`/root/code/predictions.csv`) ensuring no index is included.

## Solution

### Implementation Steps

The script `predict_with_preprocessing.py` provides a `ScaledPredictor` class that wraps the underlying model. The task involves filling in the logic to bridge the registry and the prediction output.

#### Step 1: Loading the Model from Registry
To fetch the model version tagged with the `champion` alias, use the `mlflow.pyfunc.load_model` method.

```python
# TODO 1: Load the champion version of fraud-detector
inner_model = mlflow.pyfunc.load_model(MODEL_URI)
```

#### Step 2: Batch Prediction and Output
Once the model is loaded, we apply it to the input DataFrame and save the results.

```python
# TODO 2: Run batch prediction and save results
inputs['prediction'] = inner_model.predict(inputs)
inputs.to_csv(OUTPUT_CSV, index=False)
```

#### Step 3: Script Execution
Run the script to verify the implementation:

```bash
python3 /root/code/predict_with_preprocessing.py
```

### Verification

After execution, the following state should be achieved:
- A new file `predictions.csv` exists in `/root/code/`.
- The CSV contains a header and a `prediction` column.
- The file contains exactly 10 rows of data, matching the input dataset.

## Key Concepts

| Concept | Description |
| :--- | :--- |
| **Model Aliases** | Labels like `@champion` allow for stable URIs even when version numbers change. |
| **Custom Wrappers** | Python classes that inherit from `mlflow.pyfunc.PythonModel` to encapsulate preprocessing/postprocessing. |
| **Batch Prediction** | Processing a large set of inputs simultaneously rather than single point inference. |
| **Registry Plumbing** | The programmatic interface to fetch and interact with models managed by the MLflow Registry. |

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/604415ab-c978-4dbc-8c99-0dcce69aaadb" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/a57c469b-4578-461d-ae53-439f3dfe7286" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/379aa56c-0573-4b37-b4e1-132e93d147ca" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/df6f4d99-ce50-408a-93f5-6c5b851f718c" />




