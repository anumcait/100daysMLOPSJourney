# Day 42 - Define Feature Views in Feast

The xFusionCorp Industries ML platform team keeps the fraud-detection feature definitions in a Feast repository at `/root/code/fraud-detection/feature_repo/`. A draft `features.py` exists there, and the registry that `feast apply` wrote is inconsistent with the source data in `data/transactions.parquet`. Your task is to correct `features.py`, re-apply the registry, and confirm the corrected `customer_transaction_features` view in the Feast UI.

The Feast UI is already running on port 8888. The Feast UI button at the top of the lab can be opened to confirm—the dashboard loads the `fraud_detection` project with one entity and one feature view carrying the draft declarations.

The repository layout under `/root/code/fraud-detection/feature_repo/`:
- `feature_store.yaml` – The Feast config (project `fraud_detection`, local provider, sqlite online store, file offline store). Correct and must remain intact.
- `data/transactions.parquet` – A 200-row synthetic source keyed by `customer_id`; carries `amount` as `Float32`, `hour` + `num_tx_past_day` + `is_fraud` as `Int64`, and an `event_timestamp` column. Correct and must remain intact.
- `features.py` – Declares one `FileSource`, one `Entity`, and one `FeatureView`. Needs correction to match the source.
- `data/registry.db` – Written by `feast apply` at startup from the draft definitions; must be re-applied after the fixes.

Open `features.py` in the VS Code editor, align the declarations with the source, save, and run `feast apply` from inside `/root/code/fraud-detection/feature_repo/`.

The end state must include:
- The customer entity in the registry has `join_keys = ["customer_id"]`.
- The `customer_transaction_features` feature view's `amount` field is declared as `Float32` (matching the parquet writer's output type).
- `feast apply` exits without error and the Feast UI reflects the corrected entity and feature-view schema.
- The Feast UI's Entities and Feature Views tabs surface the applied values directly—the current (draft) values are visible there so the required change is easy to eyeball against the task's end-state.

## Objective
Correct the entity definition and feature view configurations in Feast's `features.py` file to align with the underlying parquet file source schema, then compile with `feast apply` to update the local feature store registry and verify the results in the Feast UI.

## Task
- **Fix the Entity Primary/Join Key**: Change the customer entity's join key from a generic identifier to `customer_id` inside `features.py` to match the schema of the parquet source.
- **Correct the Feature Field Data Types**: Update the `amount` field data type from `Float64` to `Float32` inside the `customer_transaction_features` Feast Feature View to match the parquet database's column type.
- **Compile the Feature Definitions**: Run `feast apply` to write the updated configurations into the local registry database (`data/registry.db`).
- **Verify in the Feast UI**: Ensure the UI dashboard properly displays the newly registered schemas for the entity and feature view.

## Solution

### Step 1: Inspect the Schema Differences
The source data in `/root/code/fraud-detection/feature_repo/data/transactions.parquet` is keyed by `customer_id` and contains the `amount` feature as `Float32`. However, the starter configuration draft in `/root/code/fraud-detection/feature_repo/features.py` contains incorrect parameters, such as a wrong join key for the entity and an incorrect data type (`Float64`) for the `amount` feature. 

### Step 2: Open and Modify `features.py`
Open `/root/code/fraud-detection/feature_repo/features.py` and modify the following declarations:

**Before (Draft Definitions):**
```python
# Draft customer entity definition
customer = Entity(
    name="customer", 
    join_keys=["cust_id"]  # Or "id"
)

# Draft feature view definition
customer_transaction_features = FeatureView(
    name="customer_transaction_features",
    entities=[customer],
    ttl=timedelta(days=1),
    schema=[
        Field(name="amount", dtype=Float64),  # Mismatch: Parquet uses Float32
        Field(name="hour", dtype=Int64),
        Field(name="num_tx_past_day", dtype=Int64),
        Field(name="is_fraud", dtype=Int64),
    ],
    source=transactions_source
)
```

**After (Corrected Definitions):**
```python
# Corrected customer entity definition
customer = Entity(
    name="customer", 
    join_keys=["customer_id"]  # Aligned with parquet key
)

# Corrected feature view definition
customer_transaction_features = FeatureView(
    name="customer_transaction_features",
    entities=[customer],
    ttl=timedelta(days=1),
    schema=[
        Field(name="amount", dtype=Float32),  # Aligned with Parquet Float32 type
        Field(name="hour", dtype=Int64),
        Field(name="num_tx_past_day", dtype=Int64),
        Field(name="is_fraud", dtype=Int64),
    ],
    source=transactions_source
)
```

### Step 3: Apply the Corrected Registry Config
Navigate into the feature repository directory and run `feast apply` to synchronize the changes with the SQLite registry database:
```bash
cd /root/code/fraud-detection/feature_repo/
feast apply
```

*Note: If Feast UI does not pick up the schema immediately, or you want to do a clean reset, remove the old registry database first and compile fresh:*
```bash
rm -f data/registry.db
feast apply
```

Expected output:
```
Created entity customer
Created feature view customer_transaction_features
Created sqlite table fraud_detection_customer_transaction_features
```

### Step 4: Verify the Schema in the Feast UI
Open the Feast UI (normally running on port `8888`) or click the Feast UI button in the interface.
- In the **Entities** tab, select the `customer` entity and verify that `join_keys` displays `["customer_id"]`.
- In the **Feature Views** tab, select `customer_transaction_features` and ensure the `amount` attribute has its type listed as `Float32`.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Feast Entities** | Represents the primary keys of the features. In MLOps pipelines, they form the primary join criteria to associate specific feature vectors with concrete records (like `customer_id`). |
| **Feast Feature Views** | Cohesive collections of features defined on top of a specific data source. They contain the schemas, metadata (such as time-to-live `ttl`), and pointers to coordinate feature value mapping. |
| **Data Type Matching** | Feast enforces strong schemas matching the offline files exactly. A mismatch between the declared type (e.g. `Float64`) and the actual parquet metadata (e.g. `Float32`) will throw errors during schema application or inference retrieval. |
| **`feast apply`** | Scans the Python script definitions and registers entities and feature schemas into standard metadata stores (SQLite `registry.db`) while preparing online and offline query structures. |
| **Feast Registry** | The central database holding declarative schemas and parameters for all features, enabling training pipelines and staging environments to operate on exact metadata versions. |

## Summary
To resolve schema mismatches between Feast metadata and the underlying source `transactions.parquet`, `features.py` was corrected in two ways: updating the customer Entity join key to `customer_id` and setting the `amount` feature data type to `Float32`. Running `feast apply` re-populated the local SQLite registry database `data/registry.db` and updated the online tables, ensuring Feast has consistent feature/schema references that are correctly exposed in the Feast UI.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/48a6ce72-d2ac-4c95-890b-07bedaca15cc" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/4003b712-cd2b-4e83-bcf6-f1b85e91b55d" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/5dc3f4a4-b53f-40eb-b808-5ef3077d159d" />


