# Day 46 - Author Data-Quality Expectations with Great Expectations

The xFusionCorp Industries ML platform team wants data-schema contracts on every batch that feeds the fraud-detector model—catch the malformed row upstream of training, not three hours later in production. A Great Expectations project is already initialised at `/root/code/dataquality/gx/` with a pandas data source reading `data/transactions.csv`, an empty `fraud_schema` suite, and a default checkpoint wired to publish results to Data Docs on every run. Your task is to populate the suite with four expectations and run the checkpoint so Data Docs shows them green.

The end state must include:
- `gx/expectations/fraud_schema.json` has all four expectations by type (`expect_table_columns_to_match_set`, two `expect_column_values_to_be_between` entries – One per column — and `expect_column_values_to_be_in_set`).
- Each expectation's kwargs match the TODO spec.
- The most recent validation JSON under `gx/uncommitted/validations/` has `success: true`.
- The Data Docs index page served on port `8081` references `fraud_schema`.

Great Expectations treats data quality as code—expectation suites are versioned artefacts in the same repo as the model that consumes the data, run by the same CI that runs pytest. A run's result JSON is machine-readable (a downstream CI-gate lab consumes it), and Data Docs is the human-readable rendering of the same content. This lab lays the ground for both.

## Objective
Declare database validation contracts using Great Expectations to capture data schema errors upstream of training, asserting correct column sets, valid amount ranges, hour distributions, and categorical targets inside a repeatable workflow validation program.

## Task
- **Identify Target Locations**: Locate `/root/code/dataquality/author_expectations.py` containing placeholders for adding expectations to the `fraud_schema` suite.
- **De-serialize Expectation Rules**: Configure table-set matching specifications for input columns, ensure non-negative bounds for transaction amounts, constrain timestamps to valid hours, and enforce binary classification flags.
- **Trigger Checkpoint Validation**: Run the configuration script to serialize expectation settings to `gx/expectations/fraud_schema.json` and validate `data/transactions.csv` against the parameters.
- **Verify HTML Outputs**: Navigate via port `8081` to observe Data Docs validation outcomes and confirm all test asserts are rendered green (Success).

## Solution

### Step 1: Open code placeholders
1. Access the VS Code editor pointing at `/root/code/dataquality/author_expectations.py`.

### Step 2: Establish the Validation Conditions
1. Locate the TODO section within the script.
2. Replace the comment placeholder `# (expectations go here)` with the operations declaring expectations on the suite:
   ```python
   # TODO 1: Declare the required schema -- the four columns
   suite.add_expectation(
       ge.ExpectTableColumnsToMatchSet(
           column_set=["amount", "hour", "num_tx_past_day", "is_fraud"]
       )
   )

   # TODO 2: Guard amount -- no negative transactions.
   suite.add_expectation(
       ge.ExpectColumnValuesToBeBetween(
           column="amount",
           min_value=0,
       )
   )

   # TODO 3: Guard hour -- valid 24-hour range.
   suite.add_expectation(
       ge.ExpectColumnValuesToBeBetween(
           column="hour",
           min_value=0,
           max_value=23,
       )
   )

   # TODO 4: Guard is_fraud -- binary label.
   suite.add_expectation(
       ge.ExpectColumnValuesToBeInSet(
           column="is_fraud",
           value_set=[0, 1],
       )
   )
   ```
3. Save the edits to the configuration script.

### Step 3: Run the Verification Script
1. Open a terminal and run the authoring script:
   ```bash
   python3 /root/code/dataquality/author_expectations.py
   ```
   **Expected terminal response:**
   ```
   Persisted 4 expectations to fraud_schema
   Checkpoint default result: success=True
   ```

### Step 4: Verify Success in Data Docs
1. Click the **Data Docs** link or open port `8081` in your browser.
2. Confirm the validation run under `fraud_schema` logs is marked successful and every expectation displays a green checkbox/Success status.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Great Expectations (GX)** | An open-source Python library used to test, document, and profile data to maintain pipeline integrity. |
| **Expectation Suite** | A collection of validation assertions grouped together to describe the schema, types, and domains of a dataset. |
| **Checkpoints** | The primary runtime entrypoint to run validation matches, generate validation artifacts, and run actions like refreshing Data Docs logs. |
| **Data Docs** | HTML pages automatically compiled from structured JSON validations and expectations files, providing a readable portal for data quality results. |

## Summary
The xFusionCorp Industries team required schema-level constraints on incoming transaction streams before feeding them to downstream models. By editing `/root/code/dataquality/author_expectations.py` to declare a Great Expectations `fraud_schema` suite, parameters were validated for the columns set (`amount`, `hour`, `num_tx_past_day`, and `is_fraud`), transaction bounds (amount $\ge$ 0), hour limits (0–23), and classification domains. Running the configuration script registered the rules, validated `data/transactions.csv`, and rendered the validation results to the Data Docs site on port `8081` as successful.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/18615848-b0c1-463c-8ea4-a217cfc17238" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6ac8214d-fd3f-4e4d-9775-3ed8b94e2157" />
<img width="50" height="300" alt="image" src="https://github.com/user-attachments/assets/d4e25455-82cb-425d-b194-43ba6458d0cf" />



