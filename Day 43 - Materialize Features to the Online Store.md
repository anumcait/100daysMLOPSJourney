# Day 43 - Materialize Features to the Online Store

The xFusionCorp Industries ML platform team stages a materialisation script (`materialize.sh`) under `/root/code/fraud-detection/feature_repo/` so the batch job that populates the Feast online store is always repeatable. The registry has already been applied against a correct `features.py`, but running the script writes zero rows into the online sqlite store. Your task is to correct `materialize.sh` so `./materialize.sh` actually populates the online store, and confirm a non-null amount comes back from `store.get_online_features()` for a known customer.

The Feast UI is already running on port 8888. The Feast UI button at the top of the lab can be opened to confirm—the dashboard loads the `fraud_detection` project, the customer entity, and the `customer_transaction_features` feature view. Materialisation status is not visible in the UI; the online store is inspected from the terminal.

The repository layout under `/root/code/fraud-detection/feature_repo/`:
- `feature_store.yaml` – Local provider, sqlite online store at `data/online_store.db`. Correct.
- `features.py` – Declares the customer entity (`join_keys=["customer_id"]`) and the `customer_transaction_features` view over the transactions source. Correct.
- `data/transactions.parquet` – 200-row synthetic source, event timestamps from 2024-01-01 onward.
- `data/registry.db` – Already written by `feast apply` at startup.
- `materialize.sh` – Single-purpose shell script that calls `feast materialize-incremental "$END_DATE"`.

Open `materialize.sh` in the VS Code editor, correct the `END_DATE` so it lands at or after the source's last event (e.g. `2025-12-31T23:59:59`), save, and run `./materialize.sh` from inside `/root/code/fraud-detection/feature_repo/`.

The end state must include:
- `data/online_store.db` exists and its on-disk size is comfortably larger than the bare sqlite header (≥ 4 KB).
- `store.get_online_features(features=["customer_transaction_features:amount", ...], entity_rows=[{"customer_id": i}, ...])` returns at least one non-null amount value for a customer id present in the source.
- `materialize.sh`'s end date is an ISO-8601 date on or after `2024-01-01`.
- `feast materialize-incremental` takes a single ISO-8601 end date and uses the feature view's TTL to pick the start watermark on the first run. The source events span 2024-01-01 to 2024-01-09, and the feature view's TTL is generous enough that any end date on or after the source's last event materialises every row.

## Objective
Materialize Feast features from your offline store into the online SQLite database registry by adjusting the query target window date in the shell script config, execution, and confirmation via client SDK feature retrieval.

## Task
- **Fix materialize.sh End Date**: Update the `END_DATE` variable in `materialize.sh` from `1970-12-31T00:00:00` to a target date after the parquet data window (e.g. `2025-12-31T23:59:59`).
- **Run Materialization**: Execute `./materialize.sh` to trigger the feature pipeline.
- **Inspect DB Size**: Confirm that the SQLite online database (`data/online_store.db`) size is ≥ 4 KB, indicating that data has successfully populated.
- **Verify Online Querying**: Run a Python snippet using `FeatureStore.get_online_features` to verify that Feast serves actual, non-null values for a registered customer ID.

## Solution

### Step 1: Inspect the Script
Review the existing raw script in `/root/code/fraud-detection/feature_repo/materialize.sh`.
```bash
#!/bin/bash
set -euo pipefail
cd "$(dirname "$0")"

# Incorrect END_DATE: Parquet records begin in 2024, so this writes nothing
END_DATE="1970-12-31T00:00:00"

feast materialize-incremental "$END_DATE"
```

### Step 2: Edit `materialize.sh`
Modify the script to change the timestamp window to a future target (or a timestamp after the latest event in the parquet data, which spans `2024-01-01` to `2024-01-09`):
```bash
#!/bin/bash
set -euo pipefail
cd "$(dirname "$0")"

# Corrected END_DATE to cover 2024 events
END_DATE="2025-12-31T23:59:59"

feast materialize-incremental "$END_DATE"
```

### Step 3: Run the Script and Populated Database
Grant execution permissions and run the script:
```bash
cd /root/code/fraud-detection/feature_repo
chmod +x materialize.sh
./materialize.sh
```

Expected output:
```
Materializing 1 feature views from 2023-12-31 23:59:59+00:00 to 2025-12-31 23:59:59+00:00 into the sqlite online store.

Database tables created.
```

Verify that the online store database file size increases:
```bash
ls -lh data/online_store.db
```

### Step 4: Verify Online Retrieval
Run a python script or interactive session to retrieve online features for entity `customer_id: 1` using the Feast SDK:
```bash
python3
```
```python
from feast import FeatureStore

# Initialize the FeatureStore pointing to the local workspace
store = FeatureStore(repo_path=".")

# Retrieve the online feature amount
features = store.get_online_features(
    features=["customer_transaction_features:amount"],
    entity_rows=[{"customer_id": 1}],
).to_dict()

print(features)
```
Expected output:
```python
{'customer_id': [1], 'amount': [58.21]}  # Returns valid amount instead of None
```

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Feast Online Store** | A low-latency database (SQLite in local dev, Redis/DynamoDB in prod) used to serve the most recent version of feature values for real-time predictions. |
| **Materialization** | The process of pulling telemetry data from the offline raw sources (parquet/databases), tracking watermarks, and inserting the newest record values into the online database. |
| **`feast materialize-incremental`** | Command that moves features into the online store starting from the last materialized timestamp up to the defined end date, avoiding redundant full table scans. |
| **TTL (Time to Live)** | Configures how far back Feast is willing to look to find a feature value in the offline store if no events occurred recently. The start watermark defaults to `END_DATE - TTL`. |
| **Online Feature Querying** | Low-latency feature retrieval via `get_online_features()` by supplying the key-value dictionary identifying the target entities. |

## Summary
By adjusting the `END_DATE` in the project's staging `materialize.sh` script to `2025-12-31T23:59:59`, Feast was able to successfully identify offline parquet records starting in 2024. Running the script triggered the incremental MLOps pipeline, updating `/root/code/fraud-detection/feature_repo/data/online_store.db` with the newly populated tables. The verification step confirmed the successful write operations, allowing developers to query non-null customer transaction amounts in real-time.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4c16426c-a3b0-426a-9d5b-705863fa3fd2" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b53918fc-d032-48ad-8651-ab271c1d6c3b" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/aff311f5-e64b-40eb-940d-4b53bc6122f5" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5d753c8c-fb51-4187-bd51-32bbea7c10c8" />
