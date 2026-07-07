# Day 47 - Debug a Failing Great Expectations Checkpoint

The xFusionCorp Industries ML platform team extended the fraud_schema suite to a second batch—data/transactions_drifted.csv, a week's worth of real production rows. The drift_check checkpoint runs the existing suite against this file and fails on its first run. Your task is to use Data Docs to diagnose which expectation failed and why, widen the offending bound in the fix-script, re-run the checkpoint, and confirm Data Docs goes green.

The end state must include:
- The `drift_check` checkpoint is still present in `gx/checkpoints/`.
- `gx/expectations/fraud_schema.json` still has all four core expectation types (the fix is a widening, not a deletion).
- The most recent validation JSON under `gx/uncommitted/validations/` for checkpoint `drift_check` reports `success: true`.

The failing-validation page is the core debug surface for data-quality incidents: it tells you WHICH expectation failed, WHAT was observed, and by how much. A real team uses that same signal to decide whether the data genuinely drifted (update the rule) or whether the data is broken (fix upstream). Either way, the read-the-evidence step comes first.

## Objective
Identify and resolve data quality anomalies caused by data drift in production datasets using Great Expectations. Locate the failing metric assertion via Data Docs, modify bounds in the schema declaration config to accommodate drifted distributions with safe margins, and execute validations to verify compliance.

## Task
- **Diagnose Validation Failures**: Access Data Docs on port `8081` to investigate the unsuccessful validation run under the `drift_check` checkpoint.
- **Widen Threshold Constraints**: Locate `/root/code/dataquality/fix_drift.py` and modify the failing expectation bounds (replacing the non-negative limit `min_value=0` with `min_value=-400` to handle negative transaction values with safe headroom).
- **Run the Realignment Script**: Execute the fix script to update expectations metadata cache and re-validate transactions.
- **Ensure Green Verification**: Verify that the latest validation log in Data Docs indicates Success for all four criteria.

## Solution

### Step 1: Access Data Docs
1. Click the **Data Docs** shortcut or navigate to `http://localhost:8081` in your browser.
2. Select the latest failed validation run matching the `drift_check` checkpoint under the `fraud_schema` suite.

### Step 2: Analyze the Validation Error
1. In the validation report, observe the column marked failed:
   * **Column**: `amount`
   * **Expectation**: values must be $\ge$ 0
   * **Observed**: 12 negative values, with the lowest value recorded as `-347.22`

### Step 3: Edit the Drift Fix Script
1. Open active file `/root/code/dataquality/fix_drift.py` in your VS Code editor.
2. Locate the expectation configuration mapping for the `amount` column:
   ```python
   suite.add_expectation(
       ge.ExpectColumnValuesToBeBetween(
           column="amount",
           min_value=0,
       )
   )
   ```
3. Update the lower bound (`min_value`) to `-400` to accommodate negative drift attributes while maintaining safety margins:
   ```python
   suite.add_expectation(
       ge.ExpectColumnValuesToBeBetween(
           column="amount",
           min_value=-400,
       )
   )
   ```
4. Save the script edits.

### Step 4: Run the Realignment Pipeline
1. Run the script from the terminal to persist modifications to `gx/expectations/fraud_schema.json` and execute validation checks:
   ```bash
   python3 /root/code/dataquality/fix_drift.py
   ```
   **Expected output:**
   ```
   Persisted 4 expectations to fraud_schema
   Checkpoint drift_check result: success=True
   ```

### Step 5: Confirm Update Status in Data Docs
1. Refresh the Data Docs index at `http://localhost:8081`.
2. Confirm the newest `drift_check` run is marked **Success** (green) and all four expectations pass.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Data Drift** | The changes over time in the statistical properties of features feeding a model, impacting predictive reliability or triggering telemetry checks. |
| **Bound Widening** | Modifying assertion limits to reflect altered production ranges safely rather than deleting the expectation block entirely. |
| **Observation Parsing** | Checking metrics validation output values (such as range violations) to identify precise scaling properties for model boundaries. |
| **Pipeline Gates** | Using validation checkpoint structures to monitor drifted attributes before updating downline target directories. |

## Summary
The Great Expectations `drift_check` checkpoint was failing when validating `data/transactions_drifted.csv` because the transaction `amount` column dataset contained negative parameters down to `-347.22`, violating the existing $\ge 0$ schema constraint. By diagnosing the error through Data Docs on port `8081` and editing `/root/code/dataquality/fix_drift.py` to change the `min_value` to `-400` (widening the threshold with adequate headroom), the validation parameters were realigned. Running the script saved the new rules to `gx/expectations/fraud_schema.json`, ran the validation check successfully, and transitioned the Data Docs status to Success.
