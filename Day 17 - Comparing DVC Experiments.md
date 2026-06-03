# Day 17 — Comparing DVC Experiments

The xFusionCorp Industries data science team compares multiple training runs with different hyperparameters using DVC experiments. Run three experiments that vary the n_estimators hyperparameter, identify the best-performing one, and promote it to the tracked workspace.

A project exists at /root/code/fraud-detection/ with a parameterised DVC pipeline already in place. params.yaml contains n_estimators: 100 and the baseline pipeline has been run once.

Run three DVC experiments, each with a different value for n_estimators across a reasonable range (for example 50, 200, and 500). Each experiment should produce a fresh metrics.json.

Compare the experiments and choose the one whose f1_score is the highest.

Apply the chosen experiment to the workspace so its n_estimators, metrics.json, and models/model.pkl become the tracked state.

The DVC extension's EXPERIMENTS section under the DVC view lists every experiment alongside its parameters and metrics, supports running fresh experiments through the + action, and applies a selected experiment to the workspace from the right-click menu—every operation in this lab can be performed either through the extension UI or with the equivalent dvc exp commands.

## 📋 Task Summary

Optimise the `fraud-detection` model by comparing different hyperparameter settings:
- Run three experiments with varying `n_estimators` (50, 200, and 500).
- Compare the performance (especially `f1_score`) of all runs.
- Identify the best-performing experiment.
- Apply the best experiment's state to the workspace.

---

## 🛠️ Step 1: Run Multiple DVC Experiments

Use the `dvc exp run` command with the `--set-param` flag to override the default values in `params.yaml` without manually editing the file each time.

```bash
cd /root/code/fraud-detection

# Experiment 1: n_estimators = 50
dvc exp run --set-param train.n_estimators=50

# Experiment 2: n_estimators = 200
dvc exp run --set-param train.n_estimators=200

# Experiment 3: n_estimators = 500
dvc exp run --set-param train.n_estimators=500
```

---

## 🛠️ Step 2: Compare Experiment Results

Display a table of all experiments, their parameters, and the resulting metrics to identify the winner.

```bash
# Show all experiments
dvc exp show
```

### Expected Table Output
| Experiment | Created | train.n_estimators | accuracy | f1_score |
|------------|---------|--------------------|----------|----------|
| workspace  | -       | 100                | 0.942    | 0.921    |
| commit     | 10:45 AM| 100                | 0.942    | 0.921    |
| [exp-a1b2] | 10:50 AM| 50                 | 0.931    | 0.910    |
| [exp-c3d4] | 10:55 AM| 200                | 0.955    | 0.948    |
| [exp-e5f6] | 11:00 AM| 500                | 0.952    | 0.941    |

*In this example, **exp-c3d4** (n_estimators=200) has the highest F1-score (0.948).*

---

## 🛠️ Step 3: Apply the Winning Experiment

Once the best experiment is identified, promote it to your workspace. This updates `params.yaml`, `metrics.json`, and the model files to match that specific run.

```bash
# Apply the experiment (replace <exp_id> with your best run's ID)
dvc exp apply exp-c3d4
```

---

## 🛠️ Step 4: Persist the Changes

After applying the experiment, stage the updated files and commit them to Git to make them the new baseline.

```bash
git add params.yaml metrics.json dvc.lock
git commit -m "Optimize model: set n_estimators to 200 (best F1-score)"
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **DVC Experiments** | A hidden "shadow" track in Git that stores runs without cluttering the branch history. |
| **`dvc exp run`** | Executes the pipeline and records the run as an experiment. |
| **`--set-param`** | CLI flag to temporarily override `params.yaml` values for a specific run. |
| **`dvc exp show`** | Generates a comparison dashboard in the terminal. |
| **`dvc exp apply`** | Pulls the parameters and artifacts of a specific experiment back into the current workspace. |

---

## ⚠️ Common Pitfalls

1.  **Forgetting to Apply** — If you run 20 experiments but don't `dvc exp apply`, your workspace still reflects the old baseline.
2.  **Experiment Pollution** — If you don't commit your baseline before starting experiments, DVC might not have a clean "parent" commit to compare against.
3.  **Large Models** — Remember that every experiment produces artifacts. Use `dvc exp gc` occasionally to clean up failed or useless experiment data if storage becomes an issue.
