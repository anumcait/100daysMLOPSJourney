# Day 11 — Track a Dataset with DVC

Day 11: Track a Dataset with DVC kodekloud 100days mlops A teammate has added the transactions dataset to the xFusionCorp Industries fraud-detection repository, but it was committed directly to Git instead of being tracked with DVC. Bring the repository in line with the team standard—every dataset under data/ must be tracked by DVC, not by Git.


A project exists at /root/code/fraud-detection/ with DVC already initialised. The dataset data/raw/transactions.csv is currently tracked by Git, and the team standard requires DVC to own it instead.

Stop Git from tracking the dataset without deleting it from disk.

Track the same dataset with DVC so a .dvc pointer file is produced and data/raw/.gitignore excludes the dataset itself.

Stage the new .dvc pointer and the new .gitignore, then record a Git commit with the message Track transactions dataset with DVC.

Once tracking is moved to DVC, the DVC TRACKED section in the EXPLORER panel will list the dataset, confirming the extension recognises it as a DVC-managed file.


## 📋 Task Summary

A teammate has added the `transactions.csv` dataset to the `fraud-detection` repository, but it was committed directly to Git instead of being tracked with DVC. The team standard requires every dataset under `data/` to be tracked by DVC. The goal is to move the dataset from Git control to DVC control without deleting it from disk.


---

## 🛠️ Step 1: Stop Git from Tracking the Dataset

Remove the file from Git's tracking (the index) while keeping it physically on the disk.

```bash
cd /root/code/fraud-detection/

git rm --cached data/raw/transactions.csv
```

---

## 🛠️ Step 2: Track the Dataset with DVC

Use DVC to take ownership of the file. This creates a `.dvc` pointer file and updates `.gitignore`.

```bash
dvc add data/raw/transactions.csv
```

---

## 🛠️ Step 3: Stage and Commit Changes to Git

Stage the new `.dvc` pointer file and the updated `.gitignore` file, then record the change in Git.

```bash
git add data/raw/transactions.csv.dvc data/raw/.gitignore

git commit -m "Track transactions dataset with DVC"
```

---

## ✅ Step 4: Verify the Setup

Check that the dataset is now ignored by Git but tracked by DVC.

```bash
# Verify the .dvc file exists
ls data/raw/transactions.csv.dvc

# Verify git status (should not show transactions.csv)
git status
```

---

## 📂 Final Project Structure

```
/root/code/fraud-detection/
├── data/
│   └── raw/
│       ├── transactions.csv       # Actual data (excluded by Git)
│       ├── transactions.csv.dvc   # DVC pointer file (tracked by Git)
│       └── .gitignore             # Automatically updated to ignore transactions.csv
├── .dvc/
└── ...
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| `git rm --cached` | Removes a file from the Git index but keeps it on the local filesystem |
| `dvc add` | Tells DVC to track a file, moves it to cache, and creates a `.dvc` pointer |
| `.dvc` File | A small text file containing the hash of the original data; tracked by Git |
| DVC and `.gitignore` | `dvc add` automatically adds the data file to the local `.gitignore` |
| Dataset Ownership | Moving data from Git to DVC keeps the repository lightweight |

---

## ⚠️ Common Pitfalls

1. **Deleting the file** — Using `git rm` without `--cached` will delete the dataset from disk.
2. **Forgetting `.gitignore`** — If you don't commit the updated `.gitignore`, other teammates might accidentally commit the data back to Git.
3. **Not committing the `.dvc` file** — Teammates won't be able to `dvc pull` the data if the pointer file is missing from Git.
4. **Committing both** — Ensure the actual data file is no longer tracked by Git before adding it to DVC.

### Screenshots

<img width="500" height="590" alt="image" src="https://github.com/user-attachments/assets/7ba95b2b-cec1-47da-9d6d-3d49ffa248e6" />
<img width="500" height="591" alt="image" src="https://github.com/user-attachments/assets/6262928b-6dba-4934-b0a5-95ad92eebd3e" />


