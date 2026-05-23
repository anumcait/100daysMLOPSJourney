# Day 10 — Install and Initialize DVC in an ML Project

## 📋 Task Summary

The xFusionCorp Industries ML team is adopting **DVC (Data Version Control)** so that datasets and model files are versioned separately from code. Initialise DVC inside the existing Git repository at `/root/code/fraud-detection/` and record the initialisation in Git.

---

## 🛠️ Step 1: Navigate to the Project & Initialize DVC

```bash
cd /root/code/fraud-detection/

dvc init
```

This creates:

| File/Directory | Purpose |
|----------------|---------|
| `.dvc/` | DVC control directory (internal config, cache, tmp) |
| `.dvc/.gitignore` | Prevents DVC internals (cache, tmp) from being committed to Git |
| `.dvc/config` | DVC configuration file (remote storage settings, etc.) |
| `.dvcignore` | Works like `.gitignore` but for DVC — tells DVC which files to ignore |

---

## 🛠️ Step 2: Stage and Commit DVC Files

```bash
git add .dvc/ .dvcignore

git commit -m "Initialize DVC"
```

---

## ✅ Step 3: Verify the Setup

```bash
git log --oneline -1
ls -la
```

Expected `git log` output:

```
<hash> Initialize DVC
```

The `ls -la` output should show `.dvc/` and `.dvcignore` alongside the existing `.git/` directory.

---

## 📂 Final Project Structure

```
/root/code/fraud-detection/
├── .dvc/
│   ├── .gitignore      # Excludes cache/, tmp/ from Git
│   └── config          # DVC configuration (empty initially)
├── .dvcignore          # DVC's ignore patterns
├── .git/               # Existing Git directory
└── ...                 # Existing project files
```

---

## 🔄 Complete Commands (Quick Reference)

```bash
cd /root/code/fraud-detection/
dvc init
git add .dvc/ .dvcignore
git commit -m "Initialize DVC"
```

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| DVC | Data Version Control — a Git-like tool for versioning data and models |
| `dvc init` | Initializes DVC in an existing Git repo, creating `.dvc/` and `.dvcignore` |
| `.dvc/` directory | Internal DVC control directory (config, cache, tmp) |
| `.dvc/config` | Stores DVC configuration such as remote storage settings |
| `.dvc/.gitignore` | Automatically excludes DVC internals (cache, tmp) from Git |
| `.dvcignore` | Tells DVC which files/directories to ignore (similar to `.gitignore`) |
| Code vs Data versioning | Git tracks code + DVC metafiles; DVC tracks large data/model files |

---

## ⚠️ Common Pitfalls

1. **Running `dvc init` outside a Git repo** — DVC requires an existing Git repository
2. **Forgetting to `git add .dvc`** — The `.dvc/` directory contents must be committed to Git
3. **Running `dvc init` twice** — Will error; use `dvc init --force` to re-initialize
4. **Confusing `.dvcignore` with `.gitignore`** — `.dvcignore` is for DVC operations, `.gitignore` is for Git
5. **Committing DVC cache** — The `.dvc/.gitignore` prevents this, but don't remove it
