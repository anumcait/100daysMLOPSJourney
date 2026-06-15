# Day 28: Fix a Broken MLflow Project and Re-Run It
The xFusionCorp Industries ML platform team packages their training runs as MLflow Projects so any engineer can reproduce them with a single mlflow run invocation. A project has been pre-staged at /root/code/trainer/, but the first run from the lab startup already failed—the MLproject file carries a subtle command-line bug. Your task is to fix the bug and then run the project end to end twice so successful runs are recorded.


The MLflow tracking server is already running on port 5000. The MLflow UI button at the top of the lab can be opened to view the dashboard; the trainer experiment already contains a FAILED run from the automated run that the lab startup triggered against the broken project.

/root/code/trainer/ contains:

MLproject – The project descriptor (this file has the bug).
train.py – The trainer entry point. This file is correct and must not be modified.
The fix is confined to MLproject. Once repaired, two successful mlflow run invocations must be recorded in the trainer experiment:

One explicit call: mlflow run . -e train -P n_estimators=200 -P max_depth=10 --env-manager=local (run from /root/code/trainer).
One default call: mlflow run . -e train --env-manager=local.
The end state must include:

The MLproject command line invokes train.py with flag names that match train.py's argparse declarations.
The trainer experiment contains at least two FINISHED runs whose params.n_estimators values differ (one at 200, one at the default 100).
The original FAILED run from the startup is still present – It must not be deleted, because the lab's diagnosis trail depends on it.
The failed run's page in the MLflow UI and /tmp/mlflow-run-initial.log both carry the underlying error. Running mlflow run . manually from /root/code/trainer reproduces the same error directly in the terminal.

## Task Description

Identify and fix the command-line interface mismatch in the `MLproject` file to make the project fully reproducible.

### Requirements:

1. **Fix the MLproject file**: Correct the `command` line in the `MLproject` file to match the `argparse` declarations in `train.py`.
2. **Execute Explicit Run**: Run the training with specific parameters: `n_estimators=200` and `max_depth=10`.
3. **Execute Default Run**: Run the training using default parameters defined in the `MLproject` file.
4. **Environment**: Use the `--env-manager=local` flag for all runs.
5. **Validation**: Ensure two new successful runs are recorded in the MLflow tracking server without deleting the original failed run.

## Solution

### Implementation Steps

The failure is caused by a flag name mismatch between the `MLproject` command string and the training script's expected arguments.

#### Step 1: Identify the Bug
Inspecting the `MLproject` file revealed that the command line was using incorrect flag formats or mismatched parameter placeholders (e.g., `--n-estimators` instead of `--n_estimators`).

#### Step 2: Fix the MLproject File
Update the `/root/code/trainer/MLproject` file with the corrected command mapping:

```yaml
name: trainer

entry_points:
  train:
    parameters:
      n_estimators: {type: int, default: 100}
      max_depth: {type: int, default: 5}
    command: "python train.py --n_estimators {n_estimators} --max_depth {max_depth}"
```

#### Step 3: Run the Reproducible Project
Execute the project twice from the `/root/code/trainer` directory.

**1. Explicit Run:**
```bash
mlflow run . -e train -P n_estimators=200 -P max_depth=10 --env-manager=local
```

**2. Default Run:**
```bash
mlflow run . -e train --env-manager=local
```

### Verification

After execution, the MLflow UI for the `trainer` experiment should show:
- **1 FAILED run**: The initial pre-existing run (preserved for diagnostic history).
- **2 FINISHED runs**:
    - One run with `n_estimators = 200`.
    - One run with `n_estimators = 100` (the default).

## Key Concepts

| Concept | Description |
| :--- | :--- |
| **MLflow Project** | A convention for organizing and describing machine learning code so it can be run on any platform. |
| **MLproject File** | A YAML file that defines the entry points, parameters, and environment for an MLflow Project. |
| **Parameter Propagation** | How parameters passed via the CLI (`-P`) are injected into the execution command defined in the `MLproject` file. |
| **Reproducibility** | The ability to re-run an experiment and achieve consistent results by standardizing the execution environment and command line interface. |
