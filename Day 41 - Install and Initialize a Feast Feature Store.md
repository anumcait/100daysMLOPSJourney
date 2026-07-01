# Day 41 - Install and Initialize a Feast Feature Store

The xFusionCorp Industries ML platform team is adopting Feast as the feature store for their fraud-detection workflow. The first step is to scaffold a working feature repository with the Feast CLI, apply the starter definitions to the local registry, and confirm everything loads in the Feast UI. Your task is to initialise a new feature repository under /root/code/, apply the registry, and verify the empty project loads in the Feast UI.


Feast is already installed in the lab image. feast version can be run in the terminal to confirm.

The target project layout:

/root/code/feature_repo/feature_repo/feature_store.yaml – The feast init scaffold config (provider, registry, online/offline stores).
/root/code/feature_repo/feature_repo/data/registry.db – Written by feast apply from the repo root.
/root/code/feature_repo/feature_repo/feature_definitions.py – The starter feature definitions Feast ships with the scaffold.
The end state must include:

The /root/code/feature_repo/feature_repo/ directory is populated with the feast init scaffold.
feature_store.yaml parses as valid YAML and carries the project, provider, and registry keys.
data/registry.db exists – feast apply completed without error.
feast version exits zero.
The Feast UI button at the top of the lab opens a responsive dashboard that lists the scaffold's project.
feast ui is a long-running process; run it in a second VS Code terminal (or append & to the command) so the shell remains usable. The UI loads the registry at start-up—start the UI after feast apply has written registry.db.


### Objective

The xFusionCorp Industries ML platform team is adopting Feast as the feature store for their fraud-detection workflow. The first step is to scaffold a working feature repository with the Feast CLI, apply the starter definitions to the local registry, and confirm everything loads in the Feast UI. Your task is to initialise a new feature repository under `/root/code/`, apply the registry, and verify the empty project loads in the Feast UI.

Feast is already installed in the lab image. `feast version` can be run in the terminal to confirm.

The target project layout:
- `/root/code/feature_repo/feature_repo/feature_store.yaml` – The feast init scaffold config (provider, registry, online/offline stores).
- `/root/code/feature_repo/feature_repo/data/registry.db` – Written by feast apply from the repo root.
- `/root/code/feature_repo/feature_repo/feature_definitions.py` – The starter feature definitions Feast ships with the scaffold.

The end state must include:
- The `/root/code/feature_repo/feature_repo/` directory is populated with the feast init scaffold.
- `feature_store.yaml` parses as valid YAML and carries the project, provider, and registry keys.
- `data/registry.db` exists – `feast apply` completed without error.
- `feast version` exits zero.
- The Feast UI button at the top of the lab opens a responsive dashboard that lists the scaffold's project.
- `feast ui` is a long-running process; run it in a second VS Code terminal (or append `&` to the command) so the shell remains usable. The UI loads the registry at start-up—start the UI after `feast apply` has written `registry.db`.

## Objective
Scaffold a Feast feature repository, apply the starter feature definitions to build a local feature registry, and serve the Feast UI dashboard.

## Task
- **Verify Feast Installation**: Confirm that Feast is installed and check its configuration using `feast version`.
- **Initialize the Feature Repo**: Scaffold the feature store repository structure under `/root/code/` with the Feast CLI.
- **Apply Feast Schema Definitions**: Compile the boilerplate feature registry using `feast apply` to generate the SQLite registry database.
- **Launch the Feast UI**: Serve the dashboard in the background and confirm it displays the scaffolded project.

## Solution

### Step 1: Verify Feast Installation
Run `feast version` to ensure the Feast CLI tool is globally available in the lab environment:
```bash
feast version
```
Expected output:
```
Feast SDK Version: "0.xx.x" (or similar)
```

### Step 2: Initialize the Repository
Navigate to `/root/code/` and create the new feature repository directory:
```bash
cd /root/code
feast init feature_repo
```
This commands scaffolds a project template with the following layout:
```
feature_repo/
└── feature_repo/
    ├── data/
    ├── feature_definitions.py
    └── feature_store.yaml
```

### Step 3: Inspect the Configuration
Open `/root/code/feature_repo/feature_repo/feature_store.yaml` to verify the pre-configured parameters:
```yaml
project: feature_repo
registry: data/registry.db
provider: local
online_store:
  type: sqlite
  path: data/online_store.db
```

### Step 4: Apply Schema Definitions
Navigate into the repository directory containing `feature_store.yaml` and initialize the local SQLite registry database:
```bash
cd /root/code/feature_repo/feature_repo
feast apply
```
Expected output shows the successful registration of the feature views and entities:
```
Created entity driver
Created feature view driver_hourly_stats
Created feature view driver_hourly_stats_fresh
Created sqlite table feature_repo_driver_hourly_stats
Created sqlite table feature_repo_driver_hourly_stats_fresh
```
This creates the `/root/code/feature_repo/feature_repo/data/registry.db` database.

### Step 5: Start the Feast UI Dashboard
Launch the dashboard server in the background so the console remains usable. By default, it runs on port 8888:
```bash
feast ui &
```
Open a browser or click the Feast UI button in the workspace interface to view the dashboard and verify the `feature_repo` project and its features/entities are registered and visible.

## Key Concepts

| Concept | Detail |
|---------|--------|
| **Feast Feature Store** | An operational data system that manages and serves machine learning features for both training (offline via files/databases) and inference (online via low-latency key-value stores). |
| **`feast init`** | CLI command that scaffolds a boilerplate Feast repository with a default provider config (`feature_store.yaml`), model definitions (`feature_definitions.py`), and storage folders. |
| **`feast apply`** | Scans the repo for Python files containing feature/entity definitions, builds the local registry database (`registry.db`), and prepares the infrastructure (e.g., SQLite tables) to serve features. |
| **`feature_store.yaml`** | The central configuration file defining the Feast project name, metadata registry path, offline store engine (e.g. File/Parquet), and online store engine (e.g. SQLite/Redis). |
| **Feast Registry (`registry.db`)** | A metadata catalog stored locally in SQLite/S3/GCS that tracks entities, feature views, schemas, and configurations. It acts as the single source of truth for Feast. |
| **Feast UI** | A built-in web dashboard served locally that provides documentation, search, and visibility into registered entities, feature views, and source datasets. |

## Summary
By running `feast init feature_repo` under `/root/code`, the team successfully scaffolded a structured Feast feature repository. Using `feast apply` within the inner directory compiled the starter feature definitions, registering entities and feature views in a local SQLite database (`data/registry.db`). Finally, launching `feast ui &` exposed a local web interface that reads this registry, establishing Feast as the operational telemetry layer for managing fraud detection features.

### Screenshots
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/de3c6362-dd95-4910-91ae-6d490e118544" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/6304d260-cff8-4283-8797-353e7af265b3" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/3638185c-fed1-4aec-9e86-4957b48aa818" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/b85fa47d-e17b-4ccc-9f77-0588dda55094" />
<img width="500" height="300" alt="image" src="https://github.com/user-attachments/assets/7452b571-94fe-4c2f-adad-89d139c78162" />





