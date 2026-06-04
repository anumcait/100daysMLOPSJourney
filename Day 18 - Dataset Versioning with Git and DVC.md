# Day 18 — Dataset Versioning with Git and DVC

The xFusionCorp Industries ML team keeps different dataset and model versions on different Git branches so that the team can roll between versions cleanly. Tag the current state as v1.0, produce a v2-improved branch based on a newer dataset, and confirm that switching back restores the original data.


A project exists at /root/code/fraud-detection/ with a working DVC pipeline and the baseline data/raw/transactions.csv already tracked.

An improved dataset has been pre-staged at /root/code/fraud-detection/data/raw/transactions_v2.csv and is visible in the file explorer. Do not delete this file.

On the main branch, tag the current state as v1.0.

Create a new branch named v2-improved. Replace the tracked dataset with the contents of the v2 file, re-track it with DVC, re-run the pipeline, and commit the changes.

Switch back to the main branch and use dvc checkout to restore the v1 dataset on disk. The restored content must match the hash recorded by the v1.0 tag.

The DVC extension's DVC TRACKED section in the EXPLORER panel will reflect the current branch's tracked state—it should show different dataset hashes on main and v2-improved.

## 📋 Task Summary

Optimise the `fraud-detection` project flow by versioning data alongside code:
- Tag the current `main` branch state as `v1.0`.
- Create a new branch `v2-improved`.
- Replace the tracked dataset with `transactions_v2.csv`.
- Re-track the dataset with DVC and re-run the pipeline.
- Commit the changes to `v2-improved`.
- Switch back to `main` and verify the data restoration.

---

## 🛠️ Step 1: Tag the Baseline Version

First, ensure the current state of the `main` branch is tagged so we can always return to this exact dataset and model combination.

```bash
cd /root/code/fraud-detection

# Tag the current commit
git tag -a v1.0 -m "Baseline dataset (v1) and model"
```

---

## 🛠️ Step 2: Create an Improvement Branch

Create a new branch where we will incorporate the improved dataset.

```bash
# Create and switch to the new branch
git checkout -b v2-improved
```

---

## 🛠️ Step 3: Update the Dataset and Pipeline

Replace the existing `transactions.csv` with the content from the pre-staged `transactions_v2.csv`, update the DVC tracking, and re-run the pipeline to generate a new model.

```bash
# Replace the old dataset with the improved version
cp data/raw/transactions_v2.csv data/raw/transactions.csv

# Update DVC tracking (this updates the .dvc file with the new hash)
dvc add data/raw/transactions.csv

# Re-run the pipeline to update dvc.lock and model.pkl
dvc repro

# Commit the changes to Git
git add data/raw/transactions.csv.dvc dvc.lock
git commit -m "Update dataset to v2 and retrain model"
```

---

## 🛠️ Step 4: Switch Back and Restore Data

Verify that Git and DVC work in tandem to restore the original data when switching branches.

```bash
# Switch back to the main branch
git checkout main

# Use DVC to sync the data on disk with the metadata in the current branch
dvc checkout
```

---

## 🛠️ Step 5: Verification

The DVC extension's **DVC TRACKED** section in the EXPLORER panel (or `dvc status`) will reflect the current branch's tracked state. It should show different dataset hashes on `main` and `v2-improved`.

```bash
# Verify the hash in the .dvc file
cat data/raw/transactions.csv.dvc
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| **`git tag`** | Marks a specific point in Git history as important (e.g., a release). |
| **`dvc add`** | Updates the `.dvc` file with the new data hash and updates the local cache. |
| **`dvc repro`** | Reproduces the pipeline, ensuring `dvc.lock` is consistent with the new data. |
| **`dvc checkout`** | Restores the versions of data files tracked by DVC into the workspace based on the current `.dvc` files. |
| **Data Immutability** | DVC ensures that different versions of the same file path are stored uniquely in the cache. |

---

## ⚠️ Common Pitfalls

1.  **Forgetting `dvc checkout`** — Git only restores the `.dvc` pointer files. You must run `dvc checkout` to actually swap the large files on your disk.
2.  **Deleting Source Files** — When replacing data, ensure you don't delete files that DVC is currently tracking until you've successfully committed the new state.
3.  **Hash Mismatch** — If you manually edit a `.dvc` file, DVC will report a hash mismatch. Always use `dvc add` or `dvc repro` to update metadata.
