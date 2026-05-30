# Day 14 — Reproducible ML Pipelines with DVC

The xFusionCorp Industries ML team uses DVC pipelines to keep data processing reproducible. A draft `dvc.yaml` exists in the `fraud-detection` project, but `dvc repro` does not complete the full pipeline. The goal is to correct the pipeline definition so it runs cleanly end to end.

The project exists at `/root/code/fraud-detection/` with DVC initialized. Python scripts are at `src/data/process_data.py` and `src/data/split_data.py`; raw input is at `data/raw/transactions.csv`.

The corrected pipeline must declare two stages:
1. **process_data** – Depends on `data/raw/transactions.csv` and `src/data/process_data.py`; produces `data/processed/clean_transactions.csv`.
2. **split_data** – Depends on `data/processed/clean_transactions.csv` and `src/data/split_data.py`; produces `data/processed/train.csv` and `data/processed/test.csv`.

3. Review the existing dvc.yaml and correct everything that prevents dvc repro from completing.

4. After your changes, dvc repro must run end to end and dvc status must report no stale stages.

| Once the pipeline is valid, the DVC extension's PIPELINES section under the DVC view will list both stages and visualise the dependency graph between them.

## 📋 Task Summary

Correct the `dvc.yaml` file to ensure:
- Both stages are defined with correct dependencies and outputs.
- `dvc repro` completes successfully.
- `dvc status` reports no stale stages.
- The pipeline visualization shows the correct dependency flow.

---

## 🛠️ Step 1: Update the `dvc.yaml` Pipeline

Create or update the `dvc.yaml` file in the root of the project with the correct stage definitions.

```yaml
stages:
  process_data:
    cmd: python src/data/process_data.py
    deps:
      - data/raw/transactions.csv
      - src/data/process_data.py
    outs:
      - data/processed/clean_transactions.csv

  split_data:
    cmd: python src/data/split_data.py
    deps:
      - data/processed/clean_transactions.csv
      - src/data/split_data.py
    outs:
      - data/processed/train.csv
      - data/processed/test.csv
```

---

## 🛠️ Step 2: Run the Pipeline

Execute the pipeline to ensure all stages run correctly and produce the expected outputs.

```bash
cd /root/code/fraud-detection

# Run the pipeline
dvc repro
```

### Expected Output
DVC will detect the stages and execute the commands in order:
1. `python src/data/process_data.py`
2. `python src/data/split_data.py`

---

## 🛠️ Step 3: Verify Pipeline Status

Check the status to ensure the pipeline is up to date and no stages are stale.

```bash
dvc status
```

### Expected Output
```text
Data and pipelines are up to date.
```

---

## ✅ Step 4: Visualize the Pipeline (Optional)

You can visualize the dependency graph using the DVC extension or the CLI.

```bash
dvc dag
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **`dvc.yaml`** | The file where DVC pipeline stages are defined. |
| **Stages** | Individual steps in a pipeline, each with its own command, dependencies, and outputs. |
| **`deps` (Dependencies)** | Files or directories that a stage depends on. If these change, the stage is considered stale. |
| **`outs` (Outputs)** | Files or directories produced by a stage. DVC tracks these automatically. |
| **`dvc repro`** | Reproduces the pipeline by running stale stages in the correct order. |
| **DAG (Directed Acyclic Graph)** | The structure formed by the dependencies between stages, ensuring a logical execution flow. |

---

## ⚠️ Common Pitfalls

1. **Incorrect Paths** — Ensure paths in `deps` and `outs` are relative to the project root and match the script logic.
2. **Missing Dependencies** — Forgetting to include the Python script itself in `deps` means the stage won't re-run if the code changes.
3. **Overlapping Outputs** — Multiple stages should not produce the same output file, as this creates ambiguity in the DAG.
4. **Manual Output Modification** — Never manually edit files in `outs`; they should only be modified by the stage command to maintain reproducibility.

### Screenshots

<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6ea70e4c-8d9c-4544-8297-ee2a6f0b6716" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/2e1945f2-5db8-47b6-8af6-71de9b8fdd09" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/aa2f967f-8e04-4347-b6dd-7f6797514735" />



