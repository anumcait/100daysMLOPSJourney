# MLOps: Comprehensive Day-Wise Study Notes

This document provides full MLOps notes with day-wise explanations right out of the box. Designed at a student level, everything is recorded here without leaving any corner untouched. It serves as an ongoing, exhaustive log of the MLOps journey, meticulously detailing every task, theoretical concept, and the full step-by-step execution information.

---

## 📅 Day 1: Create a Python Virtual Environment for ML

### Task Description
The team needs a standardized Python environment for a new ML project. The goal is to set up a virtual environment with required ML libraries (numpy, pandas, scikit-learn, matplotlib) and generate a `requirements.txt` file.

### Concept Summary
A **Virtual Environment** is an isolated workspace for a Python project. Instead of installing software globally on your computer (which can cause version conflicts between different projects), a virtual environment keeps the project's dependencies entirely separated.

### Step-by-Step Execution
**Step 1: Create a dedicated directory**
We first create an organized folder to house our code and environments.
```bash
mkdir -p /root/code/
```

**Step 2: Initialize the Virtual Environment**
Using Python's built-in `venv` module, we generate the isolated environment named `ml-env`.
```bash
python3 -m venv /root/code/ml-env
```

**Step 3: Activate the Environment**
Before installing packages, we must activate the environment so our system knows to install them locally, not globally.
```bash
source /root/code/ml-env/bin/activate
```

**Step 4: Install Required ML Libraries**
We install the specific packages needed for our machine learning tasks using `pip`.
```bash
pip install numpy pandas scikit-learn matplotlib
```

**Step 5: Lock Dependencies**
We generate a `requirements.txt` file. This acts as a blueprint so anyone else can recreate this exact environment.
```bash
pip freeze > /root/code/requirements.txt
```

---

## 📅 Day 2: Set Up and Configure Jupyter Notebook Server

### Task Description
Configure an existing JupyterLab server to listen on port `8888`, bind to all IPs (`0.0.0.0`), set the root directory to `/root/notebooks/`, and open the classic notebook interface by default.

### Concept Summary
**Jupyter Notebook** is a web-based interactive development environment. Data scientists use it to write code, execute it sequentially, and visualize data instantly. Configuring the server correctly ensures it can be accessed securely across networks.

### Step-by-Step Execution
**Step 1: Edit the Configuration File**
We update the Jupyter settings file (`/root/code/jupyter_lab_config.py`) to properly expose the server and set default behaviors.
*Modifications made:*
- `c.ServerApp.ip = '0.0.0.0'` (Allows external connections)
- `c.ServerApp.port = 8888` (Defines the port)
- `c.ServerApp.root_dir = '/root/notebooks/'` (Sets the default working directory)
- `c.ServerApp.default_url = '/tree'` (Forces the classic notebook UI)

**Step 2: Create the Root Directory**
Ensure the folder designated in our configuration actually exists.
```bash
mkdir -p /root/notebooks/
```

**Step 3: Start the Server**
Activate the environment and launch JupyterLab using our custom configuration file.
```bash
source /root/code/ml-env/bin/activate
jupyter lab --config=/root/code/jupyter_lab_config.py --allow-root --no-browser &
```

---

## 📅 Day 3: Dependency Management (Lockfiles)

### Task Description
A teammate left a broken `requirements.in` file. We must correct the file with proper version constraints and compile it into a pinned lockfile using the `uv` tool.

### Concept Summary
A **Lockfile** is a stringent version of a requirements file. It records the *exact* versions of every package and sub-package your project uses. This entirely eliminates the "it works on my machine" problem by guaranteeing identical setups across all developers.

### Step-by-Step Execution
**Step 1: Fix the Requirements Input File**
Open `/root/code/fraud-detection/requirements.in` and explicitly state the minimum acceptable versions for our core libraries:
```text
scikit-learn>=1.3.0
mlflow>=2.0.0
pandas>=2.1.0
numpy>=1.26.0
```

**Step 2: Compile the Lockfile**
Use the `uv` package manager to compile the `.in` file into a highly detailed `.txt` lockfile. `uv` is preferred because it resolves dependencies blazingly fast.
```bash
cd /root/code/fraud-detection/
uv pip compile requirements.in -o requirements.txt
```

---

## 📅 Day 4: Standard ML Project Structure

### Task Description
Bring a messy new ML project into compliance with team conventions by constructing a strict, standardized directory layout.

### Concept Summary
A **Standard Project Structure** organizes a codebase logically. By separating raw data, processed data, source code, models, and configurations, the project becomes immediately readable and manageable for any new team member.

### Step-by-Step Execution
**Step 1: Create the Standard Folders**
We generate all the required overarching directories.
```bash
cd /root/code/fraud-detection
mkdir -p data/raw data/processed models notebooks src/data src/features src/models src/utils tests configs
```

**Step 2: Initialize Python Packages**
We add `__init__.py` to all `src` subdirectories. This tells Python to treat these folders as importable modules.
```bash
touch src/data/__init__.py
touch src/features/__init__.py
touch src/models/__init__.py
touch src/utils/__init__.py
```

**Step 3: Create Core Documentation and Specs**
Initialize standard files like `README.md` and `requirements.txt`.
```bash
cat > requirements.txt <<EOF
scikit-learn
pandas
numpy
mlflow
EOF

cat > README.md <<EOF
# fraud-detection
EOF
```

---

## 📅 Day 5: Orchestrate Workflows with a Makefile

### Task Description
Create a `Makefile` that successfully orchestrates the environment setup, data processing, model training, testing, and workspace cleanup.

### Concept Summary
A **Makefile** is an automation script. Instead of developers manually typing out sequences of complex terminal commands every time they want to test or train a model, they define shortcuts (targets) in the Makefile.

### Step-by-Step Execution
**Step 1: Create the Makefile**
Create `/root/code/fraud-detection/Makefile` and define the automated targets. Every target command must be indented with a true `Tab` character.
```makefile
.PHONY: setup data train test clean all

all: setup data train test

setup:
	python3 -m venv mlops-venv
	./mlops-venv/bin/pip install -r requirements.txt

data:
	python src/data/process_data.py

train:
	python src/models/train.py

test:
	pytest tests/

clean:
	find . -type d -name __pycache__ -exec rm -rf {} +
	rm -rf .pytest_cache
	rm -rf models/*
```

**Step 2: Execute the Automation**
We can now run the entire MLOps workflow from start to finish with a single command.
```bash
make clean
make all
```

---

## 📅 Day 6: Set Up Code Quality Tools for ML Code

### Task Description
Enforce code quality by configuring `ruff` and `black` to automatically format code and catch linting errors on a 120-character line length limit.

### Concept Summary
**Code Quality Tools** maintain a clean, readable, and uniform codebase. 
- **Black** is a formatter; it restructures code layout automatically. 
- **Ruff** is a linter; it reads the code to find bad practices, unused imports, or errors and fixes them.

### Step-by-Step Execution
**Step 1: Configure Tool Rules**
Define the strict code quality rules in the project's central configuration file (`/root/code/fraud-detection/pyproject.toml`):
```toml
[tool.black]
line-length = 120

[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "W", "I"]
```

**Step 2: Auto-Lint with Ruff**
Scan the source code (`src/`) for structural errors and unorganized imports, and fix them automatically.
```bash
cd /root/code/fraud-detection
ruff check src/ --fix
```

**Step 3: Auto-Format with Black**
Re-format all the code in the `src/` directory to perfectly comply with the 120-character line limit and spacing rules.
```bash
black src/
```

---

## 📅 Day 7: Package an ML Project as Installable Python Package

### Task Description
Package the fraud detection model as an installable Python package under `/root/code/fraud-detection/` by configuring a PEP 517-compliant `pyproject.toml` file and building the distribution artifacts.

### Concept Summary
**Python Packaging** is the process of bundling code, metadata, and dependencies into a standard distributable format (e.g., a wheel file `.whl` or source archive `.tar.gz`).
- **`pyproject.toml`**: The modern PEP 517 configuration file used to define build dependencies, build backend, package metadata (name, version, license), and target execution dependencies.
- **Build Backend (`setuptools.build_meta`)**: Compiles code into a standard distributable package structure.
- **Distribution Formats**:
  - **Source Distribution (sdist)**: Contains the raw source code and setup files (`.tar.gz`).
  - **Built Distribution (wheel)**: A pre-compiled binary format that can be installed instantly by `pip` (`.whl`).

### Step-by-Step Execution

**Step 1: Create the Packaging Configuration**
Create or update the configuration file at `/root/code/fraud-detection/pyproject.toml` to specify the build requirements, project metadata, python constraints, and dependencies:
```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "fraud_detection"
version = "0.1.0"
requires-python = ">=3.10"
dependencies = [
    "scikit-learn",
    "pandas",
    "numpy"
]

[tool.setuptools.packages.find]
where = ["src"]
```

**Step 2: Build the Distribution Package**
Run python's standard build module from the root directory to generate the sdist and wheel:
```bash
cd /root/code/fraud-detection
python3 -m build
```

**Step 3: Verify the Build Artifacts**
Inspect the `dist/` directory to verify the creation of files:
```bash
ls -l dist/
```

**Step 4: Validate Package Installation**
Install the package locally to confirm it resolves and operates correctly:
```bash
pip install dist/fraud_detection-0.1.0-py3-none-any.whl
python -c "import fraud_detection; print(fraud_detection.__name__, 'installed successfully!')"
```

---

## 📅 Day 8: Configure Pre-Commit Hooks for ML Repository

### Task Description
Configure pre-commit hooks for the `fraud-detection` git repository under `/root/code/fraud-detection/` to automate code quality checks (formatting and linting) on every commit.

### Concept Summary
**Pre-Commit Hooks** are scripts that run automatically before each Git commit. They inspect the code being committed to ensure it complies with the codebase's styling and quality rules.
- **Git Hooks**: Native Git mechanism to run custom scripts on git events (such as `pre-commit`, `commit-msg`, `pre-push`).
- **Pre-Commit Framework**: A multi-language package manager for pre-commit hooks that makes it easy to declare, install, and update hooks via a simple configuration file.
- **Repository-Level Checks**: Ensures bad styling, missing formatting, or syntax warnings are blocked from being added to the Git history, keeping the repository clean and standardized.

### Step-by-Step Execution

**Step 1: Create the Pre-commit Configuration**
Create or update `.pre-commit-config.yaml` in the root of the repository (`/root/code/fraud-detection/`) with the following five hooks:
```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.11.13
    hooks:
      - id: ruff

  - repo: https://github.com/psf/black-pre-commit-mirror
    rev: 25.1.0
    hooks:
      - id: black
```

**Step 2: Register Hooks with Git**
Install the git hook scripts into your local git repository:
```bash
cd /root/code/fraud-detection
pre-commit install
```

**Step 3: Run Hooks Against All Tracked Files**
Execute the hooks manually across all tracked files to perform an initial code scan:
```bash
pre-commit run --all-files
```

---

## 📅 Day 9: Create a Custom ML Project Template with Cookiecutter

### Task Description
Fix a broken Cookiecutter template at `/root/code/mlops-template/` so it renders correctly, then use it to generate a new ML project at `/root/code/churn-model/` with `sklearn` as the ML framework.

### Concept Summary
**Cookiecutter** is a command-line tool that generates projects from predefined templates. Instead of manually creating folder structures, config files, and boilerplate code for every new ML project, teams maintain a single Cookiecutter template. Every new project is then stamped out from this template, guaranteeing consistency across the entire organization.

- **`cookiecutter.json`**: The configuration file at the template root that declares all user-facing variables and their default values. Cookiecutter reads this file to know what questions to ask (or what defaults to use with `--no-input`).
- **Template Directory (`{{cookiecutter.project_name}}/`)**: A folder named using Jinja2 syntax. When the project is generated, Cookiecutter replaces this with the actual project name provided by the user.
- **Jinja2 Templating**: Inside any file within the template directory, you can use `{{ cookiecutter.variable }}` for variable substitution and `{% if %}` / `{% elif %}` / `{% endif %}` for conditional logic.
- **Choice Variables**: When a variable in `cookiecutter.json` is defined as a JSON array (e.g., `["sklearn", "pytorch", "tensorflow"]`), Cookiecutter presents it as a selectable list. The first item is the default.

### Step-by-Step Execution

**Step 1: Inspect the Existing Broken Template**
First, examine what currently exists to identify all the issues:
```bash
# View the folder structure
find /root/code/mlops-template/ -type f | sort

# Check the current cookiecutter.json
cat /root/code/mlops-template/cookiecutter.json

# Check the template directory name
ls -la /root/code/mlops-template/
```

Common problems that prevent rendering:
- Malformed JSON in `cookiecutter.json` (missing keys, wrong types, trailing commas)
- Incorrectly named template directory (must be exactly `{{cookiecutter.project_name}}`)
- Jinja2 syntax errors in template files (missing `{{`, `}}`, `{%`, `%}`)
- Missing required files or directories

**Step 2: Fix `cookiecutter.json`**
The configuration file must declare exactly four variables with specific defaults. The `ml_framework` variable uses a JSON array for choice selection:
```bash
cat > /root/code/mlops-template/cookiecutter.json << 'EOF'
{
  "project_name": "my-ml-project",
  "author": "xFusionCorp",
  "python_version": "3.11",
  "ml_framework": ["sklearn", "pytorch", "tensorflow"]
}
EOF
```

> **Key Detail**: `ml_framework` is an array `["sklearn", "pytorch", "tensorflow"]`, not a plain string. This tells Cookiecutter to present choices. The first element (`sklearn`) becomes the default.

**Step 3: Fix the Template Directory Name**
The template directory must be named exactly `{{cookiecutter.project_name}}`. If it has any other name (e.g., `{{project_name}}`, `project/`, etc.), rename it:
```bash
cd /root/code/mlops-template/

# Rename the incorrectly named directory (adjust source name as needed)
mv "$(ls -d */ | head -1)" "{{cookiecutter.project_name}}" 2>/dev/null

# OR create it fresh if it doesn't exist
mkdir -p "{{cookiecutter.project_name}}"
```

**Step 4: Create Required Subdirectories**
The template must contain `data/`, `models/`, `src/`, and `tests/` directories. Since Cookiecutter skips empty directories during generation, we add `.gitkeep` placeholder files:
```bash
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/data"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/models"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/src"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/tests"

touch "/root/code/mlops-template/{{cookiecutter.project_name}}/data/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/models/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/src/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/tests/.gitkeep"
```

**Step 5: Fix `README.md` Template**
The README must reference both `project_name` and `author` using Jinja2 substitution:
```bash
cat > "/root/code/mlops-template/{{cookiecutter.project_name}}/README.md" << 'EOF'
# {{ cookiecutter.project_name }}

Author: {{ cookiecutter.author }}

## Overview

This is the {{ cookiecutter.project_name }} ML project created by {{ cookiecutter.author }}.

## Python Version

This project uses Python {{ cookiecutter.python_version }}.

## ML Framework

This project uses the {{ cookiecutter.ml_framework }} framework.

## Project Structure

```
{{ cookiecutter.project_name }}/
├── data/          # Dataset storage
├── models/        # Trained model artifacts
├── src/           # Source code
├── tests/         # Unit tests
├── README.md
└── requirements.txt
```
EOF
```

**Step 6: Fix `requirements.txt` Template**
The requirements file uses Jinja2 conditional logic to map framework choice names to their actual PyPI package names:
```bash
cat > "/root/code/mlops-template/{{cookiecutter.project_name}}/requirements.txt" << 'REQEOF'
{% if cookiecutter.ml_framework == "sklearn" -%}
scikit-learn
{% elif cookiecutter.ml_framework == "pytorch" -%}
torch
{% elif cookiecutter.ml_framework == "tensorflow" -%}
tensorflow
{% endif -%}
pandas
numpy
REQEOF
```

> **Critical Mapping**: The framework choice name and PyPI package name are different:
> - `sklearn` → installs `scikit-learn`
> - `pytorch` → installs `torch`
> - `tensorflow` → installs `tensorflow`
>
> The `-%}` syntax strips trailing whitespace to keep the output file clean.

**Step 7: Verify the Final Template Structure**
```bash
find /root/code/mlops-template/ -not -path '*/.git/*' | sort
```

Expected:
```
/root/code/mlops-template/
/root/code/mlops-template/cookiecutter.json
/root/code/mlops-template/{{cookiecutter.project_name}}/
/root/code/mlops-template/{{cookiecutter.project_name}}/README.md
/root/code/mlops-template/{{cookiecutter.project_name}}/data/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/models/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/requirements.txt
/root/code/mlops-template/{{cookiecutter.project_name}}/src/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/tests/.gitkeep
```

**Step 8: Generate the Project**
```bash
cookiecutter /root/code/mlops-template/ -o /root/code/ --no-input project_name=churn-model ml_framework=sklearn
```

**Step 9: Verify the Generated Project**
```bash
# Check structure
find /root/code/churn-model/ | sort

# Verify requirements.txt lists scikit-learn
cat /root/code/churn-model/requirements.txt

# Verify README.md mentions xFusionCorp (default author)
cat /root/code/churn-model/README.md

# Quick grep checks
grep "scikit-learn" /root/code/churn-model/requirements.txt
grep "xFusionCorp" /root/code/churn-model/README.md
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| `cookiecutter.json` | Declares template variables, defaults, and choices |
| Choice variables | JSON arrays: `["opt1", "opt2", "opt3"]` — first item is default |
| Template directory | Must be `{{cookiecutter.project_name}}` exactly |
| `{{ var }}` | Jinja2 substitution — replaced with actual value at render time |
| `{% if %} / {% endif %}` | Jinja2 conditional blocks for dynamic content |
| `-%}` | Strips trailing whitespace/newlines for clean output |
| `--no-input` | Non-interactive mode — uses defaults or CLI overrides |
| `-o` | Output directory for the generated project |
| `.gitkeep` | Placeholder to preserve empty directories in templates |

### Common Pitfalls
1. **Wrong PyPI names**: `sklearn` ≠ `scikit-learn`, `pytorch` ≠ `torch`
2. **Trailing commas in JSON**: `cookiecutter.json` must be valid JSON (no trailing commas)
3. **Template dir typos**: Even one wrong character in `{{cookiecutter.project_name}}` breaks rendering
4. **Empty directories skipped**: Cookiecutter won't copy empty folders — always add `.gitkeep`
5. **Jinja2 whitespace**: Use `-%}` to avoid unwanted blank lines in generated files

---

## 📅 Day 10: Install and Initialize DVC in an ML Project

### Task Description
The xFusionCorp Industries ML team is adopting DVC (Data Version Control) so that datasets and model files are versioned separately from code. Initialise DVC inside the existing Git repository at `/root/code/fraud-detection/` and record the initialisation in Git.

### Concept Summary
**DVC (Data Version Control)** is an open-source tool that works alongside Git to version large files — datasets, model weights, and other binary artifacts — that Git was never designed to handle efficiently. Instead of storing the actual data in the Git repository, DVC creates lightweight `.dvc` metafiles (pointers) that Git tracks, while the real data lives in configurable remote storage (S3, GCS, local, etc.).

- **`.dvc/` directory**: The internal control directory for DVC, similar to `.git/` for Git. Contains configuration (`config`), a local file cache (`cache/`), and temporary files (`tmp/`). The `.dvc/.gitignore` file automatically prevents the cache and temp files from being committed to Git.
- **`.dvcignore`**: Functions identically to `.gitignore` but for DVC operations. It tells DVC which files and directories to skip when scanning the workspace.
- **`dvc init`**: The initialization command that creates the `.dvc/` directory and `.dvcignore` file inside an existing Git repository. DVC requires Git — it cannot be initialized independently.
- **Separation of Concerns**: After initialization, Git continues to track code and DVC metafiles, while DVC manages the actual large data files. This keeps the Git repository small and fast while still maintaining full version history of datasets and models.

### Step-by-Step Execution

**Step 1: Navigate to the Project & Initialize DVC**
Navigate into the existing Git repository and run the DVC initialization command:
```bash
cd /root/code/fraud-detection/

dvc init
```

This creates the following files and directories:

| File/Directory | Purpose |
|----------------|---------|
| `.dvc/` | DVC control directory (internal config, cache, tmp) |
| `.dvc/.gitignore` | Prevents DVC internals (cache, tmp) from being committed to Git |
| `.dvc/config` | DVC configuration file (remote storage settings, etc.) |
| `.dvcignore` | Works like `.gitignore` but for DVC — tells DVC which files to ignore |

**Step 2: Stage and Commit DVC Files**
Stage all files produced by DVC initialization and record them in a Git commit:
```bash
git add .dvc/ .dvcignore

git commit -m "Initialize DVC"
```

> **Key Detail**: Only `.dvc/.gitignore` and `.dvc/config` get committed to Git. The `.dvc/.gitignore` ensures internal directories like `cache/` and `tmp/` are excluded automatically.

**Step 3: Verify the Setup**
```bash
git log --oneline -1
ls -la
```

Expected output from `git log`:
```
<hash> Initialize DVC
```

The `ls -la` output should show `.dvc/` and `.dvcignore` alongside the existing `.git/` directory.

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| DVC | Data Version Control — a Git-like tool for versioning data and models |
| `dvc init` | Initializes DVC in an existing Git repo, creating `.dvc/` and `.dvcignore` |
| `.dvc/` directory | Internal DVC control directory (config, cache, tmp) |
| `.dvc/config` | Stores DVC configuration such as remote storage settings |
| `.dvc/.gitignore` | Automatically excludes DVC internals (cache, tmp) from Git |
| `.dvcignore` | Tells DVC which files/directories to ignore (similar to `.gitignore`) |
| Code vs Data versioning | Git tracks code + DVC metafiles; DVC tracks large data/model files |

### Common Pitfalls
1. **Running `dvc init` outside a Git repo** — DVC requires an existing Git repository; it will error without one
2. **Forgetting to `git add .dvc`** — The `.dvc/` directory contents must be committed to Git for team collaboration
3. **Running `dvc init` twice** — Will error; use `dvc init --force` to re-initialize if needed
4. **Confusing `.dvcignore` with `.gitignore`** — `.dvcignore` is for DVC operations, `.gitignore` is for Git
5. **Committing DVC cache** — The `.dvc/.gitignore` prevents this, but removing that file would bloat the repo

---

## 📅 Day 11: Track a Dataset with DVC

### Task Description
A teammate added the `transactions.csv` dataset directly to Git. The goal is to migrate it to DVC control to align with team standards, ensuring that all datasets under `data/` are managed by DVC while keeping the local files intact.

### Concept Summary
Moving a dataset from Git to DVC involves two main phases: **de-tracking** from Git and **onboarding** to DVC. Git is excellent for source code but struggles with large binary files. DVC solves this by storing the data in a cache and providing Git with a tiny "receipt" or "pointer" (the `.dvc` file).

- **`git rm --cached`**: This command is surgical—it tells Git to stop watching the file without deleting it.
- **`.dvc` Pointer Files**: These are the bridge between Git and DVC. They contain the MD5 hash of the data, allowing Git to version the *metadata* while DVC versions the *actual data*.
- **Automatic Gitignore**: When you `dvc add` a file, DVC is smart enough to update (or create) a `.gitignore` file in that directory to prevent the data from ever leaking back into Git.

### Step-by-Step Execution

**Step 1: Remove from Git Index**
We must first strip Git of its control over the file. The `--cached` flag is critical here to ensure the data remains on our disk.
```bash
cd /root/code/fraud-detection
git rm --cached data/raw/transactions.csv
```

**Step 2: Initialize DVC Tracking**
By adding the file to DVC, we generate the tracking metadata.
```bash
dvc add data/raw/transactions.csv
```

**Step 3: Commit the Result**
We commit the `.dvc` "receipt" and the updated `.gitignore` to Git. This tells the rest of the team that this file is now managed by DVC.
```bash
git add data/raw/transactions.csv.dvc data/raw/.gitignore
git commit -m "Track transactions dataset with DVC"
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| Metadata vs. Data | Git stores the `.dvc` metadata; DVC stores the actual file content |
| `git rm --cached` | Decouples a file from Git without physical deletion |
| `.dvc` Pointer | A small, text-based manifest file containing hashes and file paths |
| DVC Workflow | Add data → Commit pointer to Git → Push data to DVC remote |
| Repo Hygiene | Keeps the Git history clean and clones lightning fast |

### Common Pitfalls
1. **Accidental Deletion**: Running `git rm` without `--cached` will remove your source data.
2. **Ignored .dvc files**: Never add `.dvc` files to `.gitignore`; they MUST be in Git.
3. **Dirty State**: Always ensure your Git workspace is clean before starting this migration to avoid confusion.
4. **Mismatched Remote**: Ensure DVC is configured to push to a remote storage if you intend for others to access the data.

---

## 📅 Day 12: Configure a DVC Remote Storage

### Task Description
The xFusionCorp Industries ML team uses SeaweedFS as the shared S3-compatible object store for DVC-tracked data. A `.dvc/config` already declares a remote called `s3` for the fraud-detection project, but `dvc push` currently fails. Correct the configuration and push the tracked data into the SeaweedFS bucket.

### Concept Summary
A **DVC Remote** is a storage location (S3, GCS, Azure, SSH, etc.) where DVC-tracked data is stored and shared among team members. When using **S3-Compatible Storage** (like SeaweedFS, MinIO, or Ceph), DVC must be configured with a custom `endpointurl` to point to the correct server instead of the default AWS S3 service.

### Step-by-Step Execution

**Step 1: Navigate to the Project & Verify Config**
```bash
cd /root/code/fraud-detection
cat .dvc/config
```

**Step 2: Correct the Remote URL**
Point the `s3` remote to the specific bucket name.
```bash
dvc remote modify s3 url s3://dvc-storage
```

**Step 3: Configure the Custom S3 Endpoint**
Tell DVC to use the SeaweedFS S3 endpoint instead of standard AWS.
```bash
dvc remote modify s3 endpointurl http://localhost:8333
```

**Step 4: Set the Default Remote**
Mark `s3` as the default destination for push/pull operations.
```bash
dvc remote default s3
```

**Step 5: Push Data to Remote**
Upload the tracked dataset to the SeaweedFS storage.
```bash
dvc push
```

**Step 6: Verify Success**
```bash
dvc status -c
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| `dvc remote modify` | Used to update specific parameters (url, endpoint, credentials) of an existing remote |
| `endpointurl` | Critical for S3-compatible storage to redirect requests from AWS to the local/private server |
| Default Remote | Simplifies workflow by allowing `dvc push/pull` without specifying the remote name |
| `dvc status -c` | Compares local data with the remote cache to ensure everything is synchronized |

### Common Pitfalls
1. **Missing Schema** — Forgetting `s3://` in the URL prevents DVC from knowing which driver to use.
2. **Incorrect Port** — Ensure the `endpointurl` includes the correct port (e.g., `8333` for SeaweedFS S3).
3. **No Default Set** — Running `dvc push` without a default remote or a specified `-r` flag will result in an error.

---

## 📅 Day 13: Pull DVC-Tracked Data from Remote

### Task Description
A new xFusionCorp Industries team member has cloned the `fraud-detection` repository onto a fresh machine. The DVC remote is already configured to point at the team's SeaweedFS bucket, but `dvc pull` is failing. Diagnose the cause, correct the configuration, and pull the dataset.

### Concept Summary
**Pulling Data** in DVC is the process of downloading the actual file content from a remote storage to the local cache and linking it to the workspace. On a new machine, DVC pointers (`.dvc` files) exist, but the data does not. 

When working with private remotes, credentials should be stored in the **Local Configuration** (`.dvc/config.local`). This file is excluded from Git to prevent sensitive keys from leaking, making it the correct place for individual developer access keys.

- **Local Config**: Stores credentials and environment-specific settings.
- **`use_ssl false`**: Required when the remote server (like a local SeaweedFS) uses pure HTTP.
- **`dvc pull`**: Merges `dvc fetch` (downloading data to cache) and `dvc checkout` (linking cache to workspace).

### Step-by-Step Execution

**Step 1: Navigate to the Project**
```bash
cd /root/code/fraud-detection
```

**Step 2: Configure Local Credentials**
Use the `--local` flag to ensure these settings are saved to `.dvc/config.local` and not shared via Git.
```bash
# Set access key
dvc remote modify --local s3 access_key_id weedadmin

# Set secret key
dvc remote modify --local s3 secret_access_key weedadmin123

# Set custom endpoint URL and disable SSL
dvc remote modify --local s3 endpointurl http://localhost:8333
dvc remote modify --local s3 use_ssl false
```

**Step 3: Verify the Local Config**
```bash
cat .dvc/config.local
```
Expected output:
```ini
['remote "s3"']
    access_key_id = weedadmin
    secret_access_key = weedadmin123
    endpointurl = http://localhost:8333
    use_ssl = false
```

**Step 4: Pull the Data**
Download the dataset from SeaweedFS to the local machine.
```bash
dvc pull -v
```

**Step 5: Verify the Dataset**
Check that the file is present and readable.
```bash
ls -l data/raw/transactions.csv
head data/raw/transactions.csv
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| `dvc pull` | Synchronizes the local workspace with the remote storage |
| `--local` | Saves configuration changes to `.dvc/config.local` (ignored by Git) |
| Credentials | Access keys and secrets should always be set locally, never in the shared config |
| SSL Configuration | `use_ssl false` is necessary for local development environments on HTTP |

### Common Pitfalls
1. **Missing `--local` flag**: Accidentally committing credentials to the shared `.dvc/config` is a major security risk.
2. **Incorrect Endpoint**: Forgetting the `http://` prefix in `endpointurl` will cause connection errors.
3. **Cache Sync**: If `dvc pull` fails, ensure the remote bucket actually contains the objects listed in the `.dvc` files.

---

## 📅 Day 14: Reproducible ML Pipelines with DVC

### Task Description
Correct the `dvc.yaml` pipeline in the `fraud-detection` project to ensure data processing and splitting run end-to-end. The pipeline must consist of two stages: `process_data` (cleans raw transactions) and `split_data` (divides cleaned data into train/test sets).

### Concept Summary
**DVC Pipelines** allow you to define ML workflows as a sequence of modular steps. By declaring **dependencies** (`deps`) and **outputs** (`outs`) for each stage in a `dvc.yaml` file, DVC can automatically track which parts of the pipeline need to be re-run when data or code changes. This creates a **Directed Acyclic Graph (DAG)** that guarantees reproducibility.

### Step-by-Step Execution

**Step 1: Configure the Pipeline Stages**
Update `/root/code/fraud-detection/dvc.yaml` to define the data processing and splitting steps:
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

**Step 2: Reproduce the Pipeline**
Run the complete workflow using the `dvc repro` command. DVC will execute the stages in the correct order based on their dependencies.
```bash
cd /root/code/fraud-detection
dvc repro
```

**Step 3: Verify Pipeline Integrity**
Check the status of the pipeline to ensure all outputs are up to date and consistent with the current dependencies.
```bash
dvc status
```
*Expected Output:* `Data and pipelines are up to date.`

**Step 4: Visualize Dependencies**
Display the pipeline's DAG to verify the logical flow between stages.
```bash
dvc dag
```
```text
+--------------+          +------------+
| process_data | -------->| split_data |
+--------------+          +------------+
```

---

## 📅 Day 15: Parameter Management and Reproducibility with DVC

### Task Description
The xFusionCorp Industries ML team manages model hyperparameters through `params.yaml` so experiments can vary without code changes. The goal is to correct the parameter wiring in `dvc.yaml` and demonstrate that DVC re-runs the train stage when a parameter changes.

### Concept Summary
**Parameter Management** in DVC decouples hyperparameters from source code. By tracking `params.yaml`, DVC can detect changes in configuration without requiring changes to the execution scripts. This ensures that every experiment is reproducible and the exact parameters used are recorded in `dvc.lock`.

### Step-by-Step Execution

**Step 1: Define Parameters**
Ensure `params.yaml` contains the correct keys.
```yaml
train:
  n_estimators: 100
```

**Step 2: Wire Parameters in `dvc.yaml`**
Update the `train` stage to reference the parameters correctly.
```yaml
  train:
    cmd: python src/models/train.py
    deps:
      - data/processed/train.csv
      - src/models/train.py
    params:
      - train.n_estimators
    outs:
      - models/model.pkl
```

**Step 3: Execute and Verify**
Run the pipeline to record the initial state.
```bash
cd /root/code/fraud-detection
dvc repro
```

**Step 4: Change Parameters and Re-run**
Modify `n_estimators` to `200` in `params.yaml` and run `dvc repro`. DVC skips the data processing stages and only re-executes the training stage.
```bash
# Update params.yaml to n_estimators: 200
dvc repro
```

*Expected Output:*
```text
Stage 'process_data' didn't change, skipping
Stage 'split_data' didn't change, skipping
Running stage 'train':
> python src/models/train.py
Updating lock file 'dvc.lock'
```

**Step 5: Verify results in `dvc.lock`**
The `dvc.lock` file now shows the updated parameter value for the train stage.

---

## 📅 Day 16: Track ML Metrics with DVC

### Task Description
Integrate metric tracking into the `fraud-detection` pipeline to automatically capture model performance (accuracy, precision, recall) during the evaluation stage. Use DVC to display and compare these metrics.

### Concept Summary
**ML Metrics** are numerical values that quantify how well a model is performing. Traditionally, these are buried in logs or manually entered into spreadsheets. DVC makes metrics **first-class objects** by:
- **Tracking**: Automatically recording performance files (JSON/YAML) along with the code and data that produced them.
- **Reporting**: Aggregating metrics across the project into a clean table with `dvc metrics show`.
- **Comparing**: Showing differences in performance between experiments or Git branches.

### Step-by-Step Execution

**Step 1: Create an Evaluation Script**
Develop a script `src/models/evaluate.py` that loads the model, runs predictions on the test set, and writes the results to a structured `metrics.json` file.

```python
import json
import pickle
import pandas as pd
from sklearn.metrics import accuracy_score

# Sample Evaluation Logic
model = pickle.load(open("models/model.pkl", "rb"))
test_data = pd.read_csv("data/processed/test.csv")
# ... calculation ...
metrics = {"accuracy": 0.942, "precision": 0.915}
with open("metrics.json", "w") as f:
    json.dump(metrics, f)
```

**Step 2: Add the Evaluate Stage to `dvc.yaml`**
Register the new stage and explicitly mark `metrics.json` as a metric file.

```yaml
stages:
  evaluate:
    cmd: python src/models/evaluate.py
    deps:
      - data/processed/test.csv
      - models/model.pkl
      - src/models/evaluate.py
    metrics:
      - metrics.json:
          cache: false
```

**Step 3: Run the Pipeline**
Execute the pipeline to trigger the evaluation and generate the metrics file.
```bash
dvc repro
```

**Step 4: View the Performance results**
Use the built-in DVC command to display the scalars captured in the file.
```bash
dvc metrics show
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **`metrics` field** | Identifies output files that contain scalar metrics for DVC to parse. |
| **`dvc metrics show`** | Command to visualize and compare metrics in the terminal. |
| **`cache: false`** | Prevents metrics from being stored in the large-file cache, keeping them lightweight. |
| **Reproducibility** | Since metrics are tied to the pipeline, they are updated only when dependencies or code change. |

### Common Pitfalls
1. **Wrong File Format**: DVC prefers JSON or YAML for metrics. Using CSV for scalars may require extra configuration.
2. **Missing Dependencies**: If `evaluate.py` isn't listed as a dependency, DVC won't know to re-run evaluation if the scoring logic changes.
3. **Not using `dvc repro`**: Manually running the script won't update DVC's understanding of the metrics state.

---

## 📅 Day 17: Run and Compare DVC Experiments

### Task Description
Run multiple training experiments varying the `n_estimators` hyperparameter, compare their performance results (F1-score), and apply the best configuration to the project workspace.

### Concept Summary
**DVC Experiments** provide a way to iterate on models without creating extra Git branches. They allow you to test hundreds of hyperparameter combinations while keeping the project history clean. 
- **Shadow commits**: Experiments are stored in special Git refs (`refs/exps`) that don't appear in your regular `git log`.
- **Param Overrides**: You can change values on the fly without editing the `params.yaml` file manually.
- **Applying Results**: Once you find a "winner," you can swap the current workspace state with that experiment's state.

### Step-by-Step Execution

**Step 1: Execute Experiments with Parameter Overrides**
Run three separate experiments by setting the `n_estimators` to 50, 200, and 500 using the CLI.
```bash
# cd /root/code/fraud-detection
dvc exp run --set-param train.n_estimators=50
dvc exp run --set-param train.n_estimators=200
dvc exp run --set-param train.n_estimators=500
```

**Step 2: Compare Experiment Performance**
Display the experiment leaderboard to compare the `f1_score` and other metrics across all runs.
```bash
dvc exp show
```

**Step 3: Identify and Select the Best Run**
Look for the run with the highest `f1_score`. In our scenario, let's assume the run with `n_estimators=200` performed best.

**Step 4: Promote the Winning Experiment to Workspace**
Apply the chosen experiment's state to the workspace. This overwrites the local `params.yaml`, `metrics.json`, and `model.pkl`.
```bash
# Substitute <exp_id> with the actual ID from dvc exp show (e.g., exp-c3d4)
dvc exp apply <exp_id>
```

**Step 5: Verify and Commit**
Verify that the workspace now reflects the optimal parameters and commit the change to Git.
```bash
cat params.yaml  # Should show n_estimators: 200
git add .
git commit -m "Promote best experiment (n_estimators=200) to main track"
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **`dvc exp run`** | Runs the pipeline and captures the state as an experiment. |
| **`--set-param`** | Overrides `params.yaml` keys specifically for the duration of that experiment run. |
| **`dvc exp show`** | A powerful summary table for comparing metrics and parameters. |
| **`dvc exp apply`** | Updates the workspace with the data and config from a specific experiment. |

### Common Pitfalls
1. **Uncommitted Changes**: DVC requires a clean Git state (or at least a baseline commit) to anchor experiments.
2. **Naming Experiments**: If you don't name them, DVC gives them random names like `topaz-pug`. Use `--name` if you want identifiable runs.
3. **Conflicting parameters**: If you override a parameter via CLI, it takes precedence over the value in `params.yaml`.

---

## 📅 Day 18: Dataset Versioning with Git and DVC

### Task Description
The xFusionCorp Industries ML team keeps different dataset and model versions on different Git branches so that the team can roll between versions cleanly. Tag the current state as v1.0, produce a v2-improved branch based on a newer dataset, and confirm that switching back restores the original data.

### Concept Summary
**Dataset Versioning** with DVC and Git combines Git's branching and tagging capabilities with DVC's efficient large-file handling. 
- **Git Tags**: Used to mark specific milestones (like `v1.0`) in the project history, including the exact `.dvc` file versions at that time.
- **Git Branches**: Allow parallel development (e.g., experimental data or model architectures) without affecting the stable `main` branch.
- **`dvc checkout`**: The command that "materializes" the data files corresponding to the current Git branch or tag by linking them from the DVC cache.

### Step-by-Step Execution

**Step 1: Tag the Current Version (v1.0)**
Mark the current stable state of the project.
```bash
cd /root/code/fraud-detection
git tag -a v1.0 -m "Baseline dataset and model"
```

**Step 2: Create a New Branch for Improvements**
Branch off to work on the updated dataset.
```bash
git checkout -b v2-improved
```

**Step 3: Update the Dataset**
Replace the tracked dataset with the new version and update DVC metadata.
```bash
# Replace old data with new data
cp data/raw/transactions_v2.csv data/raw/transactions.csv

# Update tracking
dvc add data/raw/transactions.csv
```

**Step 4: Reproduce the Pipeline and Commit**
Update the model and lockfile for the new branch.
```bash
dvc repro
git add data/raw/transactions.csv.dvc dvc.lock
git commit -m "Incorporate v2 dataset and retrain model"
```

**Step 5: Restore to Previous Version**
Switch back to `main` and restore the original `v1` dataset.
```bash
git checkout main
dvc checkout
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **`git tag`** | Creates a named reference to a specific commit. |
| **Branching** | Isolates different data/model evolutions. |
| **`dvc checkout`** | Syncs data on disk with the metadata on the current branch/tag. |
| **MD5 Hashing** | DVC uses content hashes to avoid duplicating identical files in the cache. |

### Common Pitfalls
1. **Forget `dvc checkout`**: Switching Git branches only changes the `.dvc` files; the actual massive data stays the same on disk until `dvc checkout` is run.
2. **Tagging Uncommitted Changes**: Ensure everything (especially `dvc.lock` and `.dvc` files) is committed before tagging.
3. **Cache Size**: Keeping many versions of huge datasets can consume significant disk space in `.dvc/cache`.
---

## 📅 Day 19: Build Complete DVC ML Pipeline with Remote Storage and Experiments

### Task Description
Complete the xFusionCorp Industries fraud-detection production DVC pipeline. Fix an incorrect output path in the existing stages, add `train` and `evaluate` stages, run the full pipeline, push artifacts to SeaweedFS remote storage, and tag the release as `v1.0`.

### Concept Summary
A **Full DVC Pipeline** orchestrates the entire ML lifecycle from data ingestion to model evaluation. 
- **SeaweedFS / S3 Remotes**: Provide a centralized, scalable storage for datasets and models, ensuring that anyone on the team can reproduce the results by pulling the exact data versions.
- **Reproducibility**: By defining clear `deps` and `outs` for each stage, DVC ensures that only necessary steps are re-run when changes occur.
- **Release Management**: Tagging a specific Git commit as `v1.0` (along with its `dvc.lock` and `.dvc` files) creates a permanent record of a production-ready model and the exact data used to train it.

### Step-by-Step Execution

**Step 1: Fix Existing Configuration**
Identify and correct any path errors in `dvc.yaml`. In this case, ensuring that early stages like `ingest` or `preprocess` correctly define their output paths so downstream stages can find them.

**Step 2: Prepare Pipeline Scripts**
Copy the necessary Python scripts for training and evaluation into the `scripts/` directory.
```bash
cp scripts-staging/train.py scripts/train.py
cp scripts-staging/evaluate.py scripts/evaluate.py
```

**Step 3: Define the Train Stage**
Add the training logic to `dvc.yaml`, linking it to hyperparameter tracking.
```yaml
  train:
    cmd: python scripts/train.py
    deps:
      - data/processed/preprocessed.csv
      - scripts/train.py
    params:
      - n_estimators
      - max_depth
      - test_size
      - random_seed
    outs:
      - models/model.pkl
      - data/processed/test_split.csv
    metrics:
      - metrics.json:
          cache: false
```

**Step 4: Define the Evaluate Stage**
Add the evaluation step to capture model performance.
```yaml
  evaluate:
    cmd: python scripts/evaluate.py
    deps:
      - models/model.pkl
      - data/processed/test_split.csv
      - scripts/evaluate.py
    outs:
      - reports/evaluation.json:
          cache: false
```

**Step 5: Run the Pipeline and Push to Remote**
Execute the workflow and sync the results to the remote SeaweedFS storage.
```bash
dvc repro
dvc push
```

**Step 6: Tag the Release**
State the version clearly in Git.
```bash
git add .
git commit -m "Complete pipeline and release v1.0"
git tag -a v1.0 -m "Release v1.0: Complete fraud-detection pipeline"
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **Production DAG** | A fully interconnected graph of stages from raw data to final report. |
| **Remote Cache** | Shared storage (e.g., SeaweedFS) that prevents re-running expensive stages on different machines. |
| **Pipeline Metrics** | Integrating performance data (`metrics.json`, `evaluation.json`) directly into the pipeline metadata. |
| **Versioning** | Using Git tags to lock the entire state (code + data + model) for production stability. |

### Common Pitfalls
1. **Dangling Dependencies**: If a stage's `deps` are missing or misnamed, `dvc repro` will fail to link the stages.
2. **Cache Policy**: Forgetting `cache: false` on small metrics files can clutter the remote storage with many tiny versions of text files.
3. **Environment Mismatch**: Ensure that libraries used in scripts are consistent across the development and remote environments.

---

## 📅 Day 20: Setting Up and Launching MLflow Tracking Server

### Task Description
Configure and launch a persistent MLflow Tracking Server using a SQLite backend and a local artifact store. Ensure the server is accessible through a proxy by configuring CORS and host settings.

### Concept Summary
**MLflow Tracking** is an API and UI for logging parameters, code versions, metrics, and output files. 
- **Backend Store**: Where MLflow stores metadata (experiment names, runs, parameters, metrics). SQLite is a common choice for local or small-team setups.
- **Artifact Store**: Where MLflow stores large files like models, plots, and data samples. This can be a local directory or a cloud bucket (S3, GCS).
- **Tracking Server**: A centralized service that allows multiple users/scripts to log data to the same backend and artifact stores.

### Step-by-Step Execution

**Step 1: Create Storage Directories**
Initialize the directories where the tracking data will reside.
```bash
mkdir -p /root/code/mlflow-backend
mkdir -p /root/code/mlflow-artifacts
```

**Step 2: Start the MLflow Tracking Server**
Launch the server in the background using `nohup`. We use `--host 0.0.0.0` to listen on all interfaces and configure CORS to allow proxy access.
```bash
nohup mlflow server \
  --host 0.0.0.0 \
  --port 5000 \
  --backend-store-uri sqlite:////root/code/mlflow-backend/mlflow.db \
  --artifacts-destination /root/code/mlflow-artifacts \
  --cors-allowed-origins '*' \
  --allowed-hosts '*' \
  > /root/mlflow-server.log 2>&1 &
```

**Step 3: Verify the Deployment**
Check that the server process is alive and the database file has been successfully created.
```bash
ps -ef | grep mlflow
ls -l /root/code/mlflow-backend/mlflow.db
ss -tulpn | grep 5000
```

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **`--backend-store-uri`** | Defines the database connection string. `sqlite:///` indicates a local file database. |
| **`--artifacts-destination`** | Specifies the base location for storing run artifacts. |
| **`nohup ... &`** | Commands that ensure the server runs independently of the terminal session. |
| **CORS/Allowed Hosts** | Necessary for the MLflow UI to be served correctly through web proxies. |

### Common Pitfalls
1. **Directory Permissions**: Ensure the user running the server has write access to the backend and artifact directories.
2. **Port Conflicts**: If port 5000 is already in use by another service (like a default Flask app), the server will fail to start.
3. **Database Locks**: SQLite can sometimes experience locking issues if accessed simultaneously by multiple processes in a specific way, though rare for basic tracking.


---

## 📅 Day 21: Logging Your First MLflow Experiment

### Task Description
Configure a Python script to log a baseline experiment to the MLflow tracking server, including hyperparameters, evaluation metrics, and the model artifact itself.

### Concept Summary
**Experiment Tracking** is the heart of MLflow. It allows data scientists to record all relevant information about a model training run so it can be reproduced, compared, and audited later.
- **Parameters**: Key-value inputs to the model (e.g., `n_estimators`, `max_depth`). Usually fixed before the run.
- **Metrics**: Quantitative results of the run (e.g., `accuracy`, `f1_score`). Can be updated throughout the run.
- **Artifacts**: Output files generated by the run, such as the trained model file, plots, or datasets.
- **mlflow.start_run()**: A context manager that defines the scope of a single experiment run. Everything logged inside this block is associated with that specific run ID.

### Step-by-Step Execution
**Step 1: Create the Experiment Script**
We write a script that connects to the MLflow server and logs our data using the `mlflow` library.
```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

with mlflow.start_run():
    # Log hyperparameters
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 5)
    
    # Log performance metrics
    mlflow.log_metric("accuracy", 0.85)
    mlflow.log_metric("f1_score", 0.82)
    
    # Log the model artifact
    iris = load_iris()
    model = RandomForestClassifier(n_estimators=100, max_depth=5).fit(iris.data, iris.target)
    mlflow.sklearn.log_model(model, "model")
```

**Step 2: Execute and Log**
Run the script to push the data to the tracking server.
```bash
python3 /root/code/log_experiment.py
```

**Step 3: Verify in the UI**
Navigate to the MLflow Tracking UI. Under the "Default" experiment, a new entry should appear. Clicking it reveals the logged parameters, metrics, and a "model" folder in the artifacts section containing the serialized model.

---

## 📅 Day 22: Creating Metadata-Rich Experiments in MLflow (UI)

### Task Description
Organize MLflow tracking by creating project-specific experiments with descriptions and team tags directly through the MLflow User Interface.

### Concept Summary
The **MLflow Tracking UI** provides an intuitive way to manage experiments without writing code. 
- **Centralized View**: See all experiments and their metadata in one place.
- **Interactive Metadata**: Add tags and notes to experiments to document project context.
- **Tagging for Filtering**: Adding custom tags like `team` allow for better organization as the number of experiments grows.

### Step-by-Step Execution
**Step 1: Create the Experiment via the UI**
Navigate to the MLflow UI and click the **+ Create Experiment** icon in the sidebar. This opens a creation form.

**Step 2: Enter Experiment Details and Tags**
In the creation form:
- Enter a name (e.g., `fraud-detection` or `churn-prediction`).
- Click **Add Tag** to include metadata. For `fraud-detection`, we added a `team` tag with the value `ml-platform`. For `churn-prediction`, we used `analytics`.

**Step 3: Add Descriptions and Verify**
After creation, click on the experiment to open its details page. You can add a text description or markdown notes in the "Notes" section. We added "Production fraud detection models" to provide clear context for stakeholders.

---

## 📅 Day 23: Search and Query MLflow Runs

### Task Description
Use the MLflow Python API to programmatically search through experiment runs and apply tags based on performance metrics (f1_score).

### Concept Summary
**MLflow Search & Querying** allows for automated management of experimental results.
- **MlflowClient**: A low-level API to interact with the tracking server, offering more control than the high-level `mlflow.*` functions.
- **search_runs()**: Returns a list of run objects that can be filtered or iterated upon.
- **set_tag()**: Programmatically updates the metadata of an existing run without needing to re-run the code.
- **Review Workflow**: Automating the "Shortlisting" or "Rejection" of models based on threshold metrics is a key step in MLOps pipelines.

### Step-by-Step Execution

**Step 1: Connect to Server and Search Runs**
Use the `MlflowClient` to fetch runs from the `fraud-detection` experiment.
```python
mlflow.set_tracking_uri("http://localhost:5000")
client = MlflowClient()
exp = client.get_experiment_by_name("fraud-detection")
runs = client.search_runs([exp.experiment_id])
```

**Step 2: Apply Conditional Tagging**
Iterate through the runs and check the metrics stored in `r.data.metrics`. Apply the `review-status` tag using `client.set_tag`.
```python
for r in runs:
    f1 = r.data.metrics.get("f1_score")
    if f1 == 0.95:
        client.set_tag(r.info.run_id, "review-status", "shortlisted")
    elif f1 is not None and f1 < 0.75:
        client.set_tag(r.info.run_id, "review-status", "rejected")
```

**Step 3: Verification**
Query the experiment again and print the metrics and tags to confirm the updates. High-performing models are now marked as `shortlisted`, and low-performing ones as `rejected`, ready for the next stage of the pipeline.

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **`client.search_runs`** | Essential for batch processing and automated evaluation of experiments. |
| **`client.set_tag`** | Allows adding new metadata layers (like human reviews or downstream status) post-training. |
| **Automated Flagging** | Reduces manual overhead in comparing dozens or hundreds of runs in the UI. |

### Common Pitfalls
1. **Metric Keys**: Ensure the metric key (e.g., `f1_score`) matches exactly what was logged; it's case-sensitive.
2. **Missing Metrics**: Always check if a metric exists (is not None) before performing comparisons to avoid errors.
3. **Tracking URI**: Forgetting to set the tracking URI will lead to the client attempting to use the local `./mlruns` directory instead of the server.

---

## 📅 Day 24: Enable MLflow Autologging

### Task Description
Implement MLflow autologging to automatically capture training metadata without manual logging statements.

### Concept Summary
**MLflow Autologging** is a powerful feature that "hooks" into supported libraries (like sklearn, tensorflow, pytorch) to log parameters, metrics, and models automatically.
- **`mlflow.[flavor].autolog()`**: Enables automatic logging for a specific library. It should be called *before* the training code starts.
- **Constructor Capture**: Unlike manual logging, autologging captures the full set of hyperparameters, including default values that weren't explicitly passed to the constructor.
- **Fit Instrumenting**: The logging happens behind the scenes during the `.fit()` (or equivalent) call.
- **Workflow Efficiency**: Eliminates boilerplate code and ensures consistent logging across different projects.

### Step-by-Step Execution

**Step 1: Set Experiment and Enable Autolog**
It's critical to enable autologging before initializing the model.
```python
import mlflow.sklearn
mlflow.sklearn.autolog()
mlflow.set_experiment("autolog-demo")
```

**Step 2: Train the Model**
Simply run your standard scikit-learn training code. Autologging will detect the `.fit()` call.
```python
model = LogisticRegression(C=1.0)
model.fit(X, y)
```

**Step 3: Verification in UI**
Check the MLflow UI. The run will contain a comprehensive list of parameters, training metrics (like training loss if available), and the full model artifact package.

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **Flavor-Specific** | Each library (sklearn, xgboost, etc.) has its own autolog function. |
| **Full Parameter Set** | Ensures you know exactly what defaults the model used, improving reproducibility. |
| **Artifact Packaging** | Automatically creates the `model`, `conda.yaml`, and `requirements.txt` files. |

### Common Pitfalls
1. **Timing**: Calling `autolog()` *after* `.fit()` will result in no data being logged for that run.
2. **Library Versions**: Autologging support varies by version; ensure your library version is compatible with your MLflow version.
3. **Double Logging**: If you use both autologging and `mlflow.start_run()` with manual `log_param`, you might end up with duplicate entries or minor conflicts if the keys are the same.

---

# Day 25: MLflow Model Registry

The MLflow Model Registry is a centralized model store, set of APIs, and UI, to collaboratively manage the full lifecycle of an MLflow Model. It provides model lineage (which MLflow run produced the model), model versioning, stage transitions (e.g. from staging to production or archiving), and annotations.

### Step-by-Step Execution

**Step 1: Registering Versions**
Registering a model adds it to the Registry. If the model name doesn't exist, it creates it (v1). Subsequent registrations for the same name increment the version (v2, v3, etc.).
- **Baseline Run**: Registered as v1 of `fraud-detector`.
- **Improved Run**: Registered as v2 of `fraud-detector`.

**Step 2: Model Metadata**
Descriptions can be added at the Model level (describing what the model does) or the Version level (describing what's specific about that iteration).
- **Model Description**: "Fraud detection model for xFusionCorp transactions"

**Step 3: Using Aliases**
Aliases allow you to label specific versions for deployment or comparison without changing client code. For example, your app could always query the model with the `@champion` alias.
- **v1 Alias**: `challenger`
- **v2 Alias**: `champion`

### Key Concepts & Takeaways

| Concept | Detail |
|---------|--------|
| **Model Registry** | A central repository for managing models as identifiable entities. |
| **Model Versioning** | Automatic tracking of model iterations produced from different runs. |
| **Aliases** | Mutatable labels that point to specific versions (e.g., `champion`, `production`). |
| **Model Lineage** | The link between a registered model version and its original experiment run. |

### Common Pitfalls
1. **Naming Conflicts**: Model names must be unique within the registry.
2. **Missing Artifacts**: You cannot register a run that did not successfully log a model artifact.
3. **Alias Confusion**: Ensure aliases are updated consistently; only one version can hold a specific alias at a time for a given model.

---

## 📅 Day 26: Compare Model Runs and Select the Best

### Task Description
Identify the best performing model from a set of experimental runs in MLflow and tag it as a production candidate to facilitate downstream automated deployment processes.

### Concept Summary
**Model Comparison** is the phase where different experiments (algorithms, hyperparameters, or data versions) are evaluated side-by-side. In a production MLOps workflow, identifying the "champion" model based on specific metrics (like F1-score) is critical. 

**Run Tags** in MLflow provide a flexible way to add metadata to individual runs. Unlike metrics (which are numeric) or parameters (which are fixed at start), tags can be updated manually or by automated scripts to signal the status of a run (e.g., `production-candidate: true`).

### Step-by-Step Execution

**Step 1: Metric Evaluation**
Review the logged metrics for all runs in the `model-comparison` experiment.
*   **GradientBoosting**: `f1_score` = 0.91 (Winner)
*   **RandomForest**: `f1_score` = 0.85
*   **LogisticRegression**: `f1_score` = 0.78

**Step 2: Accessing the Winning Run**
Select the run with the highest value for the target metric (F1-score in this case). In our experiment, this is the **GradientBoosting** run.

**Step 3: Applying Production-Candidate Tag**
Navigate to the run detail page and add the metadata tag:
*   **Key**: `production-candidate`
*   **Value**: `true`

**Step 4: Cleanup and Verification**
Ensure no other competing runs in the same experiment carry the same tag, preventing ambiguity for deployment scripts.

---

## 📅 Day 27: Load Model from Registry with Custom Preprocessing

### Task Description
Load a registered model with a specific alias from the MLflow Model Registry and use it in a script that includes custom preprocessing logic for batch predictions.

### Concept Summary
**Model Portability and Registry Access** allows teams to decouple model training from model consumption. By using stable URI aliases like `@champion`, the deployment environment doesn't need to know the specific version number.

**Custom Preprocessing Wrappers** are often necessary when a model requires specific data transformations (like scaling or encoding) that aren't built directly into the raw model file. MLflow's `pyfunc` flavor is the primary way to package these as logic-rich models.

### Step-by-Step Execution

**Step 1: Connecting to the Registry**
Identify the correct Model URI using the format `models:/<model_name>@<alias>`. This ensures the script always pulls the current designated champion.

**Step 2: Loading via PyFunc**
The `mlflow.pyfunc.load_model` function is used to load the model. This method is preferred for custom wrappers as it correctly instantiates the Python class containing the preprocessing logic.

**Step 3: Building the Prediction Pipeline**
The loaded model's `predict` method is called on the input data (usually a Pandas DataFrame). The results are then joined back to the original data for complete context.

**Step 4: Output Generation**
Saving the results to a CSV or a database for downstream systems to consume.

### Key Takeaways

| Tool/Feature | Utility |
| :--- | :--- |
| **mlflow.pyfunc** | The universal interface for loading any MLflow model as a Python function. |
| **@champion Alias** | A mutable pointer to a specific model version, ideal for production automation. |
| **Batch Inference** | Efficiently processing data in bulk for offline analysis. |


---

## 📅 Day 28: Fix a Broken MLflow Project and Re-Run It

### Task Description
Identify and fix a command-line interface mismatch in a pre-staged MLflow Project. The goal is to repair the `MLproject` file so it correctly passes parameters to `train.py`, then execute the project twice to demonstrate full reproducibility.

### Concept Summary
**MLflow Projects** are a self-contained format for packaging data science code. An `MLproject` file acts as the manifest, defining how to run the code, what parameters it accepts, and what environment it needs. Reproducibility hinges on the **Command-Line Interface (CLI)** mapping—if the `MLproject` command string doesn't perfectly match the script's `argparse` requirements, the orchestration fails.

### Step-by-Step Execution

**Step 1: Diagnose the Failure**
Run the project manually to see the error. The mismatch typically occurs because of incorrect flag names (e.g., using `--n-estimators` when the script expects `--n_estimators`).
```bash
cd /root/code/trainer
mlflow run . -e train --env-manager=local
```

**Step 2: Correct the MLproject File**
Update the `command` line in `MLproject` to ensure parameter placeholders and flag names match the `train.py` argument declarations exactly.
```yaml
# /root/code/trainer/MLproject (Corrected)
name: trainer

entry_points:
  train:
    parameters:
      n_estimators: {type: int, default: 100}
      max_depth: {type: int, default: 5}
    command: "python train.py --n_estimators {n_estimators} --max_depth {max_depth}"
```

**Step 3: Execute the Fixed Project**
Run the project twice to verify parameter propagation and default behavior.
```bash
# 1. Explicitly override parameters
mlflow run . -e train -P n_estimators=200 -P max_depth=10 --env-manager=local

# 2. Use default values defined in MLproject
mlflow run . -e train --env-manager=local
```

**Step 4: Verify in MLflow UI**
Check the tracking server. The `trainer` experiment should now contain one original failed run and two successful new runs with different parameter values.






## Day 29: Configure MLflow with Remote Tracking Server and Artifact Store

**Goal:** Fix artifact upload failures by explicitly configuring the SeaweedFS S3 endpoint in the MLflow tracking server.

**Step 1: Configure Environment Variables**
Add the SeaweedFS endpoint to /root/code/start-mlflow.sh.
`ash
export MLFLOW_S3_ENDPOINT_URL=http://localhost:8333
export AWS_ACCESS_KEY_ID=weedadmin
export AWS_SECRET_ACCESS_KEY=weedadmin123
`

**Step 2: Restart and Test**
`ash
bash /root/code/restart-mlflow.sh
python3 /root/code/log_test_run.py
`

**Step 3: Verification**
Check SeaweedFS Filer or use curl to ensure artifacts are stored in s3://mlflow-artifacts.

---

## 📅 Day 30: Create a Health Monitor Script for an ML Application

### Task Description
Develop a custom health monitor script (monitor.sh) that checks both a local service endpoint and internal logic, and returns appropriate exit codes (0 for healthy, 1 for unhealthy).

### Steps

#### 1. Create a health endpoint in the application
Assuming a simple Python/FastAPI app already exists on port 5001.
```python
@app.get("/health")
def health_check():
    return {"status": "ok"}
```

#### 2. Create the monitor script (/root/code/monitor.sh)
```bash
#!/bin/bash
STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:5001/health)

if [ "$STATUS" -eq 200 ]; then
  echo "healthy"
  exit 0
fi

echo "unhealthy"
exit 1
```

#### 3. Make it executable
```bash
chmod +x /root/code/monitor.sh
```

#### 4. Test the monitor
```bash
/root/code/monitor.sh
echo $?
```

#### 5. Final verification
```bash
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:5001/health
/root/code/monitor.sh
echo $?
```

---

## 📅 Day 31: Train a Scikit-Learn Model with Reproducible Script

### Task Description
The xFusionCorp Industries ML platform team maintains a config-driven training pipeline so hyperparameters can be swapped without editing Python code. The training scaffold exists at `/root/code/fraud-detection/` with the trainer already in place, but `configs/train_config.yaml` has been left in a broken state. The task is to fix the YAML config so one successful training run lands on the MLflow tracking server and the trained model ends up inside the project tree.

### Concept Summary
**Config-Driven Training** decouples hyperparameter choices from implementation code. The trainer reads all settings (estimator type, hyperparameters, file paths, MLflow coordinates) from a YAML file. This pattern means:
- Different runs can be compared by diffing config files, not code.
- Every MLflow run is reproducible given its config snapshot.
- An **Estimator Registry** (a Python dict mapping exact class-name strings to sklearn classes) makes config errors fail fast with a clear message instead of a confusing traceback.

### Step-by-Step Execution

**Step 1: Inspect the broken config**
```bash
cat /root/code/fraud-detection/configs/train_config.yaml
```
Three bugs were found:

| # | Field | Broken | Correct |
|---|-------|--------|---------|
| 1 | `model.type` | `RandomForest` | `RandomForestClassifier` |
| 2 | `data.target_column` | `target` | `is_fraud` |
| 3 | `output.model_path` | `/root/code/model.pkl` | `/root/code/fraud-detection/models/model.pkl` |

**Step 2: Fix `configs/train_config.yaml`**
```yaml
model:
  type: RandomForestClassifier
  n_estimators: 100
  max_depth: 5
  random_state: 42

data:
  train_path: /root/code/fraud-detection/data/train.csv
  target_column: is_fraud

output:
  model_path: /root/code/fraud-detection/models/model.pkl

mlflow:
  tracking_uri: http://localhost:5000
  experiment_name: fraud-detection
```

**Step 3: Run the trainer**
```bash
python3 /root/code/fraud-detection/src/models/train.py
```
Expected output:
```
accuracy=0.8000, f1_score=0.8261
model saved to /root/code/fraud-detection/models/model.pkl
```

**Step 4: Verify the model file**
```bash
ls -l /root/code/fraud-detection/models/model.pkl
```
---

## 📅 Day 32: Ensure Determinism and Reproducibility in ML Pipelines

### Task Description
Fix non-determinism in the model training script (`src/models/train.py`) to ensure that multiple training runs produce byte-identical metrics and feature importances. The success is measured by the `check_determinism.sh` script exiting with status 0.

### Concept Summary
**Determinism** in ML ensures that the same code and data always produce the same model and results. This is achieved by seeding **Pseudo-Random Number Generators (PRNGs)** at every stage where randomness is introduced, such as data shuffling, train-test splitting, and stochastic algorithm initialization (e.g., Random Forests).

### Step-by-Step Execution

**Step 1: Set a Global Seed**
Define a constant for the random state to ensure consistency across all library calls.
```python
RANDOM_STATE = 42
```

**Step 2: Seed the Data Partitioning**
Update the `train_test_split` function to use the fixed seed, preventing different data distributions across runs.
```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=RANDOM_STATE
)
```

**Step 3: Seed the Model Training**
Pass the random state to the estimator constructor. This ensures that the random feature and sample selection during tree building is repeatable.
```python
model = RandomForestClassifier(
    n_estimators=100, max_depth=5, random_state=RANDOM_STATE
)
```

**Step 4: Verify Reproducibility**
Run the determinism probe to confirm that three consecutive runs produce identical output metrics files.
```bash
./check_determinism.sh
```

**Step 5: Inspect Metrics**
Optionally check the generated JSON reports to verify they are byte-identical.
```bash
cat reports/metrics_run_1.json reports/metrics_run_2.json

---

## 📅 Day 33: Evaluate a Trained Model and Generate Classification Report

### Task Description
The xFusionCorp Industries ML platform team's release checklist requires a five-metric evaluation report for every candidate model, plus a confusion-matrix image, published to the project's reports/ directory. The goal is to correct the evaluate.py script to generate these reports in the correct location and log them to MLflow.

### Concept Summary
**Model Evaluation** in production environments requires more than just high-level metrics. A standard **Classification Report** provides precision, recall, and F1-score to detect biases or performance gaps. **Artifact Tracking** ensures that visual diagnostics like a **Confusion Matrix** are permanently linked to the specific model version that produced them, enabling better auditability and comparison.

### Step-by-Step Execution

**Step 1: Define Standardized Output Paths**
Ensure the script paths are configured to land in the project's internal reports/ directory rather than system temporary folders.
```python
REPORTS_DIR = "/root/code/fraud-detection/reports"
METRICS_JSON = os.path.join(REPORTS_DIR, "metrics.json")
CONFUSION_PNG = os.path.join(REPORTS_DIR, "confusion_matrix.png")
```

**Step 2: Expand the Metrics Dictionary**
Update the evaluation logic to calculate the full suite of required metrics: Accuracy, Precision, Recall, F1-Score, and AUC-ROC.
```python
metrics = {
    "accuracy": round(accuracy_score(y, preds), 6),
    "precision": round(precision_score(y, preds), 6),
    "recall": round(recall_score(y, preds), 6),
    "f1_score": round(f1_score(y, preds), 6),
    "auc_roc": round(roc_auc_score(y, proba), 6),
}
```

**Step 3: Run the Evaluator**
Execute the script to process the test set, generate local files, and log everything to the fraud-detection-eval MLflow experiment.
```bash
cd /root/code/fraud-detection
python src/models/evaluate.py
```

**Step 4: Verify the Results**
Confirm that the reports are created locally and visible in the MLflow UI.
- Check reports/metrics.json
- Check reports/confusion_matrix.png

---

## 📅 Day 35: Hyperparameter Tuning with Optuna

### Task Description
The xFusionCorp Industries ML platform team tunes fraud-detection hyperparameters with Optuna and inspects the full search in the MLflow Compare view. The goal is to correct the tuner so that each of the 20 trials is visible in MLflow and the best configuration saved corresponds to the highest-F1 candidate.

### Concept Summary
**Hyperparameter Tuning** with Optuna automates the search for optimal model parameters (like tree depth or number of estimators) using efficient sampling algorithms. By integrating it with **MLflow**, every trial in the search space can be tracked, visualized, and compared, ensuring that the model development process is transparent and reproducible.

### Step-by-Step Execution
**Step 1: Set Optimization Direction**
We update the Optuna study creation to "maximize" because we are optimizing for the F1 score, where a higher value is better.
```python
study = optuna.create_study(
    direction="maximize", 
    study_name=EXPERIMENT_NAME
)
```

**Step 2: Enable Per-Trial MLflow Logging**
We wrap the evaluation logic inside the `objective` function with an MLflow run block, ensuring every trial records its unique parameters and resulting score.
```python
with mlflow.start_run():
    mlflow.log_param("n_estimators", n_estimators)
    mlflow.log_param("max_depth", max_depth)
    mlflow.log_metric("f1_score", score)
```

**Step 3: Execute the Search**
Run the tuner script to perform 20 trials and identify the best hyperparameters.
```bash
python src/models/tune.py
```

**Step 4: Verify Best Parameters**
Check the output YAML file to ensure the best configuration has been persisted.
```bash
cat configs/best_params.yaml
```
---

## 📅 Day 36: Automated Model Selection with MLflow

### Task Description
The team needs to automate the selection of the best model from a competitive bake-off experiment. The goal is to fix two bugs in the orchestrator script: one that sorts models incorrectly (picking the worst instead of the best) and another that omits necessary model family metadata from the final JSON report.

### Concept Summary
**Automated Model Selection** is the process of programmatically comparing multiple trained models and selecting the optimal one based on predefined metrics. By using MLflow's search capabilities, we can ignore the noise of dozens of experiments and instantly identify the candidate with the highest performance (e.g., F1 score), ensuring that only the most accurate model is promoted to the next stage of the pipeline.

### Step-by-Step Execution
**Step 1: Execute Training Scripts**
Generate the candidate runs by executing the independent training scripts for Random Forest, Gradient Boosting, and Logistic Regression.
```bash
cd /root/code/fraud-detection
python src/models/train_rf.py
python src/models/train_gb.py
python src/models/train_lr.py
```

**Step 2: Correct Search Sorting**
Update the `mlflow.search_runs` call in `src/models/bakeoff.py` to sort by `f1_score DESC`. This ensures that the first row of the resulting dataframe is indeed the winner.
```python
runs = mlflow.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.f1_score DESC"],
    max_results=10,
)
```

**Step 3: Enrich the Winner Report**
Modify the report dictionary to include the `model_type` key, mapped from `winner["tags.candidate"]`, providing downstream systems with the model family information.
```python
report = {
    "model_type": winner["tags.candidate"],
    "run_id": winner["run_id"],
    "f1_score": float(winner["metrics.f1_score"]),
}
```

**Step 4: Verify the Selection**
Run the orchestrator and inspect the generated JSON file to confirm the correct model was selected.
```bash
python src/models/bakeoff.py
cat reports/winner.json
```

---

## 📅 Day 37: Promote the Winning Model to MLflow Model Registry

### Task Description
The team has a `reports/winner.json` from the bake-off experiment that identifies the best fraud-detection candidate. The goal is to fix a registration script at `src/models/register.py` — it reads the wrong key from the JSON report and calls a tag method instead of an alias method — so that the winning run is correctly registered under the `fraud-detector` model name and the new version is assigned the `champion` alias for use by downstream deployment pipelines.

### Concept Summary
The **MLflow Model Registry** is a centralized, versioned store for ML models that decouples a model's identity from a raw `run_id`. Every call to `register_model` creates an immutable **Model Version** snapshot. A **Model Alias** (e.g., `champion`) is a mutable, human-readable pointer that can be re-assigned to any version, allowing deployment scripts to always resolve the latest best model by a stable name (`models:/fraud-detector@champion`) rather than a hard-coded version number.

### Step-by-Step Execution
**Step 1: Fix the Report Key Lookup**
Correct the script to read the `run_id` key from `winner.json` (was incorrectly reading `winner["model"]`).
```python
with open("reports/winner.json") as f:
    winner = json.load(f)

run_id = winner["run_id"]   # Fixed: was winner["model"]
model_name = "fraud-detector"
```

**Step 2: Register the Model Version**
Use `mlflow.register_model` with a `runs:/` URI to link the new Registry entry to the logged artifact.
```python
model_uri = f"runs:/{run_id}/model"

mv = mlflow.register_model(
    model_uri=model_uri,
    name=model_name,
)
```

**Step 3: Assign the Champion Alias**
Replace the incorrect `set_model_version_tag` call with `set_registered_model_alias` so deployment systems can reference the model by alias.
```python
client = MlflowClient()

client.set_registered_model_alias(
    name=model_name,
    alias="champion",
    version=mv.version,
)
```

**Step 4: Run and Verify**
Execute the script and confirm registration via the MLflow client.
```bash
cd /root/code/fraud-detection
python src/models/register.py
```
```python
champion_mv = client.get_model_version_by_alias("fraud-detector", "champion")
print(champion_mv.version, champion_mv.run_id)
```

---

## 📅 Day 38: Serve a Registered MLflow Model as a REST API

### Task Description
The team has a registered `fraud-detector` model with a `champion` alias in the MLflow Model Registry (from Day 37). A serving script at `src/serve/predict.py` is broken — it loads the model using a hard-coded `run_id` URI instead of the stable `models:/fraud-detector@champion` alias URI, and the Flask route returns raw NumPy types that are not JSON-serializable. The goal is to correct both defects so the endpoint accepts a JSON feature payload and returns a valid prediction response.

### Concept Summary
The **`models:/` URI scheme** allows MLflow to resolve a model directly from the Registry using its registered name and either a version number or an alias (e.g., `models:/fraud-detector@champion`). This decouples the serving layer from raw `run_id` values — when the champion is re-promoted to a new version, the alias is simply reassigned and no serving code changes are required. Additionally, NumPy scalar types (such as `int64`) are not natively JSON-serializable; predictions must be cast to native Python builtins before being returned via Flask's `jsonify`.

### Step-by-Step Execution
**Step 1: Fix the Model Loading URI**
Replace the hard-coded `run_id` URI with the stable Registry alias URI so the server always loads the current champion.
```python
# Before (broken):
# model = mlflow.sklearn.load_model("runs:/abc123def456/model")

# After (correct):
model = mlflow.sklearn.load_model("models:/fraud-detector@champion")
```

**Step 2: Fix the Flask Prediction Route**
Cast the NumPy prediction result to a native Python `int` to ensure JSON serializability.
```python
from flask import Flask, request, jsonify
import pandas as pd
import mlflow.sklearn

app = Flask(__name__)
model = mlflow.sklearn.load_model("models:/fraud-detector@champion")

@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json(force=True)
    features = pd.DataFrame([data["features"]])
    prediction = model.predict(features)
    return jsonify({"prediction": int(prediction[0])})  # Fixed: cast to int

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

**Step 3: Start the Server and Verify**
Launch the Flask application and test it with a `curl` request.
```bash
cd /root/code/fraud-detection
python src/serve/predict.py &

curl -s -X POST http://localhost:8080/predict \
  -H "Content-Type: application/json" \
  -d '{"features": [1200.50, 1, 0, 3, 0.85, 22, 1, 0, 1, 0]}' | python3 -m json.tool
```

Expected response:
```json
{
    "prediction": 0
}
```

---

## 📅 Day 39: Parallelize Model Training and Compare Runs in MLflow

### Task Description
The xFusionCorp Industries ML platform team runs a parallel-training bake-off on the fraud-detection model. A draft script at `src/models/train_parallel.py` trains the same `RandomForestClassifier` twice — once serially and once in parallel — but both runs log near-identical wall times, and the MLflow Compare view cannot distinguish the two configurations because `n_jobs` is logged as a hardcoded string `"all"` instead of the actual integer, and the parallel run never actually sets `n_jobs=-1` on the classifier. The goal is to fix both defects so the bake-off produces genuinely distinct, measurable runs in the `parallel-training` experiment.

### Concept Summary
**`n_jobs=-1`** in scikit-learn instructs the estimator to use all available CPU cores. Each decision tree in a `RandomForestClassifier` is independent, making the ensemble trivially parallelizable via `joblib`. Setting `n_jobs=1` forces the serial baseline. **Wall-time measurement** (using `time.time()` before and after `fit()`) captures real-world training duration. In MLflow, `log_param` records the configuration value (an integer like `1` or `-1`) while `log_metric` records the numerical measurement (`training_time_seconds`). The **MLflow Compare view** renders a side-by-side table of all logged params and metrics across selected runs, making the configuration difference and performance delta immediately visible.

### Step-by-Step Execution
**Step 1: Identify the Two Defects in the Script**
The broken script logs a hardcoded string for `n_jobs` and fails to pass `n_jobs=-1` to the second classifier.
```python
# Defect 1 — wrong param value logged:
mlflow.log_param("n_jobs", "all")           # should be the integer variable

# Defect 2 — parallel classifier still single-threaded:
clf = RandomForestClassifier(n_estimators=500, random_state=42)  # missing n_jobs=-1
```

**Step 2: Fix the Serial Run Block**
Declare `n_jobs = 1`, pass it to the classifier, and log the integer.
```python
with mlflow.start_run(run_name="serial"):
    n_jobs = 1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)
    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start
    mlflow.log_param("n_jobs", n_jobs)           # logs integer 1
    mlflow.log_metric("training_time_seconds", elapsed)
```

**Step 3: Fix the Parallel Run Block**
Declare `n_jobs = -1`, pass it to the classifier, and log the integer.
```python
with mlflow.start_run(run_name="parallel"):
    n_jobs = -1
    clf = RandomForestClassifier(n_estimators=500, random_state=42, n_jobs=n_jobs)
    start = time.time()
    clf.fit(X_train, y_train)
    elapsed = time.time() - start
    mlflow.log_param("n_jobs", n_jobs)           # logs integer -1
    mlflow.log_metric("training_time_seconds", elapsed)
    with open("models/model.pkl", "wb") as f:
        pickle.dump(clf, f)
```

**Step 4: Run the Script**
```bash
cd /root/code/fraud-detection
export MLFLOW_TRACKING_URI=http://localhost:5000
python src/models/train_parallel.py
```

**Step 5: Verify in the MLflow Compare View**
Open the MLflow UI, navigate to the `parallel-training` experiment, select both runs, and click **Compare**. Confirm `params.n_jobs` shows `1` and `-1`, and that `metrics.training_time_seconds` for the parallel run is at least 10% lower than for the serial run.

**Step 6: Verify the Saved Model**
```bash
python3 -c "
import pickle
with open('models/model.pkl', 'rb') as f:
    model = pickle.load(f)
print('n_jobs:', model.n_jobs)
"
```

---

## 📅 Day 40: Production Training System: Tracking, Tuning, and Model Selection

### Task Description
The xFusionCorp Industries ML platform team has a five-stage production training pipeline (`validate_data → tune → select_model → register → report`) orchestrated by a `Makefile`. Running `make train-pipeline` surfaces three wiring bugs: the stages execute in the wrong order, the model-selection stage searches for a metric that the tuner never logs, and the registration stage assigns the wrong alias and fails to clean up stale aliases. The task is to identify each defect in turn, fix it with a one-line edit, and re-run until the full pipeline completes without a non-zero exit.

### Concept Summary
**Pipeline Stage Ordering** is the most fundamental concern in any multi-stage ML workflow. In a Makefile `train-pipeline` target, each shell command runs sequentially top to bottom. If `select_model` runs before `tune`, it queries an empty MLflow experiment and crashes — the fix is simply reordering the lines.

**Metric Consistency** across pipeline stages is equally critical. The tuning stage and the selection stage must agree on the exact MLflow metric key. If `tune.py` logs `metrics.f1_score` but `select_model.py` queries `metrics.accuracy`, the `search_runs` call returns an empty list and the pipeline fails silently.

**MLflow Model Aliases** (`staging`, `champion`, `production`) are mutable pointers that persist across pipeline runs. If a previous run left a `production` alias on the model, the next run sees it even after re-registering. Always explicitly delete stale aliases before assigning new ones.

### Step-by-Step Execution

**Step 1: Run the Pipeline to Surface the First Bug**
```bash
cd /root/code/fraud-detection
make train-pipeline
```
The pipeline exits non-zero because `select_model.py` ran before `tune.py` — no MLflow runs exist yet.

**Step 2: Fix the Makefile — Stage Execution Order**

Before (broken):
```makefile
train-pipeline:
python src/validate_data.py
python src/select_model.py
python src/tune.py
python src/register.py
python src/report.py
```

After (correct):
```makefile
train-pipeline:
python src/validate_data.py
python src/tune.py
python src/select_model.py
python src/register.py
python src/report.py
```

**Step 3: Re-Run and Fix `select_model.py` — Wrong Sort Metric**

Before (broken):
```python
runs = client.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.accuracy DESC"]
)
best_score = best.data.metrics["metrics.accuracy"]
```

After (correct):
```python
runs = client.search_runs(
    experiment_ids=[exp.experiment_id],
    order_by=["metrics.f1_score DESC"]
)
best_score = best.data.metrics["f1_score"]
```

**Step 4: Re-Run and Fix `register.py` — Wrong Alias and No Cleanup**

Before (broken):
```python
RELEASE_ALIAS = "production"
client.set_registered_model_alias(REGISTERED_MODEL_NAME, RELEASE_ALIAS, version.version)
```

After (correct):
```python
RELEASE_ALIAS = "staging"

for alias in ["production", "staging"]:
    try:
        client.delete_registered_model_alias(REGISTERED_MODEL_NAME, alias)
    except Exception:
        pass

client.set_registered_model_alias(REGISTERED_MODEL_NAME, RELEASE_ALIAS, version.version)
```

**Step 5: Final Clean Run and Verify**
```bash
make train-pipeline
```

Verify report files:
```bash
cat reports/training_report.json
```

Expected keys: `best_model`, `best_params`, `metrics`, `total_trials` (>= 5), `validation_status` ("ok").

Verify MLflow Registry alias:
```bash
python3 -c "
from mlflow import MlflowClient
client = MlflowClient()
aliases = client.get_registered_model('fraud-detector').aliases
print('Aliases:', aliases)
"
```

Expected: `Aliases: {'staging': '1'}` — no `production` key.

---

## 📅 Day 41: Install and Initialize a Feast Feature Store

### Task Description
The xFusionCorp Industries ML platform team is adopting Feast as the feature store for their fraud-detection workflow. The task is to initialize a new feature repository under `/root/code/`, apply the starter definitions to create the SQLite registry database, and verify the empty project loads in the Feast UI dashboard.

### Concept Summary
A **Feature Store** (like Feast) is an operational data system that manages and serves machine learning features consistently across offline training and online real-time inference.

- **`feast init`**: Scaffolds a boilerplate feature repository with standard configurations and entity/feature definitions.
- **`feast apply`**: Scans the Python source files in the repository, compiles the features metadata registry, and sets up any database tables or paths needed by the online/offline providers.
- **Feast Registry (`registry.db`)**: A lightweight SQLite metadata catalog storing schemas, views, source data references, and active definitions.

### Step-by-Step Execution

**Step 1: Check Feast CLI Version**
Confirm that the Feast tool matches expected environments.
```bash
feast version
```

**Step 2: Initialize the Repository**
Generate the starter template workspace under `/root/code/`:
```bash
cd /root/code
feast init feature_repo
```

**Step 3: Apply the Feature Schema**
Compile the boilerplate definitions into the local SQLite metadata database:
```bash
cd /root/code/feature_repo/feature_repo
feast apply
```
This generates the SQLite registry database at `/root/code/feature_repo/feature_repo/data/registry.db`.

**Step 4: Launch the Feast UI Web Server**
Start the Feast UI dashboard server in the background so the console remains active:
```bash
feast ui &
```
The server binds to port 8888 by default. You can open `http://127.0.0.1:8888` or click the workspace shortcut to explore registered feature views, datasets, and entities.

---

## 📅 Day 42: Define Feature Views in Feast

### Task Description
The team keeps the fraud-detection feature definitions in a Feast repository at `/root/code/fraud-detection/feature_repo/`. The schema stored in the metadata registry is inconsistent with the source parquet file at `data/transactions.parquet`. The task is to correct `features.py` to match the schema (set entity join key to `customer_id` and the `amount` feature dtype to `Float32`), re-apply the registry using `feast apply`, and check the entities and feature views in the Feast UI.

### Concept Summary
In **Feast**, an **Entity** acts as a primary or join key to associate features with specific records (like a customer ID). A **Feature View** coordinates feature declarations, data sources, schemas, and time-to-live settings. Mismatches between declared metadata types (such as `Float64`) and actual source datatypes (like `Float32`) result in validation or inference errors, requiring strict alignment in Feast's python specifications.

### Step-by-Step Execution

**Step 1: Open and Edit features.py**
Open `/root/code/fraud-detection/feature_repo/features.py` and modify the customer entity definition's `join_keys` and the amount field `dtype` in the feature view schema.
- Set `join_keys=["customer_id"]`
- Set `Field(name="amount", dtype=Float32)`

**Step 2: Apply the Updated Schema Registry**
Navigate to the directory and run `feast apply` to rebuild/update the SQLite registry database (`data/registry.db`).
```bash
cd /root/code/fraud-detection/feature_repo/
feast apply
```

**Step 3: Verify via Feast UI**
Launch or navigate to the running Feast UI (port `8888`) and confirm that the customer entity lists `customer_id` as the join key and the `customer_transaction_features` view lists `amount` with type `Float32`.

---

## 📅 Day 43: Materialize Features to the Online Store

### Task Description
The team stages a materialization script (`materialize.sh`) under `/root/code/fraud-detection/feature_repo/` to populate the Feast online store. Since the parquet source files start in 2024, the script's raw `END_DATE` setting of `1970` writes zero rows. The task is to set a correct `END_DATE` to a date on or after the last event in parquet (such as `2025-12-31T23:59:59`), run the materialization script to populate `data/online_store.db`, and query Feast's client SDK to confirm non-null amount features are returned.

### Concept Summary
**Materialization** transfers data from the offline raw sources (parquet/databases) to the online low-latency key-value store (SQLite/Redis) so real-time production engines can query features using entity keys. The command `feast materialize-incremental` target windows with watermarks using the entity view's TTL back from the provided end-date parameter, preventing redundant updates.

### Step-by-Step Execution

**Step 1: Correct materialize.sh**
Open `/root/code/fraud-detection/feature_repo/materialize.sh` and set the target date window to include 2024 data:
- Set `END_DATE="2025-12-31T23:59:59"`

**Step 2: Run Materialization**
Make the shell script executable and run it to populate the SQLite database:
```bash
cd /root/code/fraud-detection/feature_repo
chmod +x materialize.sh
./materialize.sh
```

**Step 3: Verification online querying**
Run Python interactive commands to query the Feast SDK:
```bash
python3
```
```python
from feast import FeatureStore
store = FeatureStore(repo_path=".")
print(store.get_online_features(
    features=["customer_transaction_features:amount"],
    entity_rows=[{"customer_id": 1}],
).to_dict())
```
Ensure retrieval displays the correct actual attribute value instead of `None`.

---

## 📅 Day 44: Store MLflow's Admin Password in HashiCorp Vault

### Task Description
The team wants credentials retrieved dynamically from Vault rather than being hardcoded. An MLflow boot wrapper polls Vault for the secret `secret/mlflow.admin_password` every 5 seconds to launch the server once it exists. The task is to log into Vault (port `8200`) using the root token from `/root/code/vault-token`, enable the KV v2 engine at `secret/`, create the `admin_password` key under path `secret/mlflow`, and confirm MLflow launches successfully on port `5000`.

### Concept Summary
**HashiCorp Vault** is a secure engine for storing secrets and credentials. Spawning containers or microservices using a **Vault-First Pattern** (where a startup wrapper extracts secrets from Vault in real-time) eliminates plaintext secrets inside configurations or docker commits.

### Step-by-Step Execution

**Step 1: Configure Vault Authentication**
Set environmental target variables and retrieve the dev root token:
```bash
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=$(cat /root/code/vault-token)
```

**Step 2: Enable Key-Value Version 2 Engine**
Mount the v2 Key-Value secrets capability at path `secret/`:
```bash
vault secrets enable -path=secret -version=2 kv
```

**Step 3: Save MLflow Administrator Secret**
Populate the engine with the required key-value pair under path `secret/mlflow`:
```bash
vault kv put secret/mlflow admin_password='Admin@123'
```

**Step 4: Verify Server Boot**
Allow 5–10 seconds for the polling script wrapper to detect the secret. Perform an HTTP check to verify the live MLflow service:
```bash
curl -I http://127.0.0.1:5000/
```
Verify that the server returns HTTP/1.1 200 OK.

---

## 📅 Day 45: Fix a Broken Vault KV Policy

### Task Description
The team switched the MLflow boot wrapper from a root token to a narrow `mlflow-reader` application token. However, the associated policy was compiled with write/update permissions instead of read capabilities on the credentials vault path. The task is to access Vault on port `8200` using the root credentials, edit the `mlflow-reader` policy rules to grant the `read` capability on `secret/data/mlflow`, and confirm that the boot wrapper successfully launches MLflow on port `5000`.

### Concept Summary
Vault enforces access controls using **ACL Policies** written in HCL. Tokens are issued with specific policies attached to authorize capabilities. The key-value v2 engine routes data requests via a `/data/` infix (e.g. `secret/data/mlflow` for secret `secret/mlflow`). A read operations requires the `read` capability, and a mismatch results in permission denied errors.

### Step-by-Step Execution

**Step 1: Authenticate with Vault UI**
Access the Vault UI (port `8200`) using the root token from `/root/code/vault-root-token`.

**Step 2: Modify mlflow-reader Policy**
Locate the `mlflow-reader` configuration under the **Policies** tab and edit it to change capabilities:
- Replace `capabilities = ["create", "update"]` with `capabilities = ["read"]` on path `secret/data/mlflow`.
- Click **Save**.

**Step 3: Verify Connection**
Verify that the narrow database token (`/root/code/vault-token`) can retrieve the configuration:
```bash
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN=$(cat /root/code/vault-token)
vault kv get secret/mlflow
```

**Step 4: Check MLflow Server**
Wait 5 seconds and verification connection using HTTP checking:
```bash
curl -I http://127.0.0.1:5000/
```
Output must return HTTP/1.1 200 OK.

---

## 📅 Day 46: Author Data-Quality Expectations with Great Expectations

### Task Description
The team at xFusionCorp Industries wants data-schema contracts on every batch that feeds the fraud-detector model to catch malformed rows upstream of training. The task is to author a Great Expectations expectation suite (`fraud_schema`) by configuring four predefined assertions inside `/root/code/dataquality/author_expectations.py` and run the default checkpoint to validate client transaction logs and update Data Docs.

### Concept Summary
Great Expectations (GX) is an open-source library that treats data quality as code. Users configure validation rules (called **Expectations**) grouped into **Expectation Suites** that define inputs and validation settings. By version-controlling these schemas alongside normal source packages and executing them in orchestration workflows, systems establish an automated gate to intercept corrupted schema changes beforehand. Validation outputs are persisted inside machine-readable JSON logs and rendered to human-navigable HTML web portals called **Data Docs**.

### Step-by-Step Execution

**Step 1: Open the Script**
Open the expectations authoring script `/root/code/dataquality/author_expectations.py` inside the VS Code editor.

**Step 2: Declare Expected Column Schema**
Add an expectation to verify that the table has the required set of matching columns:
```python
suite.add_expectation(
    ge.ExpectTableColumnsToMatchSet(
        column_set=["amount", "hour", "num_tx_past_day", "is_fraud"]
    )
)
```

**Step 3: Guard transaction amount**
Declare that transaction amount must not be negative:
```python
suite.add_expectation(
    ge.ExpectColumnValuesToBeBetween(
        column="amount",
        min_value=0,
    )
)
```

**Step 4: Guard hour Range**
Constrain hour values to a valid 24-hour window (0 to 23):
```python
suite.add_expectation(
    ge.ExpectColumnValuesToBeBetween(
        column="hour",
        min_value=0,
        max_value=23,
    )
)
```

**Step 5: Guard is_fraud Variable Domain**
Restrict the target flag schema to binary classification classes (0 and 1):
```python
suite.add_expectation(
    ge.ExpectColumnValuesToBeInSet(
        column="is_fraud",
        value_set=[0, 1],
    )
)
```

**Step 6: Run the Expectations Authoring Pipeline**
Execute the script to serialize expectations to disk and run the checkpoint to validate raw client transaction logs against the schema rules:
```bash
python3 /root/code/dataquality/author_expectations.py
```
This updates `gx/expectations/fraud_schema.json`, triggers validation, and generates updated logs under `gx/uncommitted/validations/` with `success: true`.

---

## 📅 Day 47: Debug a Failing Great Expectations Checkpoint

### Task Description
The team at xFusionCorp Industries extended the validation checks to a second batch of transaction records containing negative values (`data/transactions_drifted.csv`). The new `drift_check` checkpoint runs the existing `fraud_schema` suite but fails due to out-of-bounds observations. The task is to inspect Data Docs on port `8081` to pinpoint the failing schema expectation, modify `/root/code/dataquality/fix_drift.py` to widen the constraint to support drifted values down to `-400` with safe margins, and execute the configuration to restore successful validation runs.

### Concept Summary
Data drift represents alterations in the statistical properties and distribution boundaries of incoming files over time. In Great Expectations, when production data falls outside declared rules, check runs flag a Failure on the dashboard. In response, teams must evaluate whether the data quality is compromised, or whether boundaries must be expanded (known as **Bound Widening**) to support real-life drift without completely deleting the underlying rules/assertions.

### Step-by-Step Execution

**Step 1: Inspect validation reports in Data Docs**
Access the web dashboard on port `8081` and review the failing `drift_check` checkpoint run.
* **Observe the failing details**: Column `amount` fails because values fall below `0`. The minimum observed value is reported as `-347.22`.

**Step 2: Widen bounds in the fix script**
Open the fix script at `/root/code/dataquality/fix_drift.py` and modify the range expectation for the `amount` column from `min_value=0` to `min_value=-400`:
```python
suite.add_expectation(
    ge.ExpectColumnValuesToBeBetween(
        column="amount",
        min_value=-400,
    )
)
```

**Step 3: Run the check script**
Execute the configuration setup to update `gx/expectations/fraud_schema.json` and validate transaction logs:
```bash
python3 /root/code/dataquality/fix_drift.py
```
After execution, the checkpoint reports success with `success=True`.

**Step 4: Check index in Data Docs**
Refresh your browser index on port `8081` and verify that the newest validation run for `drift_check` is shown in green as Successful.


# Day 48: Publish Great Expectations Data Docs as a CI Artifact

# Introduction

In modern software development, CI (Continuous Integration) pipelines do much more than compile code. They also execute automated tests, security scans, code quality checks, and data validation.

For Machine Learning projects, validating the **quality of data** is just as important as validating source code. If poor-quality or unexpected data enters the system, the ML model may produce incorrect predictions.

To prevent this, the repository uses **Great Expectations**, a popular Python framework for validating datasets.

---

# What is Great Expectations?

Great Expectations (GE) is an open-source Python library used to validate data.

Instead of manually checking data, developers define **expectations**.

Examples:

- Column should not contain null values.
- Age must be greater than 0.
- Salary should be within an expected range.
- Number of rows should not suddenly decrease.
- Data types should remain unchanged.

When GE executes these expectations, it generates:

- Validation results
- HTML reports
- Data Docs

---

# What are Data Docs?

Data Docs are HTML reports generated by Great Expectations.

Think of them as a website explaining the health of your dataset.

Instead of seeing:

```
Validation Passed
```

you see a detailed report containing:

- Which expectations passed
- Which expectations failed
- Dataset statistics
- Validation history
- Detailed explanations

Example:

```
Validation Report

✓ Row count matches expectation

✓ No missing values

✗ Column "transaction_amount"
  Unexpected values found
```

These reports help reviewers understand *why* validation passed or failed.

---

# Problem Statement

The workflow already generated Data Docs.

The command

```bash
python3 -m src.gx_run
```

created the reports inside

```
gx/uncommitted/data_docs/local_site/
```

However, the workflow runs inside a temporary runner.

A runner behaves like a temporary virtual machine.

Example:

```
Workflow starts

↓

Repository is cloned

↓

Commands execute

↓

Workflow ends

↓

Runner is destroyed
```

When the runner is destroyed, every generated file disappears.

Therefore, reviewers cannot access the generated Data Docs.

The workflow only displays

```
✓ Success
```

or

```
✗ Failure
```

without showing the actual validation report.

---

# Why is this a problem?

Imagine a reviewer receives a failing CI check.

Without Data Docs, they only know:

```
Validation Failed
```

Questions remain unanswered:

- Which expectation failed?
- Which column caused the issue?
- Is this an actual data problem?
- Is the expectation outdated?

Without the HTML report, developers cannot investigate easily.

---

# Solution

The solution is to upload the generated Data Docs before the runner is deleted.

GitHub Actions (and Gitea Actions) provide **workflow artifacts**.

Artifacts are files preserved after the workflow completes.

Instead of disappearing, they become downloadable from the Actions page.

---

# What is a Workflow Artifact?

An artifact is simply a collection of files saved after a workflow finishes.

Example:

Workflow generates

```
report.pdf
coverage.html
logs/
screenshots/
```

If uploaded as an artifact,

they remain available even after the runner is deleted.

Developers can download them later.

Artifacts are commonly used for:

- Build outputs
- Test reports
- Code coverage reports
- Logs
- Data validation reports

---

# Uploading Artifacts

GitHub Actions provides a predefined action:

```yaml
actions/upload-artifact
```

This action copies files from the runner and stores them with the workflow.

Syntax:

```yaml
- uses: actions/upload-artifact@v3
```

This tells GitHub/Gitea:

> "Upload files generated during this workflow."

---

# Understanding the Workflow Step

The added step was:

```yaml
- name: Upload Data Docs
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: data-docs
    path: gx/uncommitted/data_docs/
```

Let's understand every line.

---

## name

```yaml
- name: Upload Data Docs
```

This is only the display name.

It appears in the workflow logs.

Example:

```
✓ Upload Data Docs
```

---

## uses

```yaml
uses: actions/upload-artifact@v3
```

There are two ways to perform work inside a workflow.

### Method 1

Run shell commands.

Example:

```yaml
run: python3 app.py
```

This executes commands directly.

---

### Method 2

Use an existing GitHub Action.

Example:

```yaml
uses: actions/checkout@v4
```

Instead of writing code ourselves, we reuse an existing action.

Here we reused

```
actions/upload-artifact
```

which already knows how to upload files.

---

## Version

```yaml
@v3
```

Actions are versioned.

Examples:

```
v1

v2

v3

v4
```

Using a fixed version ensures consistent behavior.

In this lab,

```
v3
```

was compatible with the provided Gitea Actions environment.

---

## if: always()

Normally,

if a workflow step fails,

later steps are skipped.

Example:

```
Step 1

↓

Step 2

↓

Step 3 fails

↓

Workflow stops
```

No artifact gets uploaded.

This is undesirable because we may still want to inspect the generated reports.

Therefore,

```yaml
if: always()
```

forces the upload step to execute regardless of previous failures.

Even if validation fails,

the HTML report is still preserved.

---

## with

Actions receive parameters using

```yaml
with:
```

Think of it like passing arguments to a function.

---

## Artifact Name

```yaml
name: data-docs
```

This becomes the downloadable artifact name.

Actions page:

```
Artifacts

data-docs
```

---

## Path

```yaml
path: gx/uncommitted/data_docs/
```

This tells the upload action

where the files are located.

The action compresses everything inside that directory.

Example:

```
gx

└── uncommitted

    └── data_docs

        └── local_site

            index.html

            validations/

            expectations/

            static/
```

Everything gets uploaded.

---

# Why upload the entire folder?

Instead of uploading only

```
index.html
```

we upload the complete directory.

Reason:

The HTML page depends on other files.

Example:

```
CSS

JavaScript

Images

Validation Pages
```

If only

```
index.html
```

is uploaded,

the website may not display correctly.

Uploading the entire folder preserves the complete Data Docs website.

---

# Git Operations

After modifying the workflow,

Git tracks the change.

Stage:

```bash
git add .gitea/workflows/data-quality.yml
```

Commit:

```bash
git commit -m "Publish Great Expectations Data Docs as CI artifact"
```

Push:

```bash
git push origin add-data-docs-artefact
```

Pushing triggers the Pull Request workflow automatically.

---

# Workflow Execution

After pushing,

the workflow executed these steps:

```
Checkout Repository

↓

Install Great Expectations

↓

Run Validation

↓

Generate Data Docs

↓

Upload Data Docs

↓

Workflow Finished
```

---

# Verification

The Pull Request displayed:

```
All checks were successful
```

The workflow page displayed:

```
Upload Data Docs

✓ Success
```

The Actions page displayed:

```
Artifacts

data-docs
```

This confirmed the Data Docs were successfully preserved.

---

# Real-World Importance

In Machine Learning projects, a simple "Pass" or "Fail" is often not enough.

Data validation reports help reviewers answer questions such as:

- Which expectation failed?
- What values caused the failure?
- Is the data actually wrong?
- Should the expectation be updated?

Uploading Data Docs as an artifact makes these reports available long after the workflow has finished, improving collaboration, debugging, and auditability.

---

# Key Takeaways

- Great Expectations validates datasets against predefined expectations.
- Data Docs are HTML reports generated by Great Expectations.
- CI runners are temporary; generated files are deleted unless preserved.
- Workflow artifacts store files after the workflow completes.
- `actions/upload-artifact@v3` uploads files generated during the workflow.
- `if: always()` ensures artifacts are uploaded even if earlier steps fail.
- Uploading the entire `gx/uncommitted/data_docs/` directory preserves the complete Data Docs site.
- Reviewers can download the `data-docs` artifact from the workflow run to inspect validation results.

# Task Description

Day 49 focused on building a complete **end-to-end MLOps release pipeline** by integrating four different platforms—**HashiCorp Vault**, **Gitea Actions**, **Great Expectations**, and **MLflow**. Unlike previous labs where each tool was used independently, this capstone demonstrated how multiple enterprise tools work together during a real production deployment.

The workflow represented a common scenario in modern organizations. A developer completes a feature, raises a Pull Request, and an automated pipeline validates every requirement before allowing the code to be merged. Instead of only testing source code, the pipeline also validates dataset quality, securely retrieves application secrets, registers a machine learning model, and finally promotes the model for production use.

The entire workflow was designed to fail immediately if any prerequisite was missing. For example, if the MLflow password was not stored inside Vault, the first job would fail, preventing the remaining jobs from executing. This demonstrates an important DevOps principle: **fail fast to prevent faulty deployments from reaching production.**

---

# Concept Summary

## Why was this lab called a Capstone?

A capstone project combines everything learned throughout a course into one practical implementation.

Earlier labs introduced tools individually:

- HashiCorp Vault for secrets management
- Gitea for Git repository management
- Great Expectations for data validation
- MLflow for experiment tracking and model registry

In this lab, all of those tools worked together inside one automated CI/CD workflow.

Instead of learning individual commands, we learned **how enterprise systems communicate with one another.**

---

# Understanding the Complete Workflow

The production release pipeline consisted of three dependent jobs.

```
fetch-secret
      │
      ▼
data-quality
      │
      ▼
register-model
```

Notice that each job depends on the previous one.

If the first job fails, the second job never starts.

If the second job fails, the third job never starts.

This dependency chain guarantees that production releases occur only after every validation succeeds.

---

# Job 1 – fetch-secret

The first job is responsible for securely obtaining the MLflow credential.

Many beginners wonder:

> Why not simply store the password inside the workflow?

That would be extremely insecure.

Imagine pushing this workflow to GitHub.

If the password were written directly inside the YAML file,

```
password = my-secret-password
```

every developer could see it.

Even worse, Git permanently stores commit history.

Deleting the password later does not remove it from Git history.

For this reason, production environments never store secrets inside repositories.

Instead,

```
Application
      │
      ▼
Vault
      │
      ▼
Returns Secret
```

The application asks Vault for the password during execution.

The password never becomes part of the source code.

---

# HashiCorp Vault

HashiCorp Vault is a centralized secrets management platform.

Instead of storing credentials in configuration files, applications request them only when required.

Vault commonly stores

- Database passwords
- Cloud credentials
- API Keys
- SSH Keys
- Certificates
- OAuth Tokens
- Encryption Keys

A company may have thousands of applications.

Without Vault,

every application stores its own passwords.

With Vault,

every application requests credentials from one secure location.

---

# KV Version 2

Vault supports multiple storage engines.

This lab used

```
KV Version 2
```

One of the most confusing concepts for beginners is the difference between

```
CLI Path

secret/mlflow
```

and

```
REST API Path

secret/data/mlflow
```

Why are they different?

The Vault CLI automatically inserts

```
data/
```

behind the scenes.

When we call the REST API ourselves,

we must specify the complete endpoint.

Therefore,

```
vault kv get secret/mlflow
```

internally becomes

```
GET

/v1/secret/data/mlflow
```

This is why the workflow accessed

```
secret/data/mlflow
```

instead of

```
secret/mlflow
```

---

# Why did we use curl?

The instructions specifically asked us to perform

```
GET $VAULT_ADDR/v1/secret/data/mlflow
```

Instead of using

```
vault kv get
```

This teaches an important concept.

Almost every DevOps tool exposes a REST API.

The CLI is simply another client built on top of that API.

Learning the REST API allows automation from

- Bash
- Python
- Go
- Java
- Jenkins
- GitHub Actions
- GitLab CI

without installing the official CLI.

---

# Understanding the JSON Response

Vault returns JSON.

Example

```json
{
  "data": {
    "data": {
      "mlflow_password": "my-secret-password"
    }
  }
}
```

Many students ask,

> Why are there two "data" objects?

The outer `data` contains metadata about the secret.

The inner `data` contains the actual secret values.

Therefore the workflow extracted

```
.data.data.mlflow_password
```

If the password was empty,

the workflow executed

```bash
echo "::error::mlflow_password not found"
exit 1
```

which immediately stopped the release.

---

# Job 2 – Data Quality

After retrieving the secret successfully,

the workflow executed

```
schema_check
```

using Great Expectations.

Many people think testing only applies to source code.

Machine Learning introduces another problem.

Even if the Python code is perfect,

the dataset itself may contain problems.

For example,

```
Age

20
25
NULL
18
```

or

```
Income

1000
2500
text
4500
```

The model may train incorrectly or fail entirely.

Great Expectations detects these issues before training begins.

Typical validations include

- Required columns exist
- Correct data types
- Missing value detection
- Schema validation
- Range validation
- Null percentage
- Duplicate rows

Instead of debugging after deployment,

the pipeline stops before a bad dataset reaches production.

---

# Data Docs

One useful feature of Great Expectations is

```
Data Docs
```

Instead of printing logs,

it generates an HTML report.

This report explains

- Which expectations passed
- Which expectations failed
- Dataset statistics
- Validation history

Data engineers usually review this report before approving production releases.

---

# Job 3 – Register Model

After data validation,

the workflow registered the trained model inside MLflow.

Model Registry acts similarly to Git,

except instead of storing source code,

it stores machine learning models.

Example

```
fraud-detector

Version 1

Version 2

Version 3
```

Each registration creates another version.

Older versions remain available for rollback.

---

# Why use Model Versions?

Imagine Version 5 performs poorly after deployment.

Instead of retraining,

we can simply return to

```
Version 4
```

This makes rollback fast and reliable.

Versioning is one of the biggest advantages of MLflow.

---

# Production Alias

The workflow ultimately produced

```
fraud-detector

Version 1

Alias

@production
```

Many beginners ask,

> Why not simply use Version 1 everywhere?

Suppose Version 2 becomes better.

Without aliases,

every application must change

```
Version 1
```

to

```
Version 2
```

across multiple services.

Instead,

applications simply load

```
@production
```

Today

```
@production

↓

Version 1
```

Tomorrow

```
@production

↓

Version 2
```

Applications never change.

Only the alias changes.

This makes deployments significantly easier.

---

# Pull Request Based Deployment

One of the most important DevOps concepts demonstrated in this lab is

```
Feature Branch

↓

Pull Request

↓

Automated Checks

↓

Merge

↓

Production
```

No developer can merge code directly into the main branch.

Every Pull Request must pass automated validation.

Those validations may include

- Unit tests
- Integration tests
- Security scans
- Secret verification
- Data quality checks
- Model registration

Only after all checks pass can the Pull Request be merged.

This protects production from broken releases.

---

# Overall Architecture

```
Developer

      │

      ▼

Push Feature Branch

      │

      ▼

Create Pull Request

      │

      ▼

Gitea Actions

      │

      ▼

Read Secret From Vault

      │

      ▼

Validate Dataset

      │

      ▼

Register Model

      │

      ▼

Merge Pull Request

      │

      ▼

Production Model
```

Every stage performs one responsibility.

Each stage must succeed before the next stage begins.

This layered validation greatly improves production reliability.

---

# Interview Questions

### Why should secrets never be stored inside Git?

Because Git history is permanent. Even if the password is deleted later, it can still be recovered from previous commits. Secret management tools like Vault prevent sensitive information from being stored in repositories.

---

### What is the difference between `secret/mlflow` and `secret/data/mlflow`?

`secret/mlflow` is the Vault CLI path.

`secret/data/mlflow` is the REST API endpoint used by KV Version 2.

---

### Why did the workflow use the Vault REST API instead of the Vault CLI?

The lab was designed to demonstrate direct API usage. Most automation tools interact with services through REST APIs, making this approach more portable and suitable for CI/CD environments.

---

### Why is data validation performed before model registration?

A model trained on poor-quality data may produce inaccurate predictions. Validating datasets before training ensures that only reliable data is used.

---

### What is the purpose of MLflow aliases?

Aliases provide stable names such as `production` or `staging` that always point to the desired model version. Applications reference the alias instead of hardcoding version numbers.

---

### What was the most important lesson from this capstone?

A production release is much more than deploying code. Modern MLOps pipelines combine **security, automated testing, data validation, version control, and model lifecycle management** to ensure every deployment is reliable, secure, and production-ready.

---

# Step-by-Step Execution

1. Configured the Vault environment variables.
2. Stored the `mlflow_password` secret inside `secret/mlflow`.
3. Edited `production.yml` and implemented the Vault secret retrieval logic.
4. Committed and pushed the changes to the `production-release` branch.
5. Created a Pull Request from `production-release` to `main`.
6. Verified successful execution of `fetch-secret`, `data-quality`, and `register-model`.
7. Confirmed that Great Expectations generated the Data Docs report.
8. Merged the Pull Request after all checks passed.
9. Verified that the `fraud-detector` model was registered in MLflow.
10. Confirmed that the `@production` alias pointed to Version 1 of the model.

# Day 50 Notes – Docker Image for ML Training Environment

## Introduction

In machine learning projects, different developers often use different operating systems, Python versions, or package versions. These differences can cause the same code to behave differently.

Docker solves this problem by packaging the application, Python runtime, libraries, and dependencies into a single portable image. Anyone running the image gets the same environment and therefore the same results.

---

# What is a Docker Image?

A Docker image is a blueprint for creating containers.

It contains:

- Operating system libraries
- Programming language runtime
- Required packages
- Application source code
- Default startup command

Once an image is built, it can be shared and executed on any machine with Docker installed.

---

# Dockerfile

A **Dockerfile** is a text file that contains instructions Docker follows to build an image.

Each instruction creates a separate layer, allowing Docker to cache unchanged layers and speed up future builds.

Example:

```dockerfile
FROM python:3.11-slim
WORKDIR /app
RUN pip install ...
COPY train.py /app/train.py
CMD ["python3", "train.py"]
```

---

# FROM

```dockerfile
FROM python:3.11-slim
```

This specifies the base image.

We use **python:3.11-slim** because:

- Official Python image
- Lightweight
- Compatible with scikit-learn
- Faster downloads
- Smaller image size

Avoid Alpine Linux for this project because many ML libraries do not provide compatible wheels and may require slow source compilation.

---

# WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory inside the container.

Every subsequent instruction executes relative to this directory.

Instead of repeatedly writing:

```
/app/train.py
```

Docker automatically works inside `/app`.

---

# Installing Dependencies

```dockerfile
RUN pip install --no-cache-dir \
    scikit-learn \
    pandas \
    numpy \
    joblib
```

The `RUN` instruction executes commands while building the image.

The `--no-cache-dir` option prevents pip from storing package caches, reducing image size.

Installed packages:

| Package | Purpose |
|----------|---------|
| numpy | Numerical computations |
| pandas | Data manipulation |
| scikit-learn | Machine learning algorithms |
| joblib | Saving trained models |

---

# COPY

```dockerfile
COPY train.py /app/train.py
```

Copies files from the host machine into the Docker image.

Source:

```
train.py
```

Destination:

```
/app/train.py
```

Without this instruction, the container would not contain the training script.

---

# CMD

```dockerfile
CMD ["python3", "train.py"]
```

Defines the default command executed when the container starts.

Running:

```bash
docker run ml-trainer:v1
```

automatically executes:

```bash
python3 train.py
```

---

# Building the Image

```bash
docker build -t ml-trainer:v1 .
```

Explanation:

| Option | Meaning |
|---------|----------|
| docker build | Builds an image |
| -t | Assigns a tag |
| ml-trainer:v1 | Image name and version |
| . | Current directory as build context |

---

# Viewing Images

```bash
docker images
```

Displays all Docker images available on the local system.

To check only this image:

```bash
docker images ml-trainer:v1
```

---

# Running the Container

Execute the training script:

```bash
docker run --rm ml-trainer:v1
```

`--rm` removes the container after it exits, keeping the system clean.

---

# Testing Installed Libraries

Verify that all required libraries are available:

```bash
docker run --rm ml-trainer:v1 python3 -c "import sklearn, pandas, numpy, joblib; print('OK')"
```

Expected output:

```
OK
```

This confirms that every required Python package is installed correctly.

---

# Docker Build Cache

Docker caches each instruction as a separate layer.

For example:

```
FROM
↓
WORKDIR
↓
RUN pip install
↓
COPY
↓
CMD
```

If only `train.py` changes, Docker reuses the cached layers (`FROM`, `WORKDIR`, and `RUN pip install`) and rebuilds only the `COPY` and `CMD` steps, making subsequent builds much faster.

---

# Benefits of Using Docker for Machine Learning

- Consistent development environments
- Easy deployment across systems
- Dependency isolation
- Reproducible experiments
- Faster onboarding for new team members
- Simplified collaboration

---

# Key Takeaways

- Docker packages applications with all required dependencies.
- A Dockerfile defines how an image is built.
- `FROM` selects the base image.
- `WORKDIR` sets the working directory.
- `RUN` installs dependencies.
- `COPY` adds application files to the image.
- `CMD` specifies the default command executed when the container starts.
- Docker layer caching speeds up repeated builds by reusing unchanged instructions.

# Day 51 Notes: Multi-Stage Docker Builds for Machine Learning Model Serving

# Introduction

One of the biggest mistakes developers make while containerizing Machine Learning applications is shipping everything into the production image.

A typical ML project contains two completely different phases:

1. **Training Phase**
2. **Serving (Inference) Phase**

These phases have different requirements.

During training we may require:

- pandas
- matplotlib
- seaborn
- jupyter
- xgboost
- tensorflow
- pytorch
- datasets
- training scripts
- notebooks

But after the model is trained, none of these are needed to make predictions.

The serving application only needs:

- the trained model
- the web server
- minimum runtime libraries

This is exactly why Docker provides **Multi-Stage Builds**.

---

# Understanding the Project

The project contains three files.

```
ml-serve/
│
├── Dockerfile
├── train_model.py
└── serve.py
```

Each file has a different responsibility.

---

# train_model.py

This file is responsible for **training** the Machine Learning model.

Its job is:

```
Dataset
      │
      ▼
Train RandomForest
      │
      ▼
Save model.pkl
```

It does **NOT** serve HTTP requests.

It only creates the trained model.

Internally it performs something similar to:

```python
model = RandomForestClassifier(n_estimators=10)

model.fit(X, y)

joblib.dump(model, "model.pkl")
```

Notice the important part:

```
joblib.dump(...)
```

This serializes (saves) the trained model into a file.

That file is

```
model.pkl
```

This file becomes the most important artifact.

---

# What is model.pkl?

Machine Learning models exist only in RAM while training.

For example

```
model.fit(...)
```

creates a trained model inside memory.

If the program exits

everything disappears.

So we serialize the model.

```
RAM
 │
 ▼
joblib.dump()
 │
 ▼
model.pkl
```

Now the trained model can be loaded later without training again.

---

# Why use Joblib?

Machine learning models can contain

- thousands
- millions
- even billions

of parameters.

Python's default pickle works.

But Joblib is optimized for large NumPy arrays.

Therefore Scikit-Learn officially recommends Joblib.

Saving

```python
joblib.dump(model, "model.pkl")
```

Loading

```python
model = joblib.load("model.pkl")
```

---

# serve.py

This is completely different.

It does **NOT** train anything.

Instead it loads the already trained model.

```
model.pkl
      │
      ▼
joblib.load()
      │
      ▼
Prediction API
```

It starts a Flask server.

The endpoints are

```
GET /health

POST /predict
```

---

# Health Endpoint

```
GET /health
```

returns

```json
{
  "status":"ok"
}
```

Why?

Container orchestrators like

- Kubernetes
- Docker Swarm
- ECS

need a quick way to know

"Is the application alive?"

They continuously call

```
GET /health
```

If the response is

```
200 OK
```

the container is healthy.

---

# Predict Endpoint

```
POST /predict
```

accepts JSON.

Example

```json
{
  "amount":250,
  "merchant":"Amazon",
  "country":"US"
}
```

The server loads

```
model.pkl
```

and returns

```
Fraud

or

Not Fraud
```

depending on prediction.

---

# Original Dockerfile

The original Dockerfile was

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir scikit-learn pandas numpy joblib flask

COPY train_model.py /app/train_model.py
COPY serve.py /app/serve.py

RUN python3 /app/train_model.py

EXPOSE 8080

CMD ["python3", "/app/serve.py"]
```

This works.

But it is not efficient.

---

# Problems with Single Stage Build

Everything ends up inside one image.

```
Image
│
├── train_model.py
├── serve.py
├── pandas
├── flask
├── numpy
├── scikit-learn
├── joblib
└── model.pkl
```

Production only needs

```
serve.py

model.pkl

runtime libraries
```

Everything else is unnecessary.

---

# Why is this bad?

Imagine

Training dependencies

```
3 GB
```

Runtime dependencies

```
300 MB
```

Single-stage image

```
3.3 GB
```

Multi-stage image

```
300 MB
```

Huge difference.

---

# What is a Docker Stage?

Every

```dockerfile
FROM
```

creates a completely new image layer.

Example

```dockerfile
FROM ubuntu
```

One stage.

Another

```dockerfile
FROM python
```

Second stage.

Stages are isolated.

Nothing is automatically shared.

---

# Builder Stage

Builder stage exists only during build.

```
Builder
│
├── install packages
├── compile code
├── build binary
├── train model
└── create artifacts
```

After build finishes

Docker throws away the builder image.

Only selected files are copied.

---

# Runtime Stage

Runtime stage is the final image.

```
Runtime
│
├── model.pkl
├── serve.py
└── runtime libraries
```

No training code.

No unnecessary packages.

---

# Understanding the Builder Stage

```dockerfile
FROM python:3.11-slim AS builder
```

The keyword

```
AS builder
```

creates a name.

Later we can reference it.

```
builder
```

acts like a label.

---

# Working Directory

```dockerfile
WORKDIR /app
```

Every command executes inside

```
/app
```

Equivalent to

```
cd /app
```

before every command.

---

# Copy Training Script

```dockerfile
COPY train_model.py .
```

Copies

```
host

↓

container
```

Result

```
/app/train_model.py
```

---

# Install Training Dependencies

```dockerfile
RUN pip install \
scikit-learn \
pandas \
numpy \
joblib
```

Notice

Flask is NOT installed here.

Because

Flask isn't needed during training.

---

# Train Model

```dockerfile
RUN python3 train_model.py
```

Runs

```
train_model.py
```

during image build.

The output is

```
model.pkl
```

Now builder contains

```
/app/model.pkl
```

---

# Runtime Stage

Starts from scratch.

```dockerfile
FROM python:3.11-slim
```

This is an entirely new image.

Nothing from builder exists here.

---

# Runtime Dependencies

```dockerfile
RUN pip install

flask

joblib

numpy

scikit-learn
```

Notice

```
pandas
```

is intentionally missing.

Why?

Because

```
serve.py
```

never imports pandas.

Installing unused packages only increases image size.

---

# Copy Serving Script

```dockerfile
COPY serve.py .
```

Only

```
serve.py
```

is needed.

Not

```
train_model.py
```

---

# Copy Model from Builder

This is the most important command.

```dockerfile
COPY --from=builder /app/model.pkl .
```

Meaning

```
Take

/app/model.pkl

from builder stage

↓

copy

↓

current runtime stage
```

Result

```
Runtime

│

├── serve.py

└── model.pkl
```

---

# Expose Port

```dockerfile
EXPOSE 8080
```

This documents

```
8080
```

as the application port.

It does **not** publish the port.

Publishing happens during

```
docker run
```

---

# CMD

```dockerfile
CMD ["python3","serve.py"]
```

Runs

```
python3 serve.py
```

when the container starts.

---

# Build Process

```
docker build
```

Step 1

```
Builder Stage
```

↓

Install packages

↓

Copy train_model.py

↓

Train model

↓

Generate model.pkl

↓

Stage finished

↓

Runtime Stage

↓

Install runtime packages

↓

Copy serve.py

↓

Copy model.pkl

↓

Create final image
```

---

# Why Doesn't Runtime Need train_model.py?

Because

Training is already finished.

The output

```
model.pkl
```

contains everything necessary.

It is exactly like

```
C Program

↓

gcc

↓

Executable

↓

Delete source code

↓

Run executable
```

The executable doesn't need

```
main.c
```

Similarly

```
serve.py

needs

model.pkl

not

train_model.py
```

---

# Docker Build Command

```
docker build -t ml-serve:v1 .
```

Meaning

```
build

↓

Dockerfile

↓

Current directory

↓

Tag image

↓

ml-serve:v1
```

---

# Docker Images

```
docker images
```

Shows

- Repository
- Tag
- Image ID
- Size

---

# Running the Container

```
docker run --rm -p 8090:8080 ml-serve:v1
```

Breakdown

```
docker run

Start container
```

```
--rm

Delete container after exit
```

```
-p

Port mapping
```

```
8090:8080
```

Means

```
Host

8090

↓

Container

8080
```

---

# Health Check

```
curl http://localhost:8090/health
```

Returns

```json
{
    "status":"ok"
}
```

Meaning

The server is working.

---

# Common Mistakes

## 1. Forgetting Builder Name

Wrong

```dockerfile
FROM python:3.11-slim
```

Correct

```dockerfile
FROM python:3.11-slim AS builder
```

---

## 2. Copying Wrong File

Wrong

```dockerfile
COPY train_model.py .
```

inside runtime.

Correct

Only copy

```
serve.py
```

---

## 3. Installing pandas in Runtime

Wrong

```dockerfile
RUN pip install pandas
```

The runtime doesn't need it.

---

## 4. Forgetting model.pkl

If

```dockerfile
COPY --from=builder
```

is missing

then

```
serve.py
```

fails because

```
model.pkl
```

doesn't exist.

---

## 5. Running Container During Lab Validation

If you already have

```
docker run -p 8090:8080
```

running,

the lab validator cannot start another container on the same port.

You'll get

```
Bind for 8090 failed

Port already allocated
```

Stop your container first:

```bash
docker stop <container-id>
```

---

# Interview Questions

### Why use multi-stage Docker builds?

To separate build-time dependencies from runtime dependencies, producing smaller, cleaner, and more secure images.

---

### Why train in the builder stage?

Training is only required during image creation. The final application only needs the trained model.

---

### Why is pandas excluded from the runtime image?

Because `serve.py` does not use it. Only packages required for inference should be included.

---

### What does `COPY --from=builder` do?

It copies files generated in one build stage into another stage.

---

### What is `model.pkl`?

A serialized Machine Learning model created using Joblib after training.

---

### What is the purpose of `EXPOSE 8080`?

It documents the application's listening port inside the container.

---

# Key Takeaways

- Multi-stage builds create smaller production images.
- Builder stages are temporary and discarded after the build.
- Runtime stages should contain only what the application needs.
- `model.pkl` is the artifact produced by training.
- `serve.py` performs inference by loading the trained model.
- `COPY --from=builder` transfers artifacts between stages.
- Reducing unnecessary dependencies improves security, image size, and deployment speed.
- Multi-stage builds are considered a Docker best practice for production applications, especially in Machine Learning workflows.

# Day 52 Notes: Understanding and Fixing a Docker Compose Stack (Jupyter + MLflow + SeaweedFS)

# Introduction

Modern Machine Learning (ML) development environments often consist of multiple services working together. Instead of installing each application individually, Docker Compose allows us to define and run all services from a single configuration file.

In this lab, the stack consists of three services:

1. **Jupyter Lab** – Interactive notebook environment.
2. **MLflow** – Machine learning experiment tracking server.
3. **SeaweedFS** – Object storage that provides an S3-compatible API and a web-based Filer UI.

The objective was to identify and fix configuration errors inside the `docker-compose.yml` file so that all services become accessible on their expected ports.

---

# What is Docker Compose?

Docker Compose is a tool for defining and running multiple Docker containers using a YAML configuration file.

Instead of running several long `docker run` commands, everything can be described inside one file.

Example:

```yaml
services:
  web:
    image: nginx
  db:
    image: mysql
```

Then simply run:

```bash
docker compose up -d
```

Docker Compose automatically:

- Pulls images (if necessary)
- Creates containers
- Creates networks
- Creates volumes
- Starts services

---

# Anatomy of docker-compose.yml

A Compose file generally contains:

```yaml
services:
volumes:
networks:
```

Example:

```yaml
services:
  jupyter:
    image: jupyter/base-notebook

volumes:
  data:
```

---

# Understanding the Three Services

## 1. Jupyter Lab

Image:

```
jupyter/base-notebook:python-3.11
```

Purpose:

- Write Python notebooks
- Run ML experiments
- Data analysis
- Visualization

Default Port:

```
8888
```

Normally when Jupyter starts, it generates a security token.

Example:

```
http://localhost:8888/?token=abc123...
```

Without the token, users cannot access the notebook.

For local development labs, authentication is often disabled.

---

## 2. MLflow

Image:

```
ghcr.io/mlflow/mlflow:v3.14.0
```

Purpose:

- Track experiments
- Log metrics
- Log parameters
- Save artifacts
- Compare model runs

Default Port:

```
5000
```

MLflow server command:

```bash
mlflow server
```

Example options:

```bash
--host 0.0.0.0
```

Accepts requests from outside the container.

```bash
--port 5000
```

Runs MLflow on port 5000.

```bash
--backend-store-uri
```

Database location.

```bash
--default-artifact-root
```

Artifact storage location.

---

## 3. SeaweedFS

SeaweedFS is a distributed file system.

It provides multiple services:

- File storage
- Object storage
- S3-compatible API
- Web UI (Filer)

In this lab it provides two important ports.

Container ports:

```
8333
```

S3 API

```
8888
```

Filer UI

---

# Docker Port Mapping

One of the most important Docker concepts is port mapping.

Syntax:

```
HOST_PORT:CONTAINER_PORT
```

Example:

```yaml
ports:
  - "5000:5000"
```

Meaning:

```
Browser
↓

localhost:5000

↓

Host Machine Port 5000

↓

Container Port 5000

↓

Application
```

---

# Common Mistake

Suppose the application runs inside the container on port 8888.

Incorrect:

```yaml
ports:
  - "8888:5000"
```

Correct:

```yaml
ports:
  - "8888:8888"
```

Always verify the application's internal listening port before exposing it.

---

# Understanding SeaweedFS Ports

Container exposes:

```
8333
```

S3 API

```
8888
```

Filer UI

Lab requirement:

Host

```
9000
```

↓

Container

```
8333
```

Host

```
9001
```

↓

Container

```
8888
```

Correct mapping:

```yaml
ports:
  - "9000:8333"
  - "9001:8888"
```

Incorrect mapping:

```yaml
ports:
  - "9001:8333"
  - "9000:8888"
```

Swapping these ports causes the browser UI and S3 API to appear on the wrong ports.

---

# Understanding Jupyter Authentication

By default Jupyter launches with:

- Token authentication
- Password support

Example URL:

```
http://localhost:8888/?token=...
```

This prevents unauthorized access.

For local development environments, authentication may be disabled.

Configuration:

```yaml
command:
  - start-notebook.sh
  - --ServerApp.token=
  - --ServerApp.password=
```

This sets:

```
token = ""
password = ""
```

No login prompt appears.

---

# Understanding command in Docker Compose

Compose normally starts the container using its default command.

Example:

```yaml
image: nginx
```

Internally executes:

```
nginx
```

We can override it.

Example:

```yaml
command:
  - python
  - app.py
```

In this lab:

```yaml
command:
  - start-notebook.sh
  - --ServerApp.token=
  - --ServerApp.password=
```

Compose replaces the default startup command.

---

# Volumes

Volumes store persistent data.

Example:

```yaml
volumes:
  - mlflow-data:/mlflow
```

Anything inside:

```
/mlflow
```

is preserved even if the container is removed.

Similarly:

```yaml
volumes:
  - seaweedfs-data:/data
```

stores SeaweedFS data.

---

# Container Names

Instead of random names like:

```
crazy_einstein
```

Compose specifies:

```yaml
container_name:
```

Example:

```yaml
container_name: ml-jupyter
```

Useful for:

```bash
docker logs ml-jupyter
```

instead of finding container IDs.

---

# Bringing Up the Stack

Start services:

```bash
docker compose -f docker-compose.yml up -d
```

Options:

```
up
```

Creates and starts containers.

```
-d
```

Detached mode.

Runs in the background.

---

# Viewing Running Containers

Command:

```bash
docker compose ps
```

Shows:

- Container name
- Status
- Ports
- Health

Example:

```
NAME            STATUS

ml-jupyter      Up
ml-mlflow       Up
ml-seaweedfs    Up
```

---

# Verifying Services with curl

Instead of opening a browser, use:

```bash
curl -I http://localhost:8888
```

The `-I` option fetches only HTTP headers.

Expected:

```
HTTP/1.1 200 OK
```

or

```
HTTP/1.1 302 Found
```

A `302` response indicates the application is redirecting the browser, which is still considered reachable.

---

# Why 200 and 302 Are Acceptable

**200 OK**

The requested page is returned successfully.

**302 Found**

The application redirects to another page (for example, `/tree`, `/lab`, or `/login`).

Both indicate that the service is running and responding.

---

# Debugging Steps

When a service is not accessible:

1. Check container status.

```bash
docker compose ps
```

2. Inspect logs.

```bash
docker logs <container-name>
```

Example:

```bash
docker logs ml-jupyter
```

3. Verify exposed ports.

```bash
docker port ml-seaweedfs
```

4. Test endpoints.

```bash
curl http://localhost:5000
```

---

# Best Practices

- Keep one service per container.
- Use named volumes for persistent data.
- Always expose the correct container port.
- Use meaningful container names.
- Verify services using `curl` after deployment.
- Check logs when troubleshooting.
- Store configuration in version control.
- Avoid disabling authentication in production environments.

---

# Key Commands Used

Start containers:

```bash
docker compose up -d
```

Stop containers:

```bash
docker compose down
```

Restart containers:

```bash
docker compose restart
```

View running containers:

```bash
docker compose ps
```

View logs:

```bash
docker logs ml-jupyter
```

Verify Jupyter:

```bash
curl -I http://localhost:8888
```

Verify MLflow:

```bash
curl -I http://localhost:5000
```

Verify SeaweedFS Filer:

```bash
curl -I http://localhost:9001
```

---

# Summary

This lab demonstrated how a few small configuration errors in a Docker Compose file can prevent services from being accessible. The main fixes included correcting the SeaweedFS port mappings and configuring Jupyter to start without token-based authentication for local development. It also reinforced core Docker Compose concepts such as service definitions, port mappings, volume persistence, container naming, command overrides, and verification using `docker compose ps` and `curl`. Understanding these fundamentals is essential for deploying, troubleshooting, and maintaining multi-container applications.

# Day 53 Notes: Understanding CPU vs GPU PyTorch Docker Images

## Introduction

When creating Docker images for Machine Learning workloads, it is important to install the correct version of PyTorch based on the hardware where the container will run.

PyTorch provides different wheel packages for:

- CPU-only systems
- NVIDIA GPU (CUDA) systems

If the wrong package is installed, the image may fail to build or the application may fail during runtime.

---

# What is a Dockerfile?

A Dockerfile is a text file containing instructions that Docker follows to build an image.

Example:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install torch

CMD ["python3"]
```

Each instruction creates a new image layer.

---

# Understanding the Instructions

## FROM

```dockerfile
FROM python:3.11-slim
```

This specifies the base image.

Here we start from a lightweight Python 3.11 image.

---

## WORKDIR

```dockerfile
WORKDIR /app
```

Sets the working directory inside the container.

Equivalent to:

```bash
cd /app
```

Every following command runs from this directory.

---

## RUN

```dockerfile
RUN pip install torch
```

`RUN` executes commands while building the image.

The installed software becomes part of the final Docker image.

---

## CMD

```dockerfile
CMD ["python3"]
```

`CMD` specifies the default command executed when the container starts.

Unlike `RUN`, `CMD` is **not executed during image build**.

It runs only when someone executes:

```bash
docker run image-name
```

---

# Why Did the Original Dockerfile Fail?

The original Dockerfile contained:

```dockerfile
RUN pip install \
    --index-url https://download.pytorch.org/whl/gpu \
    torch
```

The lab machine had **no GPU**.

PyTorch provides different package repositories depending on the hardware.

Trying to use the GPU package repository in a CPU-only environment causes installation problems.

---

# CPU Wheel vs GPU Wheel

CPU installation:

```text
https://download.pytorch.org/whl/cpu
```

GPU installation:

```text
https://download.pytorch.org/whl/cu124
```

or another CUDA version.

Choose the wheel based on the machine where the container will run.

---

# Why Use the CPU Wheel?

CPU wheels

- work on any machine
- require no NVIDIA drivers
- require no CUDA libraries

GPU wheels

- require NVIDIA drivers
- require CUDA runtime
- are larger
- only work correctly on supported GPU systems

---

# What is torch.cuda?

PyTorch includes the module:

```python
torch.cuda
```

This module provides CUDA-related functionality.

Example:

```python
import torch

print(torch.cuda.is_available())
```

Possible output:

```text
True
```

or

```text
False
```

---

# What Does torch.cuda.is_available() Do?

It checks whether PyTorch can access a CUDA-enabled GPU.

Example:

```python
import torch

if torch.cuda.is_available():
    print("GPU Available")
else:
    print("CPU Only")
```

Output on the lab machine:

```text
CPU Only
```

---

# Why Did the Container Crash?

The original Dockerfile contained:

```python
assert torch.cuda.is_available(), "CUDA required"
```

`assert` checks whether a condition is True.

If the condition is False:

```python
AssertionError: CUDA required
```

The Python program exits immediately.

Since the lab machine has no GPU,

```python
torch.cuda.is_available()
```

returns

```python
False
```

Therefore the container exits with an error.

---

# Better Approach

Instead of requiring CUDA, simply display its status.

Example:

```python
import torch

print(torch.__version__)
print(torch.cuda.is_available())
```

Output:

```text
2.5.0+cpu
False
```

---

# Understanding torch.__version__

```python
import torch

print(torch.__version__)
```

Example output:

```text
2.5.0+cpu
```

The suffix

```text
+cpu
```

indicates that the CPU version of PyTorch is installed.

---

# Understanding pip --index-url

Normally pip downloads packages from:

```text
https://pypi.org/simple
```

You can override this using

```bash
pip install \
--index-url URL
```

For PyTorch:

```bash
pip install torch \
--index-url https://download.pytorch.org/whl/cpu
```

This tells pip to download PyTorch wheels from the official CPU package repository.

---

# Building the Docker Image

Command:

```bash
docker build -t dl-trainer:v1 .
```

Explanation:

- `docker build` builds the image
- `-t` assigns a tag
- `dl-trainer:v1` is the image name and version
- `.` means use the current directory

---

# Listing Images

```bash
docker images
```

Example:

```text
REPOSITORY    TAG
dl-trainer    v1
```

This confirms the image was built successfully.

---

# Running the Container

```bash
docker run --rm dl-trainer:v1
```

Explanation:

- creates a new container
- executes the default CMD
- removes the container after it exits because of `--rm`

---

# Expected Output

Example:

```text
2.5.0+cpu cuda? False
```

This shows:

- PyTorch is installed successfully.
- CPU version is being used.
- CUDA is unavailable, which is expected.

---

# Best Practices

- Always install the correct package for the target hardware.
- Avoid hardcoding GPU requirements unless they are mandatory.
- Keep Docker images lightweight.
- Verify software installation during container startup.
- Use CPU wheels for CPU-only servers and CI/CD environments.
- Use GPU wheels only on systems with NVIDIA GPUs and compatible CUDA drivers.

---

# Summary

In this lab, we learned how to:

- Understand the purpose of a Dockerfile.
- Differentiate between `RUN` and `CMD`.
- Install the correct PyTorch package for CPU-only environments.
- Use the official CPU wheel repository.
- Check CUDA availability using `torch.cuda.is_available()`.
- Avoid runtime failures caused by unnecessary CUDA assertions.
- Build, verify, and run a Docker image successfully.

This is a common task in DevOps and MLOps workflows, where container images must be tailored to the hardware available in production environments.

# Docker Private Registry - Complete Notes (Day 54)

# Introduction

In modern software development, applications are usually packaged as **Docker Images**. These images contain everything required to run an application:

- Application source code
- Runtime
- Libraries
- Dependencies
- Configuration

Instead of rebuilding applications on every server, we simply build an image once and distribute it wherever needed.

But how do other servers obtain this image?

The answer is through a **Docker Registry**.

---

# What is a Docker Registry?

A Docker Registry is a storage service that stores Docker images.

Think of it as GitHub, but instead of storing source code, it stores Docker images.

Examples:

- Docker Hub
- Amazon ECR
- Google Artifact Registry
- Azure Container Registry
- Harbor
- Private Docker Registry (registry:2)

Whenever you execute

```bash
docker pull nginx
```

Docker contacts Docker Hub and downloads the image.

Similarly, if you execute

```bash
docker push my-image
```

Docker uploads the image to a registry.

---

# Public vs Private Registry

## Public Registry

Accessible by anyone.

Example:

Docker Hub

```
docker pull nginx
```

Anyone can pull it.

---

## Private Registry

Only authorized users or systems can access it.

Organizations prefer private registries because

- Images contain company code
- Images may contain ML models
- Internal services should not be public
- Better security
- Better performance

---

# Docker Registry Architecture

```
                Developer

                    |
                    |
             docker push
                    |
                    V

        ------------------------
        |   Docker Registry    |
        | Stores Docker Images |
        ------------------------

                    ^
                    |
             docker pull
                    |
                    |

             Kubernetes Cluster
```

The registry acts as a centralized image repository.

---

# The Registry Used in This Lab

The registry already exists.

Container:

```
registry:2
```

Running as

```
local-registry
```

Port Mapping

```
Host
5555

↓

Container
5000
```

Meaning

```
localhost:5555
```

actually forwards requests to

```
registry container :5000
```

---

# Understanding Port Mapping

Docker command generally looks like

```
-p HOST_PORT:CONTAINER_PORT
```

Example

```
-p 5555:5000
```

Means

```
localhost:5555

↓

registry container

↓

5000
```

So when we visit

```
http://localhost:5555
```

Docker redirects traffic into the registry container.

---

# Project Structure

```
ml-registry/

│
├── Dockerfile
├── train.py
└── push.sh
```

---

# train.py

This file trains a small machine learning model.

Example workflow

```
Load Dataset

↓

Train RandomForest

↓

Save model.pkl
```

The generated model is stored as

```
/app/model.pkl
```

---

# Dockerfile

The Dockerfile creates the image.

Typical flow

```
FROM python:3.11-slim

↓

Install sklearn

↓

Install numpy

↓

Install joblib

↓

Copy train.py

↓

Run train.py

↓

model.pkl created

↓

Image completed
```

Notice

The ML model is baked into the image during build.

That means every container created from this image already contains the trained model.

---

# push.sh

Initially

```
docker build
```

was implemented.

Publishing to registry was missing.

---

# Docker Build

Command

```bash
docker build -t fraud-detector:v1 .
```

Let's understand every part.

---

## docker

Docker CLI

---

## build

Create image

---

## -t

Assign tag

---

## fraud-detector

Repository name

---

## v1

Version tag

---

## .

Current directory

Docker searches here for Dockerfile.

---

Image Naming Convention

```
repository:tag
```

Example

```
ubuntu:22.04

nginx:latest

fraud-detector:v1
```

---

# What Happens During Build?

```
Dockerfile

↓

Download Base Image

↓

Install Packages

↓

Copy Files

↓

Run train.py

↓

Generate model.pkl

↓

Create Image
```

The image now exists only on your local Docker engine.

Nothing has been uploaded anywhere.

---

# Local Docker Images

View them

```bash
docker images
```

Example

```
REPOSITORY

fraud-detector

TAG

v1
```

Still not uploaded.

---

# Why docker tag?

Suppose image exists

```
fraud-detector:v1
```

Registry requires

```
localhost:5555/fraud-detector:v1
```

These are different names.

Docker identifies registry destination from image name.

---

Without registry

```
fraud-detector:v1
```

With registry

```
localhost:5555/fraud-detector:v1
```

Docker now knows where to push.

---

# docker tag

Syntax

```bash
docker tag SOURCE TARGET
```

Example

```bash
docker tag fraud-detector:v1 localhost:5555/fraud-detector:v1
```

Nothing gets uploaded.

Docker only creates another reference.

---

Visualization

Before

```
fraud-detector:v1
```

After

```
fraud-detector:v1

localhost:5555/fraud-detector:v1
```

Both point to exactly the same image ID.

No duplicate image is created.

Only metadata changes.

---

# Why docker push?

Tagging only creates another image name.

Registry still doesn't know anything.

Need

```bash
docker push localhost:5555/fraud-detector:v1
```

Docker now

```
Compresses Layers

↓

Connects Registry

↓

Uploads Missing Layers

↓

Uploads Manifest

↓

Registry Stores Image
```

Only after this step does the image exist in registry.

---

# Complete Flow

```
Dockerfile

↓

docker build

↓

fraud-detector:v1

↓

docker tag

↓

localhost:5555/fraud-detector:v1

↓

docker push

↓

Registry Stores Image
```

---

# Final push.sh

```bash
#!/bin/bash
set -euo pipefail

cd "$(dirname "$0")"

IMAGE="fraud-detector:v1"

docker build -t "$IMAGE" .

docker tag "$IMAGE" localhost:5555/fraud-detector:v1

docker push localhost:5555/fraud-detector:v1
```

---

# Registry API

Docker Registry exposes HTTP APIs.

---

## List repositories

```
GET

/v2/_catalog
```

Command

```bash
curl http://localhost:5555/v2/_catalog
```

Response

```json
{
  "repositories":[
      "fraud-detector"
  ]
}
```

Meaning

Registry currently stores

```
fraud-detector
```

---

# List Tags

API

```
GET

/v2/<repository>/tags/list
```

Example

```bash
curl http://localhost:5555/v2/fraud-detector/tags/list
```

Response

```json
{
    "name":"fraud-detector",
    "tags":[
        "v1"
    ]
}
```

Meaning

Repository

```
fraud-detector
```

contains version

```
v1
```

---

# Why Version Tags?

Instead of rebuilding over same name

We create

```
v1

v2

v3

latest
```

Example

```
fraud-detector:v1

fraud-detector:v2

fraud-detector:latest
```

Kubernetes can decide which version to deploy.

---

# Image Lifecycle

```
Developer writes code

↓

Docker Build

↓

Docker Image

↓

Docker Tag

↓

Docker Push

↓

Private Registry

↓

Kubernetes Pull

↓

Containers Start
```

---

# Common Interview Questions

## Difference between build and push

Build creates image locally.

Push uploads image to registry.

---

## Difference between tag and push

Tag changes image name.

Push uploads image.

---

## Can docker tag upload image?

No.

It only creates another local reference.

---

## Why registry prefix is needed?

Docker identifies destination registry using image name.

Example

```
localhost:5555/image:v1
```

---

## Does docker push upload everything every time?

No.

Docker uploads only missing layers.

Already existing layers are skipped.

---

## Why use Private Registry?

- Faster deployment
- Secure images
- Company internal software
- ML models
- Version management
- Kubernetes integration

---

# Commands Used

Build image

```bash
docker build -t fraud-detector:v1 .
```

Tag image

```bash
docker tag fraud-detector:v1 localhost:5555/fraud-detector:v1
```

Push image

```bash
docker push localhost:5555/fraud-detector:v1
```

List images

```bash
docker images
```

List repositories

```bash
curl http://localhost:5555/v2/_catalog
```

List tags

```bash
curl http://localhost:5555/v2/fraud-detector/tags/list
```

---

# Summary

- A Docker image is built locally using `docker build`.
- A Docker Registry stores Docker images for distribution.
- `docker tag` creates a registry-qualified name for an image.
- `docker push` uploads the image to the registry.
- The image becomes available to other systems only after it is pushed.
- The Docker Registry HTTP API can verify stored repositories and tags.
- In this lab, the image `fraud-detector:v1` was tagged as `localhost:5555/fraud-detector:v1` and pushed to the private registry running on port **5555**.
- Verification was done using the registry endpoints `/v2/_catalog` and `/v2/fraud-detector/tags/list`.

# Day 55: Fix a Broken Dockerfile HEALTHCHECK and EXPOSE

## Task

The xFusionCorp Industries ML platform team deploys Flask-based inference APIs as Docker images with Docker-native `HEALTHCHECK` instructions.

The existing Dockerfile had two issues:

1. The `HEALTHCHECK` target was incorrect.
2. The Docker image did not expose the application port.

The Flask API serves:

- `GET /health` → returns `{"status": "ok"}` with HTTP 200
- `POST /predict` → rule-based fraud prediction

The application runs on port **8085**.

---

## Problem

The original Dockerfile contained:

```dockerfile
HEALTHCHECK --interval=5s --timeout=3s --start-period=3s --retries=3 \
  CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8085/healthz')" || exit 1
```

The health check failed because:

- The API endpoint is `/health`
- The configured endpoint `/healthz` does not exist

The Docker image also lacked:

```dockerfile
EXPOSE 8085
```

Without `EXPOSE`, Docker image metadata does not declare the intended service port.

---

## Fixed Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN pip install --no-cache-dir flask

COPY app.py /app/app.py

EXPOSE 8085

HEALTHCHECK --interval=5s --timeout=3s --start-period=3s --retries=3 \
  CMD python3 -c "import urllib.request; urllib.request.urlopen('http://localhost:8085/health')" || exit 1

CMD ["python3", "/app/app.py"]
```

---

## Build Image

```bash
cd /root/code/ml-health

docker build -t ml-health:v1 .
```

---

## Run Container

Remove the old container:

```bash
docker rm -f ml-health-api
```

Start the updated container:

```bash
docker run -d \
  --name ml-health-api \
  -p 8085:8085 \
  ml-health:v1
```

---

## Verification

### Check Exposed Ports

Command:

```bash
docker inspect --format '{{.Config.ExposedPorts}}' ml-health:v1
```

Expected output:

```text
map[8085/tcp:{}]
```

---

### Check Container Health

Wait around 15 seconds, then run:

```bash
docker inspect --format='{{.State.Health.Status}}' ml-health-api
```

Expected output:

```text
healthy
```

---

### Test API Endpoint

Command:

```bash
curl http://localhost:8085/health
```

Expected response:

```json
{"status":"ok"}
```

HTTP status:

```text
200 OK
```

---

## Key Concepts Learned

### Docker HEALTHCHECK

`HEALTHCHECK` allows Docker to monitor whether a containerized application is working correctly.

Example:

```dockerfile
HEALTHCHECK --interval=5s --timeout=3s --retries=3 CMD <health-check-command>
```

- `--interval` → how often Docker runs the health check
- `--timeout` → maximum time allowed for each check
- `--retries` → consecutive failures required before marking unhealthy

Docker changes the container health state:

- `starting`
- `healthy`
- `unhealthy`

---

### Docker EXPOSE

`EXPOSE` documents the port that the containerized application listens on.

Example:

```dockerfile
EXPOSE 8085
```

It adds image metadata:

```text
map[8085/tcp:{}]
```

Important:

`EXPOSE` does **not** publish the port.

Port publishing is done using:

```bash
docker run -p 8085:8085 image-name
```

---

## Final Status

✅ Fixed HEALTHCHECK endpoint  
✅ Added EXPOSE 8085  
✅ Rebuilt Docker image  
✅ Container reports healthy  
✅ Flask health endpoint returns HTTP 200


# Docker CI Pipeline with Git SHA Tagging - Complete Notes

# Day 56 Notes

---

# Introduction

In modern DevOps practices, applications are rarely built manually.

Instead, every code change goes through an automated **Continuous Integration (CI)** pipeline.

A CI pipeline performs tasks such as:

- Running tests
- Building artifacts
- Packaging applications
- Creating Docker images
- Tagging images
- Uploading images to registries
- Preparing software for deployment

In this lab we worked with a **Shell-Based Docker CI Pipeline**.

Unlike Jenkins, GitHub Actions, GitLab CI or Azure Pipelines, this pipeline is written completely in Bash.

Although simple, it demonstrates the exact sequence followed by enterprise CI systems.

---

# Pipeline Flow

```
Developer
      │
      ▼
 Git Repository
      │
      ▼
Run Tests
      │
      ▼
Build Docker Image
      │
      ▼
Tag Image
      │
      ▼
Push to Registry
      │
      ▼
Deployment
```

---

# Directory Structure

```
ci/
│
├── app/
│   ├── app.py
│   ├── Dockerfile
│   ├── test_app.py
│   └── .git/
│
└── build.sh
```

Each file has a purpose.

---

# app.py

Contains the Flask application.

Endpoints

```
/health

/predict
```

Runs on

```
8086
```

This file was already correct.

---

# test_app.py

Contains automated tests.

There were three tests.

```
Health Endpoint

Fraud Detection Endpoint

Pass-through Endpoint
```

Pytest executes these tests.

No modification required.

---

# Dockerfile

Responsible for creating Docker image.

Typical flow

```
FROM python:3.11-slim

Install Flask

Copy app.py

Expose 8086

Run Flask application
```

Already correct.

---

# Local Git Repository

Inside

```
app/.git
```

Git repository already existed.

Current commit SHA was used for Docker image tagging.

Example

```
git -C app rev-parse --short HEAD
```

Output

```
d39b0ab
```

That value becomes Docker tag.

---

# Docker Registry

Instead of Docker Hub we used a local registry.

Container already running.

```
registry:2
```

Container name

```
local-registry
```

Port

```
5555
```

Registry URL

```
localhost:5555
```

---

# Understanding build.sh

Pipeline consisted of four stages.

```
Stage 1

Testing

↓

Stage 2

Building

↓

Stage 3

Tagging

↓

Stage 4

Pushing
```

---

# Script Header

```bash
#!/bin/bash
```

Specifies Bash interpreter.

---

# Error Handling

```bash
set -euo pipefail
```

This is considered best practice.

Let's understand every option.

---

## -e

```
Exit immediately if any command fails.
```

Without it

```
Command fails

↓

Script continues

↓

Unexpected behaviour
```

With it

```
Command fails

↓

Script stops immediately
```

---

## -u

Treat undefined variables as errors.

Example

```bash
echo $NAME
```

If NAME does not exist

Without

```
Nothing printed.
```

With

```
Undefined variable error
```

Very useful for catching typos.

---

## pipefail

Normally

```
A | B | C
```

Only last command determines success.

With pipefail

Entire pipeline fails if any command fails.

---

# Change Directory

```bash
cd "$(dirname "$0")"
```

Very important.

Suppose script exists

```
/root/code/ci/build.sh
```

Running

```
./build.sh
```

works.

But

```
/root/code/ci/build.sh
```

from another directory may fail.

This line ensures script always executes from its own directory.

---

# Variables

```bash
IMAGE="ml-ci-app"
```

Stores Docker image name.

Later reused as

```
ml-ci-app:latest
```

---

```bash
REGISTRY="localhost:5555"
```

Stores Docker registry.

Using variables makes scripts reusable.

Instead of changing multiple lines

Only one variable changes.

---

# Stage 1

Testing

Original

```bash
python3 -m pytest app/tests/
```

Problem

Directory

```
tests/
```

did not exist.

Actual file

```
test_app.py
```

Correct

```bash
python3 -m pytest app/test_app.py
```

---

# Why Testing First?

CI pipelines always execute tests before building.

Reason

No point creating Docker images if application is already broken.

Correct order

```
Run Tests

↓

Pass

↓

Build Image
```

Incorrect order

```
Build Image

↓

Run Tests

↓

Fail

↓

Wasted build time
```

---

# Stage 2

Building Docker Image

Command

```bash
docker build -t ml-ci-app:latest app/
```

Let's understand every part.

```
docker build
```

Starts Docker build.

```
-t
```

Assigns image tag.

```
ml-ci-app:latest
```

Image name.

```
app/
```

Build context.

Docker reads Dockerfile from here.

---

# Docker Build Process

Docker performs

```
Read Dockerfile

↓

Download Base Image

↓

Install Packages

↓

Copy Files

↓

Create Layers

↓

Produce Final Image
```

---

# Verify Image

```bash
docker images
```

Output

```
REPOSITORY

ml-ci-app

TAG

latest
```

---

# Stage 3

Tagging

Original

```bash
SHA=$(git -C app rev-parse --short HEAD)
```

This retrieves

```
Current Commit

↓

Short SHA

↓

Store in SHA
```

Example

```
2bc41ef
```

---

# What Does -C Mean?

```bash
git -C app
```

Equivalent to

```
cd app

git command

cd back
```

Useful inside scripts.

---

# rev-parse

Returns Git object information.

```
HEAD
```

means

Latest commit.

---

# --short

Without

```
2bc41ef6a9a62b839a541c...
```

With

```
2bc41ef
```

Short SHA preferred for Docker tags.

---

# The Bug

Script created

```bash
SHA
```

Later used

```bash
$GIT_SHA
```

Problem

Variable never existed.

Error

```
unbound variable
```

Because

```
set -u
```

was enabled.

---

# Correct Statement

```bash
TAGGED="$REGISTRY/$IMAGE:$SHA"
```

Result

```
localhost:5555/ml-ci-app:2bc41ef
```

---

# Docker Tag

Command

```bash
docker tag SOURCE DESTINATION
```

Example

```bash
docker tag ml-ci-app:latest localhost:5555/ml-ci-app:2bc41ef
```

No new image created.

Only another reference.

Think of tagging as

```
Same Book

Different Label
```

---

# Stage 4

Push

Command

```bash
docker push "$TAGGED"
```

Uploads image into registry.

Flow

```
Local Image

↓

Registry

↓

Available for Deployment
```

---

# Registry API

Docker registry exposes REST endpoints.

---

## List Repositories

```
GET

/v2/_catalog
```

Command

```bash
curl http://localhost:5555/v2/_catalog
```

Output

```json
{
  "repositories":[
      "ml-ci-app"
  ]
}
```

---

## List Tags

```
GET

/v2/ml-ci-app/tags/list
```

Command

```bash
curl http://localhost:5555/v2/ml-ci-app/tags/list
```

Output

```json
{
"name":"ml-ci-app",
"tags":[
"2bc41ef"
]
}
```

---

# Why Git SHA Tagging?

Using latest only

```
latest

latest

latest
```

Impossible to know which commit created image.

Using Git SHA

```
2bc41ef

ab910ef

92bd001
```

Every image becomes traceable.

Benefits

- Rollback
- Version Tracking
- Debugging
- Auditing
- Deployment History

---

# Complete Pipeline

```
Git Commit

↓

Pytest

↓

Docker Build

↓

Git SHA

↓

Docker Tag

↓

Docker Push

↓

Registry

↓

Deployment
```

---

# Final Correct Script

```bash
#!/bin/bash

set -euo pipefail

cd "$(dirname "$0")"

IMAGE="ml-ci-app"
REGISTRY="localhost:5555"

echo "[ci] stage 1/4 — running tests"
python3 -m pytest app/test_app.py

echo "[ci] stage 2/4 — building image"
docker build -t "$IMAGE:latest" app/

echo "[ci] stage 3/4 — tagging"
SHA=$(git -C app rev-parse --short HEAD)
TAGGED="$REGISTRY/$IMAGE:$SHA"
docker tag "$IMAGE:latest" "$TAGGED"

echo "[ci] stage 4/4 — pushing"
docker push "$TAGGED"

echo "[ci] complete: $TAGGED"
```

---

# Verification Commands

Verify local image

```bash
docker images ml-ci-app
```

Verify registry catalog

```bash
curl http://localhost:5555/v2/_catalog
```

Verify image tags

```bash
curl http://localhost:5555/v2/ml-ci-app/tags/list
```

Verify Git SHA

```bash
git -C app rev-parse --short HEAD
```

Both SHA values must match.

---

# Common Interview Questions

## Why use `set -euo pipefail`?

To make shell scripts fail fast and catch errors early.

---

## Why tag Docker images with Git SHA?

To uniquely identify the source code version that produced the image.

---

## Difference between `docker build` and `docker tag`?

- `docker build` creates a new image.
- `docker tag` creates another reference (name/tag) to an existing image.

---

## Why run tests before building?

To avoid wasting time and resources building images for broken code.

---

## What is Docker Registry?

A service that stores Docker images so they can be pulled by other systems or deployment platforms.

---

## What does `git rev-parse --short HEAD` do?

It returns the short commit hash of the latest commit (`HEAD`) in the Git repository.

---

# Key Takeaways

- CI pipelines automate testing, building, tagging, and publishing applications.
- `set -euo pipefail` improves shell script reliability.
- Always verify file paths used in scripts.
- Docker images are typically tagged with Git SHAs for traceability.
- A Docker registry stores versioned container images for deployment.
- REST API endpoints like `/v2/_catalog` and `/v2/<image>/tags/list` help verify pushed images.
- Shell variables should be used consistently to avoid runtime errors.
- Running tests before building is a standard CI/CD best practice.

# Day 57 Notes: Serving a Machine Learning Model with Flask

# Introduction

Machine Learning models are useful only when they can be used by other applications. Training a model is just one part of the ML lifecycle. The next important step is **serving** the trained model so that applications, websites, mobile apps, or other services can make predictions.

In this lab, we used **Flask**, a lightweight Python web framework, to expose a trained fraud detection model through an HTTP API.

The API exposes two endpoints:

- `/health` – Used to verify that the service is running.
- `/predict` – Used to send transaction data and receive a fraud prediction.

---

# What is Model Serving?

Model Serving is the process of making a trained machine learning model available through an interface such as:

- REST API
- gRPC
- Batch jobs
- Message queues

Instead of running Python scripts manually, clients send requests over HTTP.

Example:

```
Mobile App
      │
      ▼
 REST API (Flask)
      │
      ▼
 Machine Learning Model
      │
      ▼
 Prediction
```

---

# Why Use Flask?

Flask is a lightweight web framework for Python.

Advantages:

- Very small
- Easy to learn
- Easy to build REST APIs
- Excellent for ML inference services
- Works well with libraries like NumPy, Pandas and Scikit-Learn

Install Flask

```bash
pip install flask
```

---

# Project Structure

```
serving/
│
├── app.py
├── model.pkl
└── train.csv
```

### app.py

Contains the Flask application.

### model.pkl

Serialized trained machine learning model.

### train.csv

Dataset that was used for training.

---

# Understanding model.pkl

A machine learning model is trained once.

Training may take:

- Minutes
- Hours
- Days

Instead of retraining every time the server starts, we save the trained model.

Python uses **Joblib** to serialize the model.

```
Training
     │
     ▼
RandomForest Model
     │
     ▼
joblib.dump()
     │
     ▼
model.pkl
```

Later

```
model.pkl
     │
     ▼
joblib.load()
     │
     ▼
Ready-to-use model
```

---

# Loading the Model

```python
import joblib

MODEL = joblib.load("model.pkl")
```

What happens?

1. Reads model.pkl
2. Recreates the RandomForest object
3. Loads all learned parameters
4. Model becomes ready for prediction

No training happens here.

---

# Creating the Flask App

```python
from flask import Flask

app = Flask(__name__)
```

Every Flask application starts by creating an application object.

```
Browser
   │
   ▼
Flask App
   │
   ▼
Routes
```

---

# What is a Route?

A route connects a URL with a Python function.

Example

```python
@app.route("/health")
def health():
    ...
```

When a request comes to

```
GET /health
```

Flask executes

```python
health()
```

---

# HTTP Methods

There are several HTTP methods.

GET

Used for retrieving information.

Example

```
GET /health
```

POST

Used for sending data.

Example

```
POST /predict
```

PUT

Updates existing data.

DELETE

Deletes data.

---

# Health Endpoint

```python
@app.route("/health")
def health():
    return jsonify({"status":"ok"}),200
```

Purpose

Health endpoints are used by

- Kubernetes
- Docker
- Load Balancers
- Monitoring tools

to verify that the service is alive.

Response

```json
{
    "status":"ok"
}
```

---

# Understanding POST /predict

A prediction endpoint receives user input.

Example

```json
{
    "amount":3200,
    "hour":23,
    "num_tx_past_day":5
}
```

The endpoint performs

Input

↓

Validation

↓

Feature Vector

↓

Model Prediction

↓

JSON Response

---

# Reading JSON Data

```python
data = request.get_json()
```

Suppose client sends

```json
{
    "amount":3200,
    "hour":23,
    "num_tx_past_day":5
}
```

Then

```python
data
```

becomes

```python
{
    "amount":3200,
    "hour":23,
    "num_tx_past_day":5
}
```

---

# Accessing Values

```python
amount = data["amount"]
hour = data["hour"]
transactions = data["num_tx_past_day"]
```

Now

```
amount = 3200

hour = 23

transactions = 5
```

---

# Creating Feature Array

Scikit-Learn expects

```
(number_of_samples, number_of_features)
```

Since only one transaction is predicted

```
Samples = 1
```

Features

- amount
- hour
- num_tx_past_day

So

```python
features = np.array([
    [
        amount,
        hour,
        transactions
    ]
])
```

Shape

```
(1,3)
```

Visual representation

```
┌──────────────────────┐
│3200 23 5             │
└──────────────────────┘
```

---

# Why NumPy?

Scikit-Learn works internally with NumPy arrays.

Advantages

- Fast
- Memory efficient
- Vectorized operations
- Standard format for ML libraries

---

# Making Prediction

```python
prediction = MODEL.predict(features)
```

Returns

```
array([1])
```

or

```
array([0])
```

Since JSON cannot serialize NumPy integers properly

Convert

```python
prediction = int(prediction[0])
```

Now

```
1
```

or

```
0
```

---

# Returning JSON

```python
return jsonify({
    "is_fraud": prediction
}),200
```

Flask automatically sets

```
Content-Type

application/json
```

---

# Complete Predict Function

```python
@app.route("/predict", methods=["POST"])
def predict():
    data = request.get_json()

    features = np.array([[
        data["amount"],
        data["hour"],
        data["num_tx_past_day"]
    ]])

    prediction = int(MODEL.predict(features)[0])

    return jsonify({
        "is_fraud": prediction
    }),200
```

---

# Running Flask

```python
app.run(
    host="0.0.0.0",
    port=8085
)
```

Meaning

Host

```
0.0.0.0
```

Accept requests from every network interface.

Port

```
8085
```

Application listens on TCP port 8085.

---

# Why Not localhost?

```
127.0.0.1
```

means

Only local machine.

```
0.0.0.0
```

means

Accept connections from

- Localhost
- Docker
- Kubernetes
- Remote machines
- Port forwarding

---

# Testing with curl

Health

```bash
curl http://localhost:8085/health
```

Response

```json
{
    "status":"ok"
}
```

Prediction

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

Low-risk transaction

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":25.5,"hour":10,"num_tx_past_day":1}'
```

---

# Request Flow

```
Client

     │

POST /predict

     │

Flask

     │

request.get_json()

     │

NumPy Array

     │

MODEL.predict()

     │

Prediction

     │

JSON Response

     │

Client
```

---

# Common Mistakes

## Wrong Port

Using

```python
5000
```

instead of

```python
8085
```

Lab fails because the grader checks port 8085.

---

## Forgetting POST Method

Wrong

```python
@app.route("/predict")
```

Correct

```python
@app.route("/predict", methods=["POST"])
```

---

## Using List Instead of 2D Array

Wrong

```python
MODEL.predict([1,2,3])
```

Correct

```python
MODEL.predict([[1,2,3]])
```

---

## Forgetting int()

Wrong

```python
return prediction
```

Prediction is

```
numpy.int64
```

Correct

```python
int(prediction[0])
```

---

## Forgetting jsonify()

Wrong

```python
return {
    "is_fraud":1
}
```

Preferred

```python
return jsonify({
    "is_fraud":1
})
```

---

# Real-World Applications

The same architecture is used for:

- Fraud Detection
- Loan Approval
- Credit Scoring
- Spam Detection
- Recommendation Systems
- Face Recognition
- Disease Prediction
- Stock Market Prediction
- Customer Churn Prediction
- Insurance Risk Analysis

---

# Key Takeaways

- A trained ML model is loaded from `model.pkl` using Joblib.
- Flask is used to expose the model as a REST API.
- `/health` confirms the service is running.
- `/predict` accepts transaction details in JSON format.
- JSON data is converted into a NumPy feature array before prediction.
- `MODEL.predict()` returns the prediction.
- The prediction is converted to a Python integer and returned as JSON.
- Flask runs on `0.0.0.0:8085` so the service is accessible through the required port.
- `curl` is a simple way to test REST APIs from the command line.
- This workflow represents a standard pattern for deploying machine learning models in production environments.


# FastAPI Model Serving – Complete Notes

# Day 58: Serving a Machine Learning Model with FastAPI

---

# Introduction

Building a Machine Learning model is only one part of an ML project. A trained model becomes useful only when users or applications can access it.

This process is called **Model Serving**.

Instead of asking users to run Python scripts manually, we expose the model through an API so any application can send data and receive predictions.

Example:

```
Mobile App
      │
      ▼
 FastAPI Server
      │
      ▼
 Machine Learning Model
      │
      ▼
Prediction Returned
```

FastAPI is one of the most popular Python frameworks for serving ML models because it is:

- Extremely fast
- Easy to write
- Automatically generates API documentation
- Supports request validation
- Uses Python type hints

---

# What is FastAPI?

FastAPI is a modern web framework for building APIs with Python.

Unlike Flask or Django, FastAPI automatically:

- validates user input
- converts JSON into Python objects
- generates API documentation
- checks data types
- returns useful error messages

Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"message":"Hello"}
```

Running this server automatically creates

```
/docs
```

which contains interactive Swagger UI.

---

# What is Model Serving?

Suppose we trained a fraud detection model.

Normally we would predict like this:

```python
prediction = model.predict(data)
```

But users cannot execute our Python file.

Instead we create an API.

User sends

```json
{
    "amount":500,
    "hour":14,
    "num_tx_past_day":2
}
```

Server predicts

```json
{
    "is_fraud":0
}
```

The user never sees the model.

They only communicate through HTTP.

---

# Project Structure

```
serving/

│
├── app.py
├── model.pkl
└── train.csv
```

---

## model.pkl

Contains the trained Random Forest model.

It has already learned fraud detection patterns.

We simply load it.

```python
MODEL = joblib.load("model.pkl")
```

Loading is much faster than training every time.

---

## train.csv

Contains the data used for training.

It is not needed during prediction.

It only helped create

```
model.pkl
```

---

## app.py

Main FastAPI application.

Everything happens here.

---

# Loading Libraries

```python
from fastapi import FastAPI
```

Creates the web application.

---

```python
from pydantic import BaseModel
```

Creates request and response schemas.

---

```python
import numpy as np
```

Used to create arrays for prediction.

---

```python
import joblib
```

Loads saved ML models.

---

# Loading the Model

```python
MODEL = joblib.load(MODEL_PATH)
```

The model is loaded once.

If we loaded it inside every request,

```
Request 1
Load model
Predict

Request 2
Load model
Predict

Request 3
Load model
Predict
```

This would be slow.

Instead

```
Start Server
↓

Load Model Once
↓

Keep in Memory
↓

Serve Thousands of Requests
```

---

# Creating FastAPI App

```python
app = FastAPI()
```

This creates the API server.

Every endpoint belongs to this app.

---

# Request vs Response

Suppose client sends

```json
{
    "amount":100,
    "hour":15,
    "num_tx_past_day":1
}
```

This is the

**Request**

Server processes it.

Server returns

```json
{
    "is_fraud":0
}
```

This is the

**Response**

---

# Why Use Pydantic?

Without validation

User could send

```json
{
    "amount":"hello"
}
```

Your ML model crashes.

Instead

Pydantic validates automatically.

Invalid requests never reach the prediction function.

---

# PredictRequest Model

```python
class PredictRequest(BaseModel):
```

Represents incoming JSON.

---

## amount

```python
amount: float = Field(..., ge=0)
```

Meaning

Must be

- decimal
- integer
- positive

Allowed

```
50

50.5

1000
```

Rejected

```
-50

"abc"

null
```

---

## ge

Means

Greater than or Equal

```
ge=0
```

Means

```
0 ✔

1 ✔

500 ✔

-5 ✘
```

---

# hour

```python
hour: int = Field(..., ge=0, le=23)
```

Represents hour of day.

Valid values

```
0
1
2
...
23
```

Invalid

```
24

25

-1
```

---

# le

Means

Less than or Equal

```
le=23
```

---

# num_tx_past_day

```python
num_tx_past_day: int = Field(..., ge=0)
```

Cannot be negative.

Allowed

```
0

5

20
```

Rejected

```
-3
```

---

# Automatic Validation

Suppose user sends

```json
{
    "hour":25
}
```

FastAPI immediately returns

```
422 Unprocessable Entity
```

Our prediction function never runs.

This is one of FastAPI's biggest advantages.

---

# PredictResponse

```python
class PredictResponse(BaseModel):
```

Defines output.

```python
{
    "is_fraud":1
}
```

Nothing more.

---

# Prediction Endpoint

```python
@app.post("/predict")
```

Creates

```
POST /predict
```

POST is used because client sends data.

---

# Request Object

```python
def predict(req: PredictRequest):
```

FastAPI automatically converts JSON into

```
req.amount

req.hour

req.num_tx_past_day
```

No manual parsing required.

---

# Creating Feature Matrix

Scikit-learn expects

```
Rows × Columns
```

Not

```
[100,12,3]
```

Instead

```
[[100,12,3]]
```

One row

Three columns

Code

```python
features = np.array([
    [
        req.amount,
        req.hour,
        req.num_tx_past_day
    ]
])
```

Shape becomes

```
(1,3)
```

Exactly what sklearn expects.

---

# Prediction

```python
MODEL.predict(features)
```

Returns

```
array([1])
```

We only need

```
1
```

Hence

```python
prediction = int(MODEL.predict(features)[0])
```

---

# Why [0]?

Because sklearn always predicts for multiple rows.

Example

```
[[1,2,3],
 [4,5,6]]
```

Returns

```
array([0,1])
```

For one row

```
array([1])
```

First prediction

```
[0]
```

---

# Recording Prediction History

```python
prediction_history.append(...)
```

Stores

```
Amount

Hour

Transactions

Prediction
```

Useful for auditing.

Example

```python
[
 {
   "amount":100,
   "hour":12,
   "num_tx_past_day":3,
   "is_fraud":0
 }
]
```

---

# Returning Response

```python
return PredictResponse(
    is_fraud=prediction
)
```

FastAPI converts this into JSON.

---

# Health Endpoint

```python
GET /health
```

Returns

```json
{
    "status":"ok"
}
```

Used by monitoring tools.

---

# Swagger UI

FastAPI automatically generates

```
/docs
```

No extra code needed.

You can

- test APIs
- see schemas
- view request examples
- inspect responses

---

# Redirect Endpoint

```python
@app.get("/")
```

Instead of showing nothing,

it redirects

```
/
```

to

```
/docs
```

Better user experience.

---

# Running the Server

Using uvicorn

```bash
uvicorn app:app --host 0.0.0.0 --port 8085
```

Server becomes available at

```
http://localhost:8085
```

Swagger

```
http://localhost:8085/docs
```

---

# Example Request

```json
{
    "amount":3200,
    "hour":23,
    "num_tx_past_day":5
}
```

Possible response

```json
{
    "is_fraud":1
}
```

---

# Another Request

```json
{
    "amount":25.5,
    "hour":10,
    "num_tx_past_day":1
}
```

Response

```json
{
    "is_fraud":0
}
```

---

# Invalid Request

```json
{
    "hour":25
}
```

FastAPI returns

```
422
```

because

```
25 > 23
```

Validation happens before prediction.

---

# Request Flow

```
Client
   │
   ▼
POST /predict
   │
   ▼
Pydantic Validation
   │
   ▼
FastAPI
   │
   ▼
NumPy Feature Array
   │
   ▼
Random Forest Model
   │
   ▼
Prediction
   │
   ▼
Store History
   │
   ▼
Return JSON
```

---

# Why FastAPI is Excellent for ML Deployment

- Very high performance
- Automatic validation
- Automatic OpenAPI generation
- Interactive Swagger documentation
- Easy integration with scikit-learn
- Built-in type checking
- Simple syntax
- Production-ready

---

# Important Concepts Learned

- Machine Learning Model Serving
- REST APIs
- FastAPI
- Uvicorn
- HTTP Methods (GET and POST)
- Request Body
- Response Model
- Pydantic Validation
- Field Constraints (`ge`, `le`)
- Swagger UI
- JSON Serialization
- NumPy Feature Arrays
- Loading Models with Joblib
- Scikit-learn Prediction
- Prediction Logging
- API Documentation
- HTTP Status Codes
- Input Validation

---

# Summary

In this project, we converted a trained Random Forest fraud detection model into a production-style REST API using FastAPI. The application loads the trained model only once at startup, validates incoming requests using Pydantic, converts transaction data into the format expected by scikit-learn, performs predictions, stores prediction history, and returns results as JSON responses. FastAPI automatically provides interactive Swagger documentation, making the API easy to test and integrate with other applications while ensuring invalid inputs are rejected before reaching the model.

# Day 59: Batch Predictions Using a Pre-Trained Machine Learning Model

## Introduction

In this task, we are working with a machine learning inference pipeline.

The ML platform team at **xFusionCorp Industries** has already trained a fraud detection model using a RandomForest algorithm. The model is stored as a serialized file (`model.pkl`).

Our responsibility is not to train the model. Instead, we need to perform **batch inference**.

Batch inference means:

- We receive a group of input records at once.
- We pass every record through the trained model.
- We generate predictions for all records.
- We save the results for further processing.

This is different from real-time prediction, where one request is processed at a time through an API.

---

# Project Structure

The project exists inside:

```text
/root/code/serving/
```

The files are:

```text
/root/code/serving/
│
├── model.pkl
├── input.csv
├── batch_predict.py
└── predictions.csv
```

---

# Understanding Each File

## 1. model.pkl

`model.pkl` is the trained machine learning model.

The model was created using a RandomForest classifier.

It was trained on a synthetic fraud detection dataset with these features:

- `amount`
- `hour`
- `num_tx_past_day`

The target label is:

```
is_fraud
```

The model learned patterns from historical transaction data and can now classify new transactions.

The output classes are:

```
0 = Not Fraud
1 = Fraud
```

The model file is serialized using Python's `joblib` library.

Serialization means:

- Convert a trained Python object into a file.
- Store it permanently.
- Load it later without retraining.

---

# 2. input.csv

This file contains the new transaction data that needs predictions.

Example:

```csv
amount,hour,num_tx_past_day
1200,10,5
50,22,1
8000,2,15
```

Important points:

- It contains only feature columns.
- It does not contain the target label.
- The model will predict the missing label.

The columns are:

| Column | Meaning |
|---|---|
| amount | Transaction amount |
| hour | Hour of transaction |
| num_tx_past_day | Number of previous transactions in the day |

---

# 3. batch_predict.py

This is the inference script.

Its job is:

1. Load the trained model.
2. Read new transaction data.
3. Select required features.
4. Generate predictions.
5. Save predictions.

---

# Importing Required Libraries

The script starts with:

```python
import joblib
import pandas as pd
```

## joblib

`joblib` is used for loading the trained machine learning model.

Example:

```python
model = joblib.load("model.pkl")
```

It restores the saved RandomForest object.

---

## pandas

Pandas is used for working with CSV files and tables.

It provides:

- Reading CSV files
- Selecting columns
- Adding new columns
- Writing output files

Example:

```python
df = pd.read_csv("input.csv")
```

---

# File Path Configuration

The script contains:

```python
MODEL_PATH = "/root/code/serving/model.pkl"

INPUT_CSV = "/root/code/serving/input.csv"

OUTPUT_CSV = "/root/code/serving/predictions.csv"
```

These constants store the locations of:

- The trained model
- The input dataset
- The output prediction file

Using constants makes the script easier to maintain.

If the file location changes, we only update one place.

---

# Loading the Machine Learning Model

Code:

```python
model = joblib.load(MODEL_PATH)
```

What happens internally:

1. Python opens `model.pkl`.
2. The serialized RandomForest object is restored.
3. The model is ready to make predictions.

At this stage:

```
model
 |
 |
 RandomForest Classifier
 |
 |
 Ready for inference
```

---

# Reading Input Data

Code:

```python
df = pd.read_csv(INPUT_CSV)
```

This loads the CSV file into a pandas DataFrame.

A DataFrame is a table-like structure.

Example:

Before loading:

```csv
amount,hour,num_tx_past_day
100,9,2
500,15,4
```

After loading:

```
   amount  hour  num_tx_past_day

0    100     9          2
1    500    15          4
```

---

# Selecting Feature Columns

The model was trained using:

```
amount
hour
num_tx_past_day
```

Therefore, prediction input must contain exactly these columns.

Code:

```python
features = df[
    [
        "amount",
        "hour",
        "num_tx_past_day"
    ]
]
```

Why select columns?

Because:

- The model expects the same features used during training.
- Extra columns may cause errors.
- Missing columns will cause prediction failure.

Machine learning models require the same input format during training and inference.

---

# Generating Predictions

The important step:

```python
df["prediction"] = model.predict(features)
```

## predict()

`model.predict()` returns the predicted class.

Example:

Input:

```
amount = 10000
hour = 3
num_tx_past_day = 20
```

Output:

```
1
```

Meaning:

```
Fraud transaction
```

Another example:

```
amount = 20
hour = 14
num_tx_past_day = 1
```

Output:

```
0
```

Meaning:

```
Normal transaction
```

---

# predict() vs predict_proba()

A very important concept.

## model.predict()

Returns class labels.

Example:

```python
model.predict(features)
```

Output:

```text
[0,1,0,1]
```

These are final decisions.

---

## model.predict_proba()

Returns probabilities.

Example:

```python
model.predict_proba(features)
```

Output:

```text
[
 [0.95,0.05],
 [0.20,0.80]
]
```

Meaning:

First transaction:

```
95% not fraud
5% fraud
```

Second transaction:

```
20% not fraud
80% fraud
```

For this task, we need:

```
0 or 1 labels
```

Therefore we use:

```python
model.predict()
```

not:

```python
model.predict_proba()
```

---

# Converting Predictions to Integer

Code:

```python
.astype(int)
```

Example:

Before:

```
0.0
1.0
0.0
```

After:

```
0
1
0
```

The requirement says:

- Prediction must be an integer class label.
- It must not be a probability.
- Values must be only `0` or `1`.

---

# Saving the Output

Code:

```python
df.to_csv(
    OUTPUT_CSV,
    index=False
)
```

This creates:

```
predictions.csv
```

The output contains:

- Original feature columns
- New prediction column

Example:

```csv
amount,hour,num_tx_past_day,prediction
100,9,2,0
500,15,4,1
```

---

# Why index=False?

Without:

```python
index=False
```

Pandas adds an extra index column.

Example:

Wrong:

```csv
,index,amount,hour,prediction
0,100,9,0
```

Correct:

```csv
amount,hour,prediction
100,9,0
```

For ML pipelines, extra unwanted columns can create problems.

---

# Complete batch_predict.py Logic

The complete workflow:

```python
import joblib
import pandas as pd

MODEL_PATH = "/root/code/serving/model.pkl"
INPUT_CSV = "/root/code/serving/input.csv"
OUTPUT_CSV = "/root/code/serving/predictions.csv"


model = joblib.load(MODEL_PATH)

df = pd.read_csv(INPUT_CSV)

features = df[
    [
        "amount",
        "hour",
        "num_tx_past_day"
    ]
]

df["prediction"] = model.predict(features).astype(int)

df.to_csv(
    OUTPUT_CSV,
    index=False
)

print(
    f"Wrote {len(df)} rows to {OUTPUT_CSV}"
)
```

---

# Running the Script

Execute:

```bash
python3 /root/code/serving/batch_predict.py
```

Expected output:

```text
Wrote 10 rows to /root/code/serving/predictions.csv
```

---

# Validation Steps

## Check File Exists

Command:

```bash
ls -l /root/code/serving/predictions.csv
```

The file should exist.

---

## View Output

Command:

```bash
cat /root/code/serving/predictions.csv
```

Example:

```csv
amount,hour,num_tx_past_day,prediction
500,10,3,0
9000,2,20,1
```

---

# Requirements Checklist

## Model Loading

Completed:

```python
joblib.load(MODEL_PATH)
```

---

## Reading Input

Completed:

```python
pd.read_csv(INPUT_CSV)
```

---

## Feature Selection

Completed:

```python
amount
hour
num_tx_past_day
```

---

## Prediction Generation

Completed:

```python
model.predict()
```

---

## Integer Labels

Completed:

```python
.astype(int)
```

---

## Output Generation

Completed:

```python
to_csv(index=False)
```

---

## Row Count Validation

Input:

```
10 rows
```

Output:

```
10 rows
```

The number of predictions must always match the number of input transactions.

---

# Key Machine Learning Concepts Learned

## Training vs Inference

Training:

```
Historical Data
      |
      |
 Machine Learning Algorithm
      |
      |
 Saved Model
```

Inference:

```
New Data
      |
      |
 Saved Model
      |
      |
 Prediction
```

---

## Batch Inference Pipeline

The complete flow:

```
input.csv
    |
    |
Read Data
    |
    |
Select Features
    |
    |
Load Model
    |
    |
model.predict()
    |
    |
Add Prediction Column
    |
    |
Save predictions.csv
```

---

# Final Outcome

The fraud detection batch scoring pipeline is successfully implemented.

The system can now:

- Load a trained RandomForest model.
- Process multiple transactions together.
- Generate fraud predictions.
- Store results in a CSV file.

This completes the batch prediction workflow for the ML serving environment.

# Day 60 Notes: Packaging and Serving a Machine Learning Model with BentoML

# Introduction

Machine Learning is not complete when a model is trained. A trained model sitting inside a Jupyter Notebook has no practical value until other applications can use it.

This process of making a trained model available to users or applications is called **Model Serving**.

Today we learned how BentoML simplifies this entire deployment process.

---

# What is BentoML?

BentoML is an open-source framework for packaging and serving machine learning models.

Instead of writing an entire web server manually using Flask or FastAPI, BentoML handles most of the infrastructure for us.

It provides:

- Model management
- Model versioning
- REST API generation
- Swagger documentation
- Production deployment support
- Multiple framework support

Supported frameworks include:

- Scikit-Learn
- TensorFlow
- PyTorch
- XGBoost
- LightGBM
- ONNX
- HuggingFace Transformers

Think of BentoML as the bridge between your trained model and the real world.

---

# Traditional Deployment vs BentoML

Without BentoML:

```
Train Model
      ↓
Save Model
      ↓
Load Model
      ↓
Create Flask/FastAPI
      ↓
Write Routes
      ↓
Handle JSON
      ↓
Prediction
      ↓
Deploy
```

With BentoML:

```
Train Model
      ↓
Save using BentoML
      ↓
Create Service
      ↓
Run bentoml serve
      ↓
Ready API
```

Much less code.

---

# BentoML Architecture

```
               Client

                  |

          HTTP Request

                  |

          BentoML Service

                  |

          Loaded ML Model

                  |

           Prediction

                  |

          HTTP Response
```

---

# What is a Model Store?

Normally we save models like this:

```python
joblib.dump(model, "model.pkl")
```

or

```python
pickle.dump(model, file)
```

The problem:

- no versioning
- difficult management
- easy to overwrite
- manual loading

BentoML solves this using a **Model Store**.

Example:

```
fraud_detector:latest
```

or

```
fraud_detector:v1
```

or

```
fraud_detector:v2
```

Each model is stored with metadata.

---

# Saving a Model

Example:

```python
bentoml.sklearn.save_model(
    "fraud_detector",
    model
)
```

This registers the model inside BentoML's model store.

Later we can load it without knowing its exact file location.

---

# Listing Stored Models

Command:

```bash
bentoml models list
```

Example:

```
Tag

fraud_detector:latest
```

This confirms that BentoML knows about the model.

---

# Loading a Model

Our service loads the model like this:

```python
bento_model = bentoml.models.BentoModel(
    "fraud_detector:latest"
)
```

Inside the constructor:

```python
self.model = bentoml.sklearn.load_model(
    self.bento_model
)
```

Now the trained RandomForest model is available.

---

# Understanding the Service

Our service begins with

```python
@bentoml.service
class FraudService:
```

This decorator tells BentoML:

"This class is an API service."

Everything inside becomes part of the HTTP server.

---

# Service Lifecycle

When the server starts

```
Server Starts

↓

FraudService created

↓

__init__()

↓

Model Loaded

↓

Ready to receive requests
```

Notice:

The model loads **once**.

It is **not loaded for every request**.

This makes prediction much faster.

---

# __init__()

```python
def __init__(self):
```

Purpose:

Initialize everything once.

Here we load

```
RandomForest Model
```

and create

```
Prediction History
```

```
self._history = []
```

---

# API Endpoints

Every endpoint uses

```python
@bentoml.api
```

Example:

```python
@bentoml.api
def predict(...)
```

This automatically becomes

```
POST /predict
```

Similarly,

```python
@bentoml.api
def last_predictions()
```

becomes

```
POST /last_predictions
```

No Flask routing required.

---

# Input Parameters

Our prediction function accepts

```python
amount

hour

num_tx_past_day
```

These represent

Transaction Amount

Transaction Hour

Number of Transactions in Last Day

---

# Why NumPy?

Scikit-Learn expects data like

```
Rows × Columns
```

One sample containing three features becomes

```
[[3200,23,5]]
```

Notice the double brackets.

Single brackets

```
[3200,23,5]
```

are incorrect.

---

# Creating Features

```python
features = np.array([
    [amount,
     hour,
     num_tx_past_day]
])
```

Result:

```
1 sample

3 features
```

Shape:

```
(1,3)
```

---

# Prediction

```python
prediction = self.model.predict(features)
```

Output

```
array([1])
```

or

```
array([0])
```

Since JSON cannot return NumPy integers,

we convert

```python
int(prediction[0])
```

Now

```
1

or

0
```

---

# Why Convert to int?

NumPy returns

```
numpy.int64
```

JSON expects

```
int
```

Therefore

```python
int(...)
```

avoids serialization problems.

---

# Prediction History

Every request is saved.

```python
self._history.append(
{
...
}
)
```

Stored information

```
Amount

Hour

Transactions

Prediction
```

This creates an audit log.

---

# Returning JSON

We return

```python
{
"is_fraud": prediction
}
```

BentoML automatically converts this dictionary into

```
JSON
```

Example

```json
{
    "is_fraud":1
}
```

---

# last_predictions Endpoint

Returns

```python
{
    "count": len(...),
    "predictions": ...
}
```

Useful for

- debugging
- auditing
- monitoring

---

# Starting the Server

Command

```bash
bentoml serve service:FraudService --port 3000
```

Meaning

```
service

↓

Find FraudService

↓

Start HTTP server

↓

Load model

↓

Wait for requests
```

---

# Swagger UI

Open

```
http://localhost:3000
```

Swagger is automatically generated.

Benefits

- Interactive testing
- API documentation
- No Postman required
- Shows request schema
- Shows response schema

---

# Testing with curl

Example

```bash
curl -X POST \
http://localhost:3000/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

The request contains

```
JSON

↓

HTTP POST

↓

Server

↓

Prediction

↓

JSON Response
```

---

# Example Request

```json
{
    "amount":3200,
    "hour":23,
    "num_tx_past_day":5
}
```

Features

High Amount

Late Night

Many Transactions

Likely Fraud

---

# Example Response

```json
{
    "is_fraud":1
}
```

---

# Low Risk Example

```json
{
    "amount":25.5,
    "hour":10,
    "num_tx_past_day":1
}
```

Expected

```json
{
    "is_fraud":0
}
```

---

# Complete Request Flow

```
User

↓

POST /predict

↓

BentoML API

↓

FraudService

↓

NumPy Feature Array

↓

RandomForest.predict()

↓

Prediction

↓

Dictionary

↓

JSON

↓

Client
```

---

# Why Swagger Matters

Without Swagger

- Need Postman
- Need curl
- Need documentation

With Swagger

- Click endpoint
- Enter values
- Press Execute
- View response instantly

---

# Why Keep Prediction History?

Useful for

Debugging

Finding incorrect predictions

Logging

Auditing

Monitoring production systems

---

# Why BentoML is Popular

It automatically provides

Model Store

Versioning

Dependency Management

REST APIs

Swagger

Docker Support

Cloud Deployment

Model Packaging

Production Serving

Minimal Code

---

# Real-World Workflow

```
Collect Data

↓

Clean Data

↓

Train Model

↓

Evaluate

↓

Save using BentoML

↓

Create Service

↓

Serve Model

↓

Users send requests

↓

Receive Predictions

↓

Monitor Performance
```

---

# Important Commands

List models

```bash
bentoml models list
```

Start server

```bash
bentoml serve service:FraudService --port 3000
```

Test homepage

```bash
curl http://localhost:3000/
```

Predict

```bash
curl -X POST http://localhost:3000/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

Prediction history

```bash
curl -X POST http://localhost:3000/last_predictions
```

---

# Key Interview Questions

### What is BentoML?

A framework used to package, manage, and serve machine learning models as production-ready APIs.

---

### Why use BentoML instead of Flask?

Because BentoML provides built-in model management, automatic API generation, Swagger UI, versioning, and deployment tools, reducing the amount of infrastructure code you need to write.

---

### What is the BentoML Model Store?

A centralized local repository where trained models are stored with names, versions, and metadata instead of as standalone files.

---

### What does `@bentoml.service` do?

It marks a class as a BentoML service that can expose APIs over HTTP.

---

### What does `@bentoml.api` do?

It exposes a class method as an HTTP endpoint.

---

### Why is the model loaded in `__init__()`?

To load the model only once when the service starts, improving performance by avoiding repeated loading for every request.

---

### Why use `np.array([[...]])`?

Scikit-Learn expects a 2D array where each row represents one sample and each column represents one feature.

---

### Why convert the prediction to `int`?

Because Scikit-Learn returns a NumPy integer (`numpy.int64`), which should be converted to a native Python `int` for JSON serialization.

---

### What is Swagger UI?

An automatically generated web interface that documents the API and allows users to test endpoints directly from the browser.

---

# Summary

In this lesson, we learned how to take a trained Scikit-Learn model and make it available as a production-ready web service using BentoML. We explored model registration, the BentoML Model Store, service creation with `@bentoml.service`, API endpoints with `@bentoml.api`, prediction using NumPy feature arrays, maintaining request history, starting the server, testing endpoints with `curl`, and using Swagger UI for interactive API testing. These concepts form the foundation of deploying machine learning models for real-world applications.

# Day 61 Notes: Deploy a Model-Serving Container via Portainer

## Introduction

Today we are learning how to deploy a machine learning model-serving application using **Portainer**.

In a real production environment, ML teams usually do not manually run Docker commands every time. Instead, they use management platforms like Portainer to:

- Inspect Docker images
- Create and manage containers
- View container logs
- Configure networking
- Manage storage volumes
- Restart services
- Monitor running applications

In this task, the ML platform team has already created a Docker image called:

```
fraud-detector:v1
```

Our responsibility is to deploy this image as a running API service using the Portainer web interface.

---

# Understanding the Architecture

The complete flow looks like this:

```
Developer
    |
    |
    v
FastAPI Application
(app.py)
    |
    |
    v
Docker Image
(fraud-detector:v1)
    |
    |
    v
Portainer
(Container Management UI)
    |
    |
    v
Docker Container
(fraud-api)
    |
    |
    v
API Users
```

The container will run a FastAPI application that loads a machine learning model:

```
model.pkl
```

and exposes two endpoints:

```
GET  /health
POST /predict
```

---

# Project Structure

The application files are stored on the host machine:

```
/root/code/serving/
```

The directory contains:

```
serving/
|
├── app.py
├── Dockerfile
└── model.pkl
```

---

# Understanding Each File

## app.py

`app.py` contains the FastAPI application.

Its responsibilities:

1. Start the API server.
2. Load the machine learning model.
3. Create API endpoints.
4. Receive prediction requests.
5. Return prediction results.

The application loads:

```
/app/model.pkl
```

inside the container.

---

## model.pkl

This is the trained machine learning model.

The model is:

```
RandomForest
```

It was created during training and saved using joblib/pickle.

The API uses this model to predict whether a transaction is fraudulent.

---

## Dockerfile

The Dockerfile defines how the image was created.

It uses:

```
python:3.11-slim
```

as the base image.

It installs:

```
fastapi
uvicorn
joblib
scikit-learn
```

The container starts using:

```
uvicorn app:app
```

The application listens on:

```
port 8085
```

---

# What Is Portainer?

Portainer is a graphical interface for Docker management.

Instead of writing:

```bash
docker run
```

commands manually, we can create containers using forms.

Portainer communicates with Docker through:

```
/var/run/docker.sock
```

This socket allows Portainer to control the Docker engine.

---

# Portainer Docker Connection

When Portainer starts, it needs access to Docker.

The important mount is:

```
/var/run/docker.sock:/var/run/docker.sock
```

Meaning:

Host Docker socket:

```
/var/run/docker.sock
```

is shared with the Portainer container.

Without this:

- Portainer opens successfully
- But it cannot see Docker containers
- It cannot create new containers

---

# Logging Into Portainer

Portainer runs on:

```
http://<server>:9090
```

Credentials:

```
Username:
admin

Password:
xFusionCorp2026!
```

After login:

Select:

```
local
```

environment.

The local environment represents the Docker engine running on the server.

---

# Container Deployment Concepts

Before deploying, understand the important Docker options.

---

# 1. Container Name

Every running container needs an identifier.

Example:

```
fraud-api
```

This allows us to manage it:

```bash
docker logs fraud-api
docker restart fraud-api
docker inspect fraud-api
```

---

# 2. Docker Image

An image is a packaged application.

Here:

```
fraud-detector:v1
```

contains:

- Python runtime
- Required libraries
- Application code
- Startup command

A container is created from an image.

Relationship:

```
Image
 |
 |
 v
Container
```

Example:

```
fraud-detector:v1
        |
        |
        v
    fraud-api
```

---

# 3. Port Mapping

Containers have their own network.

The application listens inside the container:

```
Container port:
8085
```

Users access the service through the host machine:

```
Host port:
8085
```

Mapping:

```
Host               Container

8085  -----------> 8085
```

Docker option:

```bash
-p 8085:8085
```

Without port mapping:

- Container works internally
- Users cannot access the API

---

# 4. Volume Mounting

A volume connects host files with container files.

Required mount:

```
Host:
/root/code/serving

Container:
/app
```

Mapping:

```
/root/code/serving
        |
        |
        v
       /app
```

Docker option:

```bash
-v /root/code/serving:/app
```

Why do we need this?

Because the application expects:

```
/app/app.py
/app/model.pkl
```

The bind mount allows the container to read the files from the host.

Benefits:

- Update code without rebuilding image
- Replace model files easily
- Maintain external configuration

---

# 5. Restart Policy

Containers can stop because of:

- Server restart
- Application failure
- Docker restart

Restart policy:

```
Always
```

means Docker automatically starts the container again.

Production services usually use restart policies.

---

# Creating the Container in Portainer

Steps:

1. Open Portainer.
2. Select:

```
local environment
```

3. Go to:

```
Containers
```

4. Click:

```
Add container
```

5. Enter:

```
Name:
fraud-api
```

6. Enter image:

```
fraud-detector:v1
```

7. Configure port:

```
Host:
8085

Container:
8085
```

8. Add volume:

```
Host:
/root/code/serving

Container:
/app
```

9. Set restart policy:

```
Always
```

10. Deploy.

---

# Equivalent Docker Command

The Portainer form creates an equivalent Docker command:

```bash
docker run -d \
--name fraud-api \
-p 8085:8085 \
-v /root/code/serving:/app \
--restart always \
fraud-detector:v1
```

Understanding Docker commands helps us understand what Portainer does internally.

---

# Container Verification

After deployment:

Check running containers:

```bash
docker ps
```

Expected:

```
fraud-api
```

should be running.

---

# Inspect Container

Command:

```bash
docker inspect fraud-api
```

This shows:

- Image information
- Network configuration
- Volume mounts
- Environment variables
- Container state

Important verification:

```
/root/code/serving
        |
        v
       /app
```

---

# Testing the Application

## Health Endpoint

Purpose:

Checks whether the API is alive.

Request:

```bash
curl http://localhost:8085/health
```

Response:

```json
{
 "status":"ok"
}
```

A successful response means:

- Container is running
- FastAPI started
- Network port works

---

# Prediction Endpoint

The prediction API accepts transaction information.

Example request:

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":3200,"hour":23,"num_tx_past_day":5}'
```

Input:

```
amount = 3200
hour = 23
transactions today = 5
```

The model processes the values.

Response:

```json
{
"is_fraud":0
}
```

or:

```json
{
"is_fraud":1
}
```

Meaning:

```
0 = Not Fraud
1 = Fraud
```

---

# Troubleshooting Guide

## Problem: Portainer Cannot See Docker

Check:

```bash
docker inspect portainer
```

Look for:

```
/var/run/docker.sock
```

If missing, recreate Portainer with:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

---

## Problem: Container Starts But API Fails

Check logs:

```bash
docker logs fraud-api
```

Common issues:

- Missing model file
- Wrong application path
- Dependency errors
- Port mismatch

---

## Problem: Port Already Used

Check:

```bash
docker ps
```

or:

```bash
netstat -tulpn
```

Another service may already use:

```
8085
```

---

## Problem: Model File Missing

Check inside container:

```bash
docker exec -it fraud-api bash
```

Then:

```bash
ls /app
```

Expected:

```
app.py
model.pkl
```

---

# Important Docker Concepts Learned

## Image

A read-only package containing an application.

Example:

```
fraud-detector:v1
```

---

## Container

A running instance of an image.

Example:

```
fraud-api
```

---

## Volume

Persistent storage shared between host and container.

Example:

```
/root/code/serving:/app
```

---

## Port Mapping

Allows outside users to access container applications.

Example:

```
8085:8085
```

---

## Docker Socket

Allows tools like Portainer to manage Docker.

Example:

```
/var/run/docker.sock
```

---

# Final Architecture

```
                User
                 |
                 |
                 v
        localhost:8085
                 |
                 |
                 v
          fraud-api Container
                 |
        -------------------
        |                 |
        v                 v
     app.py          model.pkl
        |
        |
        v
   RandomForest Model
```

---

# Final Checklist

Deployment is successful when:

- Portainer is running on port `9090`
- Local Docker environment is connected
- Container name is `fraud-api`
- Image is `fraud-detector:v1`
- Port `8085` is published
- `/root/code/serving` is mounted to `/app`
- `/health` returns:

```json
{"status":"ok"}
```

- `/predict` returns:

```json
{"is_fraud":0}
```

or:

```json
{"is_fraud":1}
```

---

# Summary

In this lesson, we learned how to deploy a machine learning model-serving API using Portainer.

The main production workflow is:

1. Build Docker image.
2. Store application and model files.
3. Deploy container using Portainer.
4. Configure networking.
5. Mount required files.
6. Verify API health.
7. Test machine learning predictions.

This workflow is commonly used in MLOps environments to deploy and manage AI services.

# Day 62 Notes: Implement A/B Testing for Model Deployment

## Topic: A/B Testing in Machine Learning Model Deployment

---

# 1. Introduction

In machine learning projects, developing a model is only one part of the journey. The real challenge begins when we deploy the model into production.

A production ML system must handle:

- Receiving user requests
- Preparing input data
- Running model predictions
- Returning predictions quickly
- Monitoring model performance
- Updating models safely

When a new model version is created, replacing the existing production model immediately can be risky.

The new model may:

- Perform worse on real-world data
- Have unexpected behavior
- Create incorrect predictions
- Affect business decisions

To reduce this risk, companies use **A/B testing for model deployment**.

---

# 2. What is A/B Testing?

A/B testing is a technique where two versions of a system are operated at the same time, and real traffic is divided between them.

In machine learning:

- Version A → Existing stable model
- Version B → New candidate model

The goal is to compare both models using real production traffic.

Example:

```
Incoming Requests
        |
        |
     A/B Router
      /      \
     /        \
  80%          20%
   |            |
 MODEL_V1    MODEL_V2
 Stable      Candidate
```

The stable model continues handling most users while the new model receives a smaller percentage of requests.

---

# 3. Why Use A/B Testing for ML Models?

Deploying a new model directly can introduce problems.

A/B testing provides:

## Safety

The old model continues serving most users.

Example:

```
Old Model  → 80% traffic
New Model  → 20% traffic
```

If the new model performs poorly, the impact is limited.

---

## Real-World Evaluation

Offline testing is done using historical datasets.

However, production traffic may contain:

- New user behavior
- Different data distributions
- Unexpected patterns

A/B testing evaluates the model using real requests.

---

## Performance Comparison

Teams can compare:

- Accuracy
- Fraud detection rate
- False positives
- False negatives
- Latency
- Business impact

---

# 4. Scenario: Fraud Detection System

In this project, xFusionCorp Industries has a fraud detection platform.

Two RandomForest models exist:

```
model_v1.pkl
model_v2.pkl
```

They represent two versions of the fraud detection model.

---

## Model Versions

### MODEL_V1

Stable production model.

Properties:

- Already trusted
- Handles most traffic
- Used as the baseline

Traffic:

```
80%
```

---

### MODEL_V2

New candidate model.

Properties:

- Recently trained
- Needs production testing
- Performance must be compared

Traffic:

```
20%
```

---

# 5. Project Structure

The application directory:

```
/root/code/serving/

├── model_v1.pkl
├── model_v2.pkl
└── ab_server.py
```

Explanation:

## model_v1.pkl

Contains the first trained RandomForest model.

This is the stable production version.

---

## model_v2.pkl

Contains the second trained RandomForest model.

This is the experimental version.

---

## ab_server.py

Flask application responsible for:

- Loading models
- Receiving requests
- Routing traffic
- Running predictions
- Returning responses

---

# 6. Flask Server Architecture

The application uses Flask.

Flask provides:

- HTTP endpoints
- Request handling
- JSON responses

The server exposes two endpoints:

```
/health
/predict
```

---

# 7. Health Endpoint

The health endpoint checks whether the server is running.

Example:

Request:

```
GET /health
```

Response:

```json
{
    "status": "ok"
}
```

This is commonly used by:

- Kubernetes probes
- Monitoring systems
- Load balancers

---

# 8. Prediction Endpoint

The prediction endpoint receives transaction information.

Example request:

```json
{
    "amount": 100.5,
    "hour": 12,
    "num_tx_past_day": 3
}
```

The server converts this JSON data into model input.

---

# 9. Request Processing Flow

The complete prediction flow:

```
Client
  |
  |
POST /predict
  |
  |
Flask receives JSON
  |
  |
Convert JSON to numpy array
  |
  |
A/B Router selects model
  |
  |
Selected model predicts
  |
  |
Return prediction + model version
```

---

# 10. Input Feature Preparation

The incoming JSON:

```json
{
 "amount":100,
 "hour":12,
 "num_tx_past_day":5
}
```

is converted into:

```python
[
 [
  100,
  12,
  5
 ]
]
```

Machine learning models require numerical arrays as input.

---

# 11. A/B Routing Logic

The important part of the implementation is selecting the model.

Python provides:

```python
random.random()
```

It generates a random floating-point number between:

```
0.0 and 1.0
```

Example:

```
0.25
0.73
0.91
```

---

The routing rule:

```python
if random.random() < 0.8:
```

means:

```
80% probability
```

because numbers from:

```
0.0 - 0.79
```

represent approximately 80% of possible values.

---

The implementation:

```python
if random.random() < 0.8:
    model = MODEL_V1
    model_version = "v1"
else:
    model = MODEL_V2
    model_version = "v2"
```

---

# 12. Prediction Execution

After selecting the model:

```python
prediction = model.predict(features)[0]
```

The selected model performs inference.

Example:

MODEL_V1:

```
Input transaction
       |
       |
Prediction
       |
       |
Fraud = 0
```

---

# 13. Why Return model_version?

A prediction alone is not enough.

Example:

```json
{
 "is_fraud":1
}
```

Monitoring systems do not know:

- Which model predicted this?
- Was it v1?
- Was it v2?

Therefore every response includes:

```json
{
 "is_fraud":1,
 "model_version":"v2"
}
```

Now the system can measure each model separately.

---

# 14. Final API Response Format

Successful response:

```json
{
    "is_fraud": 0,
    "model_version": "v1"
}
```

Fields:

## is_fraud

Prediction result.

Values:

```
0 → Not Fraud
1 → Fraud
```

---

## model_version

Model that generated the prediction.

Possible values:

```
v1
v2
```

---

# 15. Testing the Application

## Start Server

Command:

```bash
python3 ab_server.py
```

The server starts:

```
Port: 8085
```

---

## Check Health

Command:

```bash
curl http://localhost:8085/health
```

Expected:

```json
{
 "status":"ok"
}
```

---

## Send Prediction Request

Command:

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":200,"hour":10,"num_tx_past_day":4}'
```

Possible output:

```json
{
 "is_fraud":0,
 "model_version":"v1"
}
```

---

# 16. Testing the Traffic Split

Because routing is random, one request does not prove the split.

We test many requests.

Example:

Send:

```
200 requests
```

Expected result:

```
MODEL_V1:
Around 160 requests

MODEL_V2:
Around 40 requests
```

Allowed variation exists because randomness is involved.

---

# 17. Production Monitoring

After deployment, monitoring systems can calculate:

## Model Usage

Example:

```
v1:
80%

v2:
20%
```

---

## Model Performance

Compare:

```
v1 accuracy
vs
v2 accuracy
```

---

## Business Metrics

For fraud detection:

- Fraud caught
- False alarms
- Customer complaints
- Transaction approval rate

---

# 18. Rollout Strategies

A/B testing is one deployment strategy.

Other strategies:

## Blue-Green Deployment

Two environments:

```
Blue  → Current version
Green → New version
```

Traffic switches completely after validation.

---

## Canary Deployment

Small percentage first:

```
99% old model
1% new model
```

Then gradually increase.

---

## Shadow Deployment

New model receives copied traffic but does not affect users.

---

# 19. Common Mistakes

## Mistake 1: Forgetting model_version

Bad:

```json
{
"is_fraud":1
}
```

Problem:

Cannot identify model source.

---

## Mistake 2: Wrong traffic split

Example:

```python
random.random() < 0.5
```

This creates:

```
50/50 split
```

Not required.

---

## Mistake 3: Loading model for every request

Bad:

```python
model = joblib.load(...)
```

inside `/predict`.

Problem:

- Slow
- Wastes resources

Correct:

Load once when application starts.

---

## Mistake 4: Returning wrong data type

Prediction should be JSON serializable.

Correct:

```python
int(prediction)
```

---

# 20. Final Implementation Checklist

Before completing the task verify:

[x] Both models are loaded

[x] Flask server starts successfully

[x] `/health` endpoint works

[x] `/predict` accepts JSON input

[x] Traffic split is 80/20

[x] MODEL_V1 returns `"v1"`

[x] MODEL_V2 returns `"v2"`

[x] Every response contains:

```json
{
"is_fraud": value,
"model_version": value
}
```

[x] Batch testing shows approximately:

```
160 requests → v1
40 requests  → v2
```

---

# 21. Key Takeaways

- A/B testing reduces deployment risk.
- ML models should be monitored after deployment.
- Traffic routing allows safe model comparison.
- Model version tracking is essential for production observability.
- Randomized routing is a simple way to implement controlled experiments.

A/B testing transforms model deployment from a risky replacement process into a measurable and controlled release process.

# Day 63 Notes – Async Predictions with a Redis-Backed Worker

# Introduction

In production Machine Learning systems, predictions are not always returned immediately.

For small models, synchronous prediction works well because the response is generated within milliseconds. However, for large models such as fraud detection, recommendation systems, image recognition, NLP models, or deep learning models, inference can take several seconds or even minutes.

If an API waits until prediction is complete, users experience slow responses, requests may time out, and the server becomes less scalable.

To solve this problem, production systems often use **asynchronous processing**.

Instead of making the client wait:

1. Accept the request immediately.
2. Give the client a unique Task ID.
3. Process the prediction in the background.
4. Store the result somewhere.
5. Let the client retrieve the result later.

This architecture is used by many large-scale systems including OpenAI, AWS, Azure ML, Google Cloud AI, Stripe, and payment fraud detection services.

---

# What is Asynchronous Processing?

Asynchronous means:

> Start a task now and finish it later.

The client does **not** wait for the task to finish.

Instead:

```
Client
   |
   | POST /predict-async
   |
Server
   |
   | Generate task_id
   |
   | Start background worker
   |
   | Return task_id immediately
   |
Client receives response instantly
```

Later...

```
Background Worker
      |
      |
Runs ML Model
      |
Stores Result in Redis
      |
Client polls for result
```

This keeps APIs extremely fast.

---

# Synchronous vs Asynchronous

## Synchronous

```
Client
   |
POST /predict
   |
Server
   |
Run ML Model
   |
Return prediction
```

The client waits.

If prediction takes:

```
5 seconds
```

The client waits for all 5 seconds.

---

## Asynchronous

```
Client
    |
POST /predict-async
    |
Server
    |
Create task_id
    |
Return task_id immediately
```

Background:

```
Run prediction
Store result
```

Client:

```
GET /result/task_id
```

The client can continue doing other work.

---

# Why Companies Prefer Async APIs

Imagine an image classification model.

Inference time:

```
12 seconds
```

Without async:

```
Client waits

12 seconds...
```

With async:

```
POST request

↓

Task ID returned in 5 ms

↓

Prediction runs in background

↓

Client checks later
```

This is a much better user experience.

---

# Real World Examples

## ChatGPT

You submit a prompt.

The request starts processing.

You don't manually poll, but internally the request is processed asynchronously.

---

## YouTube

Upload video.

Immediately:

```
Upload successful
```

Video processing happens later.

---

## Google Drive

Upload PDF.

Background tasks:

- Virus scanning
- Thumbnail generation
- OCR

---

## Banking Systems

When a transaction occurs:

```
Fraud detection

Risk scoring

Notifications

Logging
```

Many of these run asynchronously.

---

# Project Overview

The application consists of:

```
Client
    |
    |
Flask API
    |
Background Thread
    |
ML Model
    |
Redis
```

The ML model predicts whether a transaction is fraudulent.

---

# Project Flow

```
POST /predict-async

↓

Generate task_id

↓

Return task_id

↓

Background Thread Starts

↓

Run Model

↓

Prediction Generated

↓

Save in Redis

↓

Client Requests Result

↓

GET /result/task_id

↓

Prediction Returned
```

---

# Project Files

```
serving/

│

├── async_app.py

├── model.pkl
```

---

# model.pkl

Contains the trained Random Forest model.

Loaded using:

```python
MODEL = joblib.load("/root/code/serving/model.pkl")
```

---

# Why Joblib?

Machine Learning models are large Python objects.

Joblib efficiently serializes them.

```
Training

↓

Save

↓

Load later
```

---

# Redis

Redis is an in-memory database.

It stores data in RAM.

Advantages:

- Very fast
- Lightweight
- Key-value storage
- Supports expiration
- Ideal for caching
- Excellent for temporary task results

---

# Redis Connection

```python
REDIS = redis.Redis(
    host="localhost",
    port=6379,
    decode_responses=True
)
```

Explanation:

### host

```
localhost
```

Redis runs on the local machine.

---

### port

```
6379
```

Default Redis port.

---

### decode_responses

Without:

Redis returns

```
b'1'
```

(bytes)

With:

```
"1"
```

(string)

Much easier to use.

---

# Result Key

```python
RESULT_KEY = "result:{task_id}"
```

Suppose:

```
task_id

abc123
```

The stored key becomes

```
result:abc123
```

Another task:

```
xyz789
```

Stored as

```
result:xyz789
```

Each task has a unique Redis key.

---

# TTL

```python
RESULT_TTL_SECONDS = 600
```

TTL means

**Time To Live**

Redis automatically deletes the key after 600 seconds.

Why?

Prediction results are temporary.

Old data shouldn't stay forever.

---

# Flask Application

```python
app = Flask(__name__)
```

Creates the Flask server.

---

# Health Endpoint

```python
@app.route("/health")
```

Returns:

```json
{
    "status":"ok"
}
```

Used for:

- Kubernetes
- Docker
- Load Balancers
- Monitoring

To verify that the application is alive.

---

# Prediction Endpoint

```
POST /predict-async
```

Client sends:

```json
{
    "amount":100,
    "hour":15,
    "num_tx_past_day":5
}
```

---

# Reading JSON

```python
payload = request.get_json() or {}
```

Converts JSON into a Python dictionary.

---

# Feature Extraction

```python
features = [
    float(payload.get("amount",0)),
    int(payload.get("hour",0)),
    int(payload.get("num_tx_past_day",0))
]
```

The model expects numerical input.

Features become:

```
[
100.0,
15,
5
]
```

---

# Creating Task ID

```python
task_id = uuid.uuid4().hex
```

UUID generates a globally unique identifier.

Example:

```
7d7d4b2bba584b70af6ef514a25f1c0d
```

Why?

Every request must have its own identifier.

---

# Background Thread

```python
threading.Thread(
    target=_run_prediction,
    args=(task_id,features),
    daemon=True
).start()
```

Instead of blocking Flask:

```
Flask

↓

Starts Worker

↓

Immediately Returns
```

The worker executes independently.

---

# Worker Function

```python
_run_prediction()
```

Responsible for:

- Running the model
- Saving result

---

# Artificial Delay

```python
time.sleep(0.3)
```

Simulates expensive model inference.

Real models may take:

- 500 ms
- 5 seconds
- 30 seconds

---

# Running Prediction

```python
MODEL.predict(...)
```

Returns

```
0
```

or

```
1
```

Where:

```
0

↓

Not Fraud
```

```
1

↓

Fraud
```

---

# Converting to Integer

```python
is_fraud = int(...)
```

Ensures the prediction is JSON serialisable.

---

# Saving Result in Redis

```python
REDIS.set(
    RESULT_KEY.format(task_id=task_id),
    is_fraud,
    ex=RESULT_TTL_SECONDS
)
```

Breaking it down:

Key

```
result:abc123
```

Value

```
1
```

Expiration

```
600 seconds
```

Redis stores:

```
result:abc123

↓

1
```

---

# Why Expiration?

Without expiration:

Millions of completed tasks accumulate.

Memory usage increases.

Redis eventually fills up.

TTL automatically cleans up old results.

---

# Returning Task ID

```python
return jsonify({
    "task_id":task_id
}),202
```

Notice:

No prediction is returned.

Only:

```json
{
    "task_id":"abc123"
}
```

---

# HTTP Status 202

202 means:

```
Accepted
```

The server accepted the request.

Processing continues in the background.

---

# Result Endpoint

```
GET /result/task_id
```

Purpose:

Retrieve completed prediction.

---

# Reading Redis

```python
value = REDIS.get(
    RESULT_KEY.format(task_id=task_id)
)
```

If Redis contains:

```
result:abc123

↓

1
```

Value becomes

```
"1"
```

---

# Pending Case

If Redis returns

```
None
```

Prediction isn't ready.

Return:

```json
{
    "task_id":"abc123",
    "status":"pending"
}
```

HTTP

```
202
```

---

# Completed Case

If Redis contains

```
1
```

Return

```json
{
    "task_id":"abc123",
    "is_fraud":1
}
```

Status

```
200 OK
```

---

# Full Workflow

```
Client

↓

POST /predict-async

↓

Flask

↓

Generate Task ID

↓

Return Task ID

↓

Background Thread

↓

Run Model

↓

Prediction

↓

Store in Redis

↓

Client Polls

↓

GET /result/task_id

↓

Redis Returns Prediction

↓

Client Receives Result
```

---

# Why Redis Instead of a Python Dictionary?

A Python dictionary:

- Exists only while the process runs.
- Is not shared across multiple application instances.
- Loses data if the application restarts.

Redis:

- Stores data outside the Flask process.
- Can be shared by multiple workers.
- Supports key expiration.
- Is significantly faster than disk-based storage for temporary data.

---

# Why Use Background Threads?

Background threads allow Flask to continue serving new requests while a prediction is running.

Without a background thread:

```
Request 1

↓

Wait

↓

Request 2 waits

↓

Request 3 waits
```

With background threads:

```
Request 1

↓

Worker

Request 2

↓

Worker

Request 3

↓

Worker
```

The API remains responsive.

---

# Limitations of This Approach

Using Python threads is suitable for demonstrations and lightweight workloads.

For production systems, common alternatives include:

- Celery + Redis
- RQ (Redis Queue)
- Dramatiq
- Apache Kafka
- RabbitMQ
- AWS SQS
- Google Pub/Sub

These tools provide durable queues, retries, monitoring, scheduling, and better scalability.

---

# Key Takeaways

- Asynchronous APIs return immediately and perform work in the background.
- A unique `task_id` lets clients retrieve results later.
- Redis is an excellent temporary store for task results.
- TTL automatically removes stale data and prevents memory growth.
- Background threads keep the Flask application responsive.
- `POST /predict-async` starts the task, while `GET /result/<task_id>` lets clients poll for completion.
- This request/worker/result pattern is widely used in production ML serving systems.

# Day 64 Notes: Serving Multiple ML Models Behind a Unified Nginx API Gateway

---

# Introduction

In production Machine Learning systems, it is very common to have **multiple models** serving different business use cases.

For example:

- Fraud Detection Model
- Customer Churn Prediction Model
- Recommendation System
- Sentiment Analysis Model
- Price Prediction Model

Instead of exposing each model on different ports, organizations usually expose **one API endpoint** and use a **Reverse Proxy** to forward requests to the correct model.

That is exactly what this lab demonstrates.

---

# Architecture Overview

```
                Client

                  │

                  │ HTTP Request

                  ▼

         +------------------+
         |      Nginx       |
         | Reverse Proxy    |
         +------------------+

      /fraud/      /churn/      /recommend/

         │             │               │

         ▼             ▼               ▼

+-------------+ +-------------+ +--------------+
| Fraud Model | | Churn Model | | Recommend ML |
| Flask App   | | Flask App   | | Flask App    |
+-------------+ +-------------+ +--------------+
```

The client only knows about

```
localhost:8085
```

The client never directly talks to the Flask applications.

---

# Why use Nginx?

Imagine three Flask applications running on

```
Fraud       :5000
Churn       :5000
Recommend   :5000
```

Although every application listens on port 5000, they are inside different Docker containers.

Docker provides internal networking.

So

```
fraud:5000
```

means

> Container named fraud

Similarly

```
recommend:5000
```

means

> Container named recommend

Nginx forwards requests to these containers.

---

# Reverse Proxy

A Reverse Proxy receives requests from clients and forwards them to backend servers.

Example:

```
Browser

↓

Nginx

↓

Flask
```

The browser never communicates with Flask directly.

Advantages:

- Security
- Load balancing
- SSL termination
- URL routing
- Centralized access
- Hiding backend services

---

# What is Docker Compose?

Docker Compose allows us to run multiple containers using a single YAML file.

Instead of typing

```
docker run ...
docker run ...
docker run ...
```

we simply write

```yaml
services:
```

and define each container.

---

# Existing Services

Initially the project had

```
Fraud

Churn

Nginx
```

The recommendation model already existed on disk but Compose didn't know about it.

Project structure

```
multi-model/

docker-compose.yml

nginx.conf

fraud/

churn/

recommend/
```

---

# Docker Compose Services

Every application becomes one service.

Example

```yaml
services:

  fraud:

  churn:

  recommend:

  nginx:
```

Each service runs inside its own container.

---

# Fraud Service

```yaml
fraud:
  build: ./fraud
  container_name: mm-fraud
```

Explanation

```
build
```

Docker builds an image from

```
./fraud/Dockerfile
```

instead of downloading one.

```
container_name
```

Specifies the actual Docker container name.

Instead of random names like

```
happy_pike
```

we get

```
mm-fraud
```

---

# Churn Service

Exactly the same idea.

```yaml
churn:
  build: ./churn
  container_name: mm-churn
```

---

# Recommendation Service

The task was to add

```yaml
recommend:
  build: ./recommend
  container_name: mm-recommend
```

Now Compose knows another container must be built.

---

# Nginx Service

```yaml
nginx:
```

Runs the reverse proxy.

Example

```yaml
image: nginx:alpine
```

Instead of building an image ourselves, Docker downloads

```
nginx:alpine
```

---

# Port Mapping

```yaml
ports:

- "8085:80"
```

Meaning

```
Host Machine

8085
      │

      ▼

Container

80
```

The client accesses

```
localhost:8085
```

Nginx receives traffic on port

```
80
```

inside the container.

---

# Mounting Configuration

```yaml
volumes:

- ./nginx.conf:/etc/nginx/nginx.conf:ro
```

Meaning

```
Host File

nginx.conf

↓

Mounted Into

/etc/nginx/nginx.conf
```

Whenever the container starts, Nginx uses our configuration.

```
ro
```

means

Read Only.

The container cannot modify the host configuration.

---

# depends_on

Initially

```yaml
depends_on:

- fraud

- churn
```

After adding recommendation

```yaml
depends_on:

- fraud

- churn

- recommend
```

Compose starts backend services before Nginx.

---

# Understanding Nginx Upstreams

Instead of repeatedly writing

```
fraud:5000
```

Nginx lets us create aliases.

Example

```nginx
upstream fraud {

server fraud:5000;

}
```

Now

```
fraud
```

becomes an alias for

```
fraud:5000
```

---

# Why Upstreams?

Without upstream

```nginx
proxy_pass http://fraud:5000;
```

With upstream

```nginx
proxy_pass http://fraud;
```

Cleaner configuration.

Easy to change backend later.

---

# Recommendation Upstream

Added

```nginx
upstream recommend {

server recommend:5000;

}
```

Meaning

```
recommend

↓

recommend container

↓

Port 5000
```

---

# Nginx Location Block

A location block matches URLs.

Example

```nginx
location /fraud/
```

Matches

```
/fraud

/fraud/

/fraud/predict
```

---

# Recommendation Location

```nginx
location /recommend/ {

proxy_pass http://recommend;

}
```

Meaning

```
Incoming Request

/recommend/predict

↓

Nginx

↓

recommend container

↓

Port 5000
```

---

# Proxy Headers

Common configuration

```nginx
proxy_set_header Host $host;

proxy_set_header X-Real-IP $remote_addr;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_set_header X-Forwarded-Proto $scheme;
```

Purpose

Host

Original host name

Example

```
localhost
```

---

## X-Real-IP

Passes client's IP address.

Useful for logging.

---

## X-Forwarded-For

Preserves the chain of proxies.

Useful when multiple proxies exist.

---

## X-Forwarded-Proto

Indicates

```
HTTP

or

HTTPS
```

Useful for redirects.

---

# Docker Networking

One important question

How does

```
recommend
```

become an IP address?

Docker Compose automatically creates a network.

Example

```
compose_network
```

Every service joins this network.

DNS entries become

```
fraud

↓

Container IP

recommend

↓

Container IP
```

Nginx resolves names automatically.

No IP addresses are needed.

---

# Starting Containers

Command

```bash
docker compose up -d
```

Meaning

```
up
```

Create and start services.

```
-d
```

Detached mode.

Runs in background.

---

# Checking Running Containers

```bash
docker compose ps
```

Shows

- Container Name
- Status
- Ports

Expected

```
mm-fraud

mm-churn

mm-recommend

mm-nginx
```

All should be

```
Up
```

---

# Testing APIs

Fraud

```bash
curl -X POST http://localhost:8085/fraud/predict
```

Nginx forwards request

```
↓

Fraud Flask

↓

JSON Response
```

---

Churn

```bash
curl -X POST http://localhost:8085/churn/predict
```

---

Recommendation

```bash
curl -X POST http://localhost:8085/recommend/predict
```

Expected JSON

```json
{
  "service":"recommend",
  "items":[...]
}
```

---

# Complete Request Flow

```
curl

↓

localhost:8085

↓

Nginx

↓

Location Matching

↓

/recommend/

↓

Proxy Pass

↓

recommend container

↓

Flask

↓

JSON

↓

Nginx

↓

Client
```

---

# Why Not Expose Every Flask App?

Instead of

```
localhost:5001

localhost:5002

localhost:5003
```

We expose only

```
localhost:8085
```

Benefits

- Cleaner APIs
- Easier maintenance
- Better security
- One entry point
- Easy SSL configuration
- Easy authentication
- Easy logging

---

# Real Production Example

```
api.company.com

│

├── /login

├── /payments

├── /fraud

├── /recommend

├── /orders

├── /customers

└── /analytics
```

Internally

```
Each route

↓

Different microservice

↓

Different container

↓

Different language

↓

Different team
```

Users never know.

---

# Common Interview Questions

### What is a Reverse Proxy?

A server that receives client requests and forwards them to backend servers while hiding backend implementation details.

---

### Why use Nginx instead of exposing Flask directly?

- Better security
- Better performance
- SSL support
- Load balancing
- Routing
- Caching
- Compression

---

### What does `proxy_pass` do?

It forwards the incoming HTTP request to another server.

---

### What is an upstream?

An alias representing one or more backend servers.

---

### Why use Docker Compose?

To define and manage multiple containers using a single configuration file.

---

### What does `depends_on` do?

It controls startup order, ensuring dependent services start before others. It does **not** guarantee that an application inside the container is fully ready to accept traffic.

---

### Why are all containers listening on port 5000?

Each container has its own isolated network namespace, so every Flask application can safely use port 5000 without conflicting with the others.

---

### How does Nginx find the Flask containers?

Docker Compose creates an internal network with DNS-based service discovery. Service names such as `fraud`, `churn`, and `recommend` resolve automatically to the corresponding container IP addresses.

---

# Key Takeaways

- Docker Compose simplifies running multi-container applications.
- Every ML model can run in its own isolated container.
- Nginx acts as a single API gateway for all backend services.
- `upstream` blocks make backend definitions reusable and easier to maintain.
- `location` blocks route requests based on URL paths.
- Docker's internal DNS allows containers to communicate using service names instead of IP addresses.
- `docker compose up -d` builds and starts the application stack.
- `docker compose ps` verifies the status of running containers.
- `curl` is a simple way to test API endpoints exposed through the reverse proxy.

---

# Summary

In this lab, we extended an existing ML serving platform by integrating a new **Recommendation** model into a Docker Compose application. We updated the Compose configuration to build and run the new service, configured Nginx with an additional upstream and routing rule, launched the complete stack, and verified that all three machine learning models were accessible through a single unified API gateway. This pattern is widely used in production microservice and MLOps environments because it provides a single entry point for clients while allowing backend services to scale and evolve independently.


# Day 65 Notes
# Simulate a Canary Rollout for Model Updates

---

# Introduction

In modern Machine Learning systems, deploying a newly trained model directly to every user is extremely risky.

Imagine your fraud detection model has been retrained.

Although offline testing may show excellent accuracy, you never know how it will behave with **real production traffic**.

If the new model contains bugs or makes incorrect predictions, it could:

- Block legitimate transactions
- Allow fraudulent transactions
- Reduce customer trust
- Cause financial loss

Instead of replacing the old model instantly, organizations perform a **Canary Deployment**.

A Canary Deployment slowly shifts traffic from the old model to the new model while continuously monitoring its health.

If everything looks healthy, traffic gradually increases.

If anything goes wrong, the deployment immediately rolls back.

---

# Why is it called Canary Deployment?

The name comes from coal mining.

Years ago miners carried **canary birds** into coal mines.

Canaries are extremely sensitive to poisonous gases.

If dangerous gas leaked, the canary became sick before the miners noticed.

The miners could safely escape.

The canary acted as an **early warning system**.

Software deployments use exactly the same idea.

Instead of exposing every customer to the new software,

only a tiny percentage sees it first.

If problems appear,

the rollout stops before everyone is affected.

---

# Traditional Deployment vs Canary Deployment

## Traditional Deployment

```
100% Users
      │
      ▼
 Old Version
      │
      ▼
Deploy New Version
      │
      ▼
100% Users use New Version
```

Problem:

If the new version has a bug,

every user experiences it immediately.

---

## Canary Deployment

```
100% Users
      │
      ▼

95% → Old Version
5%  → New Version

↓

70% → Old Version
30% → New Version

↓

0% → Old Version
100% → New Version
```

Only a few users experience problems if something goes wrong.

---

# Real Companies Using Canary Deployments

Many major companies use this strategy.

- Google
- Netflix
- Amazon
- Microsoft
- Meta
- Uber

They rarely deploy software to everyone at once.

Instead they:

Deploy

↓

Monitor

↓

Increase traffic

↓

Monitor again

↓

Increase again

↓

Complete rollout

---

# ML Model Deployment

Suppose we have

```
Model v1
```

Current production model.

Now data scientists train

```
Model v2
```

Offline evaluation looks good.

But production traffic is different.

Therefore we never trust offline metrics alone.

Instead we perform

```
Canary Deployment
```

---

# Traffic Split

Traffic split simply means

"What percentage of users go to each version?"

Example

```
100 requests
```

95%

↓

v1

5%

↓

v2

Out of 100 users

95 use old model

5 use new model

---

# Traffic Weight

Weights define routing probability.

Example

```
v1 = 95%

v2 = 5%
```

Whenever a request arrives

Random routing decides

```
95%
```

Send to v1

or

```
5%
```

Send to v2

---

# Our Project

We have

```
canary_deploy.py
```

This file simulates a deployment.

No Kubernetes.

No Docker.

No Network.

Everything happens using Python.

---

# Project Constants

The program contains

```python
REQUESTS_PER_PHASE = 100
```

Meaning

Every rollout phase receives

100 requests.

There are

3 phases.

Therefore

```
100 × 3 = 300 requests
```

Final output

```
Total requests: 300
```

---

# Simulated Error Rate

The program contains

```python
V2_ERROR_RATE = 0.02
```

Meaning

The new model has

2%

probability of failing.

Notice

This is only a simulation.

Real deployments would calculate errors from

- HTTP failures
- Prediction failures
- Latency
- Timeouts
- Business metrics

---

# Random Simulation

The code uses

```python
random.Random(seed=42)
```

Why?

Without a seed

Every execution gives different numbers.

With a seed

Every execution produces identical results.

Example

```
Run 1

Phase 3

3% errors
```

Run 2

Same result.

This makes testing reproducible.

---

# CanaryDeployer Class

The class controls deployment.

```
CanaryDeployer
```

Responsibilities

- store weights
- promote deployment
- rollback deployment
- send requests

---

# Initial State

Initially

```python
self.v1_weight = 1.0
```

Means

```
100%
```

traffic goes to v1.

And

```python
self.v2_weight = 0.0
```

Means

No traffic reaches v2.

---

# Promotion

Promotion means

Increase traffic to the new model.

Our rollout consists of

---

## Phase 1

```
95%

↓

v1

5%

↓

v2
```

Purpose

Very small exposure.

Only a few users test the new model.

---

## Phase 2

```
70%

↓

v1

30%

↓

v2
```

Purpose

Gain more confidence.

If monitoring still looks healthy,

continue.

Notice

v1 still receives more traffic than v2.

---

## Phase 3

```
0%

↓

v1

100%

↓

v2
```

Purpose

Full deployment.

The old model is no longer serving traffic.

---

# promote()

Our implementation

```python
def promote(self):
    self.phase += 1

    if self.phase == 1:
        self.v1_weight = 0.95
        self.v2_weight = 0.05

    elif self.phase == 2:
        self.v1_weight = 0.70
        self.v2_weight = 0.30

    elif self.phase == 3:
        self.v1_weight = 0.00
        self.v2_weight = 1.00

    return self.v1_weight, self.v2_weight
```

---

# Understanding Each Line

```
self.phase += 1
```

Move to the next rollout stage.

---

```
if self.phase == 1
```

Configure first rollout.

---

```
self.v1_weight = 0.95
```

95%

traffic

↓

old model.

---

```
self.v2_weight = 0.05
```

5%

traffic

↓

new model.

---

Same logic repeats for

Phase 2

and

Phase 3.

---

# send_requests()

This function simulates

100 incoming users.

Pseudo flow

```
for each request

↓

Random number

↓

Route request

↓

v1 or v2

↓

If v2

↓

Randomly generate error

↓

Count errors
```

---

# Routing Logic

```python
if random < v1_weight
```

Suppose

```
Random = 0.31

Weight = 0.95
```

Since

```
0.31 < 0.95
```

Request goes to

v1.

Another example

```
Random = 0.98

Weight = 0.95
```

Now

```
0.98 > 0.95
```

Request goes to

v2.

---

# Error Simulation

If routed to v2

Program executes

```python
if random < V2_ERROR_RATE
```

Suppose

```
V2_ERROR_RATE

=

0.02
```

Only

2%

of requests become errors.

---

# Error Rate Formula

```
Error Rate

=

Errors

÷

Requests
```

Example

```
Errors

=

3

Requests

=

100

Rate

=

3%
```

---

# Rollback Threshold

Important variable

```python
ROLLBACK_THRESHOLD
```

We changed

```python
1.0
```

to

```python
0.05
```

Meaning

```
5%
```

---

# Why 5%?

Suppose

Threshold

```
100%
```

Even terrible models continue running.

No rollback happens.

Very dangerous.

---

Suppose

Threshold

```
1%
```

Healthy models may randomly exceed

1%.

Deployment stops unnecessarily.

False rollback.

---

Suppose

Threshold

```
5%
```

Healthy rollout continues.

Bad rollout stops.

Balanced choice.

---

# Rollback

If

```
Error Rate

>

Threshold
```

Program executes

```python
rollback()
```

Rollback sets

```python
v1 = 100%

v2 = 0%
```

Old model serves everyone again.

---

# Main Loop

Program runs

```python
for phase in range(1,4)
```

Meaning

```
Phase 1

↓

Phase 2

↓

Phase 3
```

Each phase

- updates weights
- sends requests
- calculates errors
- checks threshold

---

# Promotion Condition

If

```
Error Rate

≤

Threshold
```

Continue.

Next phase starts.

---

# Rollback Condition

If

```
Error Rate

>

Threshold
```

Stop deployment.

Rollback immediately.

---

# Final Output

Healthy rollout

```
Phase 1

↓

Healthy

↓

Phase 2

↓

Healthy

↓

Phase 3

↓

Healthy

↓

OUTCOME: PROMOTED
```

---

# Sample Output

```
Phase 1: v1=95% v2=5%
v1_requests=94
v2_requests=6
v2_error_rate=0.00%

Phase 2: v1=70% v2=30%
v1_requests=70
v2_requests=30
v2_error_rate=0.00%

Phase 3: v1=0% v2=100%
v1_requests=0
v2_requests=100
v2_error_rate=3.00%

OUTCOME: PROMOTED

Total requests:300
```

---

# Why Did Phase 3 Show 3% Instead of 2%?

The configured error probability is

```
2%
```

not exactly

2 errors every 100 requests.

Probability means

some executions may produce

- 1%
- 2%
- 3%
- 4%

Using the fixed random seed (`42`) makes the simulation deterministic, and for this sequence the final phase happens to produce **3%**.

---

# Real Deployment Tools

This simulator demonstrates the same ideas used by:

- Argo Rollouts
- Flagger
- Linkerd
- Istio
- Kubernetes Deployments

These tools automate progressive delivery by shifting traffic, monitoring metrics, and rolling back automatically when thresholds are exceeded.

---

# Interview Questions

## What is Canary Deployment?

A deployment strategy where a small percentage of traffic is routed to the new version first. If the new version remains healthy, traffic is gradually increased until all users are served by the new version.

---

## Why is Canary Deployment better than Blue-Green Deployment?

Canary deployment exposes only a small percentage of users to the new version initially, reducing risk and allowing early detection of issues before a full rollout.

---

## Why do we need a rollback threshold?

The rollback threshold defines the maximum acceptable failure rate. If the new version exceeds this value, the deployment automatically reverts to the stable version to protect users.

---

## Why use `random.Random(seed=42)`?

Using a fixed seed ensures the simulation produces the same sequence of random values every run, making results reproducible and easier to test.

---

## Why is Phase 2 still using 70% v1?

The majority of traffic remains on the stable version while confidence in the new version is built. This limits user impact if unexpected issues appear.

---

## Why do we use 5% as the rollback threshold?

A 5% threshold is a commonly used default because it balances stability and sensitivity. A much higher threshold delays rollback, while a much lower threshold can trigger unnecessary rollbacks due to normal statistical variation.

---

# Key Takeaways

- Canary deployment reduces deployment risk.
- Traffic is shifted gradually rather than all at once.
- Health metrics are checked after each rollout phase.
- Automatic rollback protects users from unhealthy releases.
- A fixed random seed makes simulations reproducible.
- Progressive delivery is a standard practice in modern ML and cloud-native deployments.
- The simulator mirrors the behaviour of real deployment tools such as Argo Rollouts, Flagger, and Linkerd.


# Day 66 Notes: Production Model Serving with Docker Compose

## Topic: Building a Production ML Serving and Observability Stack

---

# 1. Introduction

In a real production machine learning environment, deploying a trained model is not only about creating an API endpoint.

A production ML system requires:

- A model serving layer
- Request handling
- Security and traffic control
- Monitoring
- Metrics collection
- Visualization
- Container orchestration
- Health checks
- Failure recovery

In this task, we are deploying a fraud detection model with the following production architecture:

```
Client
  |
  |
nginx Reverse Proxy
  |
  |
Flask Model API
  |
  +----------------+
  |                |
Redis          Prometheus
(rate limit)   (metrics)
                   |
                   |
                Grafana
             (visualization)
```

Everything runs using Docker Compose.

---

# 2. Components Overview

The complete stack contains six containers:

## 1. model-api

This is the machine learning inference service.

Technology:

- Python
- Flask
- Scikit-learn
- Joblib
- Prometheus Flask Exporter

Responsibilities:

- Load trained model
- Accept prediction requests
- Return fraud prediction
- Provide health endpoint
- Export metrics


Endpoints:

## Health Check

```
GET /health
```

Example:

```json
{
  "status": "ok"
}
```


## Prediction API

```
POST /predict
```

Example request:

```json
{
  "amount":500,
  "hour":12,
  "num_tx_past_day":2
}
```


Example response:

```json
{
  "is_fraud":0
}
```


## Metrics Endpoint

```
GET /metrics
```

Used by Prometheus.

---

# 3. Why We Need Docker Compose

A machine learning production system contains multiple services.

Running everything manually is difficult.

Docker Compose allows us to define:

- Containers
- Networks
- Ports
- Volumes
- Dependencies
- Restart policies


Example:

```
docker-compose.yml
```

describes the complete application.

One command starts everything:

```bash
docker compose up -d
```

---

# 4. Understanding Container Networking

Docker Compose automatically creates an internal network.

Every service can communicate using the service name.

Example:

```
model-api
```

is automatically available as:

```
http://model-api
```

inside the Docker network.

This is important because containers should communicate using service names, not localhost.

Incorrect:

```
localhost:5000
```

Correct:

```
model-api:5000
```

---

# 5. Flask Application

The Flask application is responsible for serving the ML model.

The application flow:

```
Request
  |
  |
Flask
  |
  |
Validate input
  |
  |
Prepare features
  |
  |
Load model prediction
  |
  |
Return JSON response
```

---

# 6. Loading the ML Model

The model is stored as:

```
model.pkl
```

It is loaded during application startup:

```python
MODEL = joblib.load("/app/model.pkl")
```

This avoids loading the model every request.

Production systems normally load the model once when the application starts.

---

# 7. Redis Rate Limiting

## Why Rate Limiting?

Without protection, one client can send unlimited requests.

Example:

```
Attacker
   |
   |
100000 requests/sec
   |
   |
API crashes
```

To prevent this, we use Redis.

---

## Rate Limit Logic

Each client IP gets a counter.

Example:

```
IP Address:

192.168.1.10

Redis key:

ratelimit:192.168.1.10
```

Every request increases the counter:

```
Request 1
counter = 1

Request 2
counter = 2

Request 100
counter = 100
```

After the limit:

```
Request 101

Response:

429 Too Many Requests
```

---

# 8. Prometheus Monitoring

## What is Prometheus?

Prometheus is a monitoring system.

It collects numerical metrics from applications.

Applications expose metrics:

```
Application
    |
    |
/metrics endpoint
    |
    |
Prometheus
```

Prometheus periodically scrapes metrics.

---

# 9. Prometheus Flask Exporter

The Flask exporter automatically creates metrics.

Example:

```
flask_http_request_total
```

Meaning:

Total HTTP requests received.

Example:

```
flask_http_request_total{method="POST",status="200"} 159
```

Meaning:

159 successful POST requests.


---

# 10. Important Debugging Issue

Initially Prometheus was configured incorrectly.

The Flask application listens on:

```
5000
```

but Prometheus tried:

```
8000
```


Wrong:

```yaml
targets:
  - model-api:8000
```


Correct:

```yaml
targets:
  - model-api:5000
```


Why?

Because Prometheus communicates directly with the container.

It does not use the exposed host port.

---

# 11. nginx Reverse Proxy

## Why nginx?

Users should not directly access the Flask application.

Instead:

```
User

 |

nginx

 |

Flask
```


Benefits:

- Security
- Load balancing
- SSL termination
- Routing
- Traffic management


---

# 12. nginx Configuration Issue

The Flask container listens on:

```
5000
```

but nginx was forwarding to:

```
8000
```


Incorrect:

```nginx
server model-api:8000;
```


Correct:

```nginx
server model-api:5000;
```


---

# 13. Health Checks

Docker health checks verify whether the application is alive.

Example:

```yaml
healthcheck:
  test:
    curl http://localhost:5000/health
```


Docker states:

```
Starting

        |
        |
Healthy
```

A healthy container can receive traffic.

---

# 14. Starting the Application

First stop old containers:

```bash
docker compose down
```


Build and start:

```bash
docker compose up -d --build
```


Check status:

```bash
docker compose ps
```

Expected:

```
model-api       Up (healthy)
prod-redis      Up
prod-nginx      Up
prod-prometheus Up
prod-grafana    Up
prod-traffic    Up
```

---

# 15. Testing the API

Direct Flask test:

```bash
curl http://localhost:5000/metrics
```


Through nginx:

```bash
curl -X POST http://localhost:8085/predict \
-H "Content-Type: application/json" \
-d '{"amount":500,"hour":12,"num_tx_past_day":2}'
```


The second method represents real production traffic.

---

# 16. Prometheus Target Verification

Prometheus API:

```bash
curl http://localhost:9090/api/v1/targets
```


Successful output:

```json
{
"health":"up"
}
```


This confirms:

```
Prometheus
     |
     |
 Flask API
```

communication is working.

---

# 17. Grafana Dashboard

## What is Grafana?

Grafana is a visualization platform.

Prometheus stores data.

Grafana displays data.


Architecture:

```
Prometheus

     |

 Grafana

     |

 Dashboard
```


---

# 18. Creating Request Rate Dashboard

The metric:

```
flask_http_request_total
```

is a counter.

Counters continuously increase.

Example:

```
100 requests

after traffic:

200 requests
```


To calculate requests per second:

we use:

```promql
rate()
```


Query:

```promql
sum(rate(flask_http_request_total[1m]))
```


Explanation:

```
rate()
```

calculates change over time.


```
[1m]
```

means last one minute.


```
sum()
```

combines all request types.

---

# 19. Grafana Validation

Datasource check:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/datasources
```


Dashboard check:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/search?type=dash-db
```


A valid dashboard must contain:

- Dashboard object
- UID
- At least one panel

---

# 20. Complete Production Flow

Final request flow:

```
User
 |
 |
localhost:8085
 |
 |
nginx
 |
 |
model-api:5000
 |
 |
Prediction
 |
 |
Redis checks IP limit
 |
 |
Model predicts
 |
 |
JSON response
```


Monitoring flow:

```
model-api

 |
 |
/metrics

 |
 |
Prometheus

 |
 |
Grafana

 |
 |
Dashboard
```

---

# 21. Common Production Mistakes

## Mistake 1

Using localhost between containers.

Wrong:

```
localhost:5000
```

Correct:

```
model-api:5000
```


---

## Mistake 2

Using host ports internally.

Wrong:

```
model-api:8000
```

Correct:

```
model-api:5000
```


---

## Mistake 3

Importing Prometheus exporter but not enabling it.

Wrong:

```python
from prometheus_flask_exporter import PrometheusMetrics
```

alone.

Correct:

```python
metrics = PrometheusMetrics(app)
```

# Day 67 Notes
# Grafana + Prometheus Monitoring (Complete Lecture Notes)

---

# Introduction

In modern cloud-native environments, applications are no longer monitored by manually checking log files or CPU usage. Instead, applications expose **metrics**, which are numerical values that continuously describe the application's behaviour.

Examples:

- Number of HTTP requests
- CPU utilisation
- Memory usage
- Response time
- Database connections
- Prediction accuracy of an ML model
- Model inference latency
- Data drift score

These metrics are collected, stored, queried, and visualised using monitoring tools.

The most common open-source monitoring stack is:

```
Application
      │
      ▼
Prometheus
      │
      ▼
Grafana
```

---

# Architecture of This Lab

Our monitoring stack consists of three containers.

```
                Docker Network
──────────────────────────────────────────

+-------------------+
| metric-emitter    |
| Flask Application |
| Port : 5000       |
| /metrics          |
+-------------------+
          │
          │ scrape every 5 sec
          ▼

+-------------------+
| Prometheus        |
| Port : 9090       |
| Stores metrics    |
+-------------------+
          │
          │ query metrics
          ▼

+-------------------+
| Grafana           |
| Port : 3000       |
| Dashboard UI      |
+-------------------+

──────────────────────────────────────────
```

Each service has a different responsibility.

---

# What is Prometheus?

Prometheus is an **open-source monitoring system**.

Its main job is to collect metrics from applications.

Prometheus continuously asks applications:

```
Give me your latest metrics.
```

Applications expose metrics through an HTTP endpoint.

Usually:

```
/metrics
```

Example:

```
http://application:5000/metrics
```

Prometheus periodically downloads this page.

This process is called:

```
Scraping
```

---

# What Does Scraping Mean?

Scraping means:

> Collecting metrics from an application at regular intervals.

Example:

```
Every 5 seconds

Prometheus
     │
     │ GET /metrics
     ▼

Application

Returns:

prediction_accuracy 0.96
cpu_usage 40
memory_usage 1024
```

Prometheus saves these values into its own database.

---

# What is a Metric?

A metric is simply a numerical value.

Example:

```
prediction_accuracy 0.94

```

Meaning:

Current model accuracy is 94%.

Example:

```
cpu_usage 61
```

Meaning:

CPU usage is 61%.

Example:

```
requests_total 2000
```

Meaning:

Application has handled 2000 requests.

---

# Metric Types

Prometheus supports several metric types.

## Counter

Only increases.

Example:

```
http_requests_total
```

Values:

```
1
2
3
4
5
```

Never decreases.

---

## Gauge

Can increase or decrease.

Example:

```
prediction_accuracy

0.92
0.95
0.91
0.96
```

CPU usage is also a gauge.

---

## Histogram

Measures distributions.

Example:

```
Request duration

10ms
15ms
20ms
```

Useful for latency calculations.

---

## Summary

Similar to histogram but calculates quantiles.

---

# The Flask Metric Emitter

The lab contains a Flask application.

Its only purpose is exposing metrics.

```
metric-emitter
```

Available metrics include:

```
prediction_accuracy

flask_http_request_total

data_drift_score

model_inference_duration_seconds
```

Every five seconds a background thread updates values.

This allows dashboards to display live changes.

---

# What is PromQL?

PromQL stands for

```
Prometheus Query Language
```

It is SQL for Prometheus metrics.

Instead of querying tables,

you query metrics.

Example:

```
prediction_accuracy
```

Returns

```
0.95
```

Example:

```
flask_http_request_total
```

Returns

```
145
```

---

# Examples of PromQL

## View current metric

```
prediction_accuracy
```

---

## HTTP Requests

```
flask_http_request_total
```

---

## Average

```
avg(prediction_accuracy)
```

---

## Maximum

```
max(prediction_accuracy)
```

---

## Minimum

```
min(prediction_accuracy)
```

---

## Rate

```
rate(flask_http_request_total[5m])
```

Shows requests per second.

---

# What is Grafana?

Grafana is a visualisation platform.

It does **not** collect metrics.

Instead it asks another system for data.

Example:

```
Grafana

asks

Prometheus

for metrics.
```

Grafana converts numbers into:

- Graphs
- Tables
- Gauges
- Pie Charts
- Heatmaps
- Alerts

---

# Relationship Between Grafana and Prometheus

```
Application

      │

      ▼

Prometheus

stores metrics

      │

      ▼

Grafana

visualises metrics
```

Prometheus stores.

Grafana displays.

---

# Why Does Grafana Need a Data Source?

Grafana cannot magically know where metrics are stored.

We must tell Grafana:

```
Where should I get my data?
```

This configuration is called:

```
Data Source
```

Without a data source:

```
No dashboards

No queries

No graphs
```

---

# Data Source in This Lab

Type:

```
Prometheus
```

URL:

```
http://prometheus:9090
```

---

# Why NOT localhost?

This is one of the most important Docker concepts.

Suppose three containers exist.

```
Container A

Container B

Container C
```

Each container has its own network namespace.

Inside Grafana:

```
localhost
```

means

```
Grafana container
```

NOT

```
Prometheus
```

Therefore

```
http://localhost:9090
```

tries connecting to

Grafana itself.

Prometheus is elsewhere.

---

# Docker Networking

Docker Compose creates an internal network.

Each service receives a DNS name.

Example:

```
services:

grafana

prometheus

metric-emitter
```

Docker automatically creates hostnames:

```
grafana

prometheus

metric-emitter
```

Therefore

Grafana can access

```
http://prometheus:9090
```

without knowing its IP.

Docker resolves

```
prometheus
```

into the container IP.

---

# Why Service Names Are Better

IPs change.

Containers restart.

New IP assigned.

Service names remain the same.

Therefore always use

```
http://prometheus:9090
```

instead of

```
http://172.x.x.x:9090
```

---

# Adding a Data Source

Open Grafana.

Navigate

```
Connections

↓

Data Sources

↓

Add Data Source
```

Choose

```
Prometheus
```

URL

```
http://prometheus:9090
```

Save & Test.

---

# What Happens When You Click Save & Test?

Grafana sends an HTTP request.

```
Grafana

↓

Prometheus

↓

API Response
```

If successful:

```
Data source is working
```

Otherwise

```
Connection failed
```

---

# Grafana Health API

Grafana exposes REST APIs.

Example

```
GET

/api/datasources
```

Lists every configured data source.

---

Health endpoint

```
/api/datasources/uid/<uid>/health
```

Example response

```json
{
  "status":"OK"
}
```

Meaning

Grafana successfully communicated with Prometheus.

---

# Dashboard

A dashboard is a collection of panels.

Example

```
+------------------------+

CPU Usage

+------------------------+

Memory

+------------------------+

Latency

+------------------------+

Accuracy

+------------------------+
```

Each rectangle is a panel.

---

# Panel

A panel contains

- Query
- Visualisation
- Title

Example

Title

```
Prediction Accuracy
```

Query

```
prediction_accuracy
```

Visualisation

```
Time Series
```

---

# Why Save the Dashboard?

Unsaved dashboards disappear.

Saving stores

- Layout
- Panels
- Queries
- Titles

inside Grafana.

---

# API Verification

List data sources

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/datasources
```

---

Health

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/datasources/uid/<uid>/health
```

---

List dashboards

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/search
```

---

Dashboard JSON

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/dashboards/uid/<uid>
```

Useful for checking saved panels and PromQL queries.

---

# Common PromQL Queries

Prediction accuracy

```promql
prediction_accuracy
```

HTTP requests

```promql
flask_http_request_total
```

Inference latency

```promql
model_inference_duration_seconds
```

Data drift

```promql
data_drift_score
```

Request rate

```promql
rate(flask_http_request_total[5m])
```

Average accuracy

```promql
avg(prediction_accuracy)
```

Maximum accuracy

```promql
max(prediction_accuracy)
```

---

# Common Mistakes

## Using localhost

Wrong

```
http://localhost:9090
```

Correct

```
http://prometheus:9090
```

---

## Forgetting Save Dashboard

Creating a panel is not enough.

Always click

```
Save Dashboard
```

---

## Empty Query

Wrong

```

```

Correct

```promql
prediction_accuracy
```

---

## Wrong Data Source

Panel must use

```
Prometheus
```

---

## Prometheus Not Running

Always verify

```bash
docker ps
```

---

# Complete Flow

```
Flask App

↓

Exposes /metrics

↓

Prometheus scrapes metrics

↓

Prometheus stores metrics

↓

Grafana queries Prometheus

↓

Prometheus returns values

↓

Grafana displays graphs

↓

User monitors application health
```

---

# Commands Used

List data sources

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/datasources
```

Health check

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/datasources/uid/<uid>/health
```

List dashboards

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/search
```

Retrieve dashboard JSON

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/dashboards/uid/<uid>
```

---

# Interview Questions

### 1. What is Prometheus?

An open-source monitoring system that collects, stores, and queries time-series metrics from applications.

---

### 2. What is Grafana?

A visualisation platform that displays metrics from data sources such as Prometheus, InfluxDB, Elasticsearch, Loki, and many others.

---

### 3. Does Grafana store metrics?

No.

Grafana only visualises data. Storage is handled by systems like Prometheus.

---

### 4. What is scraping?

The process where Prometheus periodically requests metrics from an application's `/metrics` endpoint.

---

### 5. What is PromQL?

PromQL (Prometheus Query Language) is used to query and analyse metrics stored in Prometheus.

---

### 6. Why can't Grafana use localhost to reach Prometheus?

Because `localhost` inside the Grafana container refers to the Grafana container itself. Containers communicate using Docker networking and service names such as `http://prometheus:9090`.

---

### 7. What is a dashboard?

A collection of one or more panels used to visualise related metrics.

---

### 8. What is a panel?

A single visualisation (graph, table, gauge, etc.) backed by a PromQL query.

---

### 9. Why are Docker service names preferred over IP addresses?

Service names are stable and automatically resolved by Docker DNS, whereas container IP addresses can change when containers restart.

---

### 10. What does the Grafana data source health check verify?

It confirms that Grafana can successfully communicate with the configured Prometheus server.

---

# Summary

In this lab, we configured Grafana to connect to Prometheus using the Docker Compose service name `http://prometheus:9090`, verified connectivity through Grafana's health API, and created a dashboard with a PromQL-based panel displaying live metrics. This completed the monitoring pipeline from the Flask metric emitter through Prometheus to Grafana, demonstrating how modern cloud-native monitoring systems collect, store, query, and visualise application metrics.

# Day 68 Notes
# Grafana Time-Series Panel for Prediction Accuracy

---

# Introduction

In modern Machine Learning Operations (MLOps), training a model is only one part of the lifecycle. Once a model is deployed into production, it must be continuously monitored to ensure it continues to perform well.

One of the most important metrics to monitor is **prediction accuracy**.

If prediction accuracy starts decreasing over time, it could indicate:

- Data drift
- Concept drift
- Poor model performance
- Bad feature engineering
- Model degradation
- Incorrect deployments

To monitor these metrics visually, we commonly use:

- Flask (exports metrics)
- Prometheus (collects metrics)
- Grafana (visualizes metrics)

This lab demonstrates how these three tools work together.

---

# Architecture

```
ML Application
      │
      │ exposes metrics
      ▼
 Flask Metrics Endpoint
      │
      │ /metrics
      ▼
 Prometheus
      │
      │ scrapes metrics every few seconds
      ▼
 Time-Series Database
      │
      ▼
 Grafana
      │
      ▼
 Dashboard
```

---

# What is Grafana?

Grafana is an open-source visualization platform.

It connects to many data sources such as:

- Prometheus
- MySQL
- PostgreSQL
- Loki
- Elasticsearch
- InfluxDB
- CloudWatch
- Azure Monitor

Grafana itself does **not** store metrics.

Instead, it queries another system (called a datasource) and displays the results.

Think of Grafana as the frontend and Prometheus as the backend.

---

# What is Prometheus?

Prometheus is a monitoring and alerting toolkit.

Its job is to:

- Scrape metrics
- Store metrics
- Allow querying through PromQL
- Send alerts

Prometheus continuously collects data from applications.

Example:

```
Application

prediction_accuracy 0.94

↓

Prometheus stores

Time                  Value
------------------------------
12:00                 0.94
12:01                 0.95
12:02                 0.93
12:03                 0.96
```

Grafana simply reads this stored data.

---

# What is a Metric?

A metric is simply a numerical measurement.

Examples:

```
CPU Usage

RAM Usage

Disk Space

Prediction Accuracy

Response Time

Request Count

Error Count
```

Metrics change over time.

That is why Grafana uses time-series graphs.

---

# Why Prediction Accuracy?

Suppose a fraud detection model initially performs very well.

```
Accuracy

98%
97%
99%
98%
```

After a month:

```
89%
87%
86%
84%
```

The graph immediately tells us:

Something is wrong.

Without monitoring, we might never notice.

---

# Gauge Metric

In this lab:

```
prediction_accuracy
```

is a **Gauge**.

A Gauge is a metric whose value can both increase and decrease.

Examples:

```
Temperature

CPU Usage

Memory Usage

Accuracy

Humidity
```

Unlike counters, gauges are not cumulative.

---

# Counter vs Gauge

Counter:

```
0

1

2

3

4

5

6

```

Always increases.

Example:

```
HTTP Requests

Errors

Jobs Completed
```

---

Gauge:

```
90%

92%

88%

95%

91%

```

Can move both upward and downward.

Prediction accuracy is naturally a Gauge.

---

# Metric Exposed by Flask

Flask exposes metrics like:

```
prediction_accuracy 0.96
```

through:

```
/metrics
```

Example:

```
http://server:5000/metrics
```

Prometheus visits this endpoint periodically.

---

# Prometheus Scraping

Prometheus configuration contains:

```yaml
scrape_configs:

- job_name: flask

  static_configs:

  - targets:
      - flask:5000
```

Every scrape:

```
Flask

↓

Prometheus

↓

Database
```

This happens automatically.

---

# Datasource in Grafana

A datasource tells Grafana where data comes from.

Examples:

```
Prometheus

MySQL

Loki

InfluxDB
```

In this lab:

```
Datasource = Prometheus
```

It is already configured.

So no manual setup is required.

---

# Dashboard

A dashboard is a collection of panels.

Example:

```
Dashboard

+---------------------+

CPU Usage

+---------------------+

Memory

+---------------------+

Prediction Accuracy

+---------------------+

Network

+---------------------+
```

Each rectangle is a panel.

---

# Panel

A panel is an individual visualization.

Examples:

- Time Series
- Gauge
- Stat
- Table
- Pie Chart
- Bar Gauge
- Heatmap

Our task requires a **Time Series** panel.

---

# Why Time Series?

Prediction accuracy changes with time.

Example:

```
Time

12:00

12:05

12:10

12:15

↓

Accuracy

95

94

96

97
```

A Time Series chart is ideal because the X-axis is time and the Y-axis is the metric value.

---

# Other Visualization Types

## Stat

Shows one value.

```
95%
```

No historical trend.

---

## Gauge

Looks like a speedometer.

```
      95%

|------●----|
```

Shows current state only.

---

## Table

Displays rows.

```
Time

Accuracy
```

No graph.

---

## Time Series

Displays change over time.

```
98 ──────╮

97 ────╮ │

96 ──╮ │ │

95 ╮ │ │ │

──────────────►
```

Perfect for monitoring.

---

# PromQL

Prometheus uses its own query language called PromQL.

Example:

```promql
prediction_accuracy
```

This returns every recorded value.

---

Other examples:

Average:

```promql
avg(prediction_accuracy)
```

Maximum:

```promql
max(prediction_accuracy)
```

Minimum:

```promql
min(prediction_accuracy)
```

Rate:

```promql
rate(flask_http_request_total[5m])
```

---

# Available Metrics

This lab includes:

```
prediction_accuracy

flask_http_request_total

data_drift_score

model_inference_duration_seconds
```

We only need:

```promql
prediction_accuracy
```

---

# Step-by-Step Lab

## Login

```
http://localhost:3000
```

Username

```
admin
```

Password

```
grafana2026
```

---

## Verify Datasource

Navigate:

```
Connections

↓

Data Sources
```

Ensure Prometheus exists.

---

## Create Dashboard

```
Dashboards

↓

New Dashboard

↓

Add Visualization
```

---

## Select Datasource

Choose:

```
Prometheus
```

---

## Enter Query

```promql
prediction_accuracy
```

---

## Choose Visualization

Select:

```
Time Series
```

Do **not** select:

- Gauge
- Stat
- Table
- Pie Chart

The validator specifically checks for:

```
"type":"timeseries"
```

---

## Apply

Click:

```
Apply
```

---

## Save Dashboard

Click:

```
Save Dashboard
```

Example name:

```
ML Monitoring
```

---

# Validator Checks

The automated checker verifies:

## 1.

Datasource exists.

```
Prometheus
```

---

## 2.

Dashboard exists.

API:

```
GET /api/search?type=dash-db
```

Must return at least one dashboard.

---

## 3.

Dashboard contains a panel.

Specifically:

```
"type":"timeseries"
```

---

## 4.

Panel query contains:

```promql
prediction_accuracy
```

---

# Common Mistakes

## Forgot to Save Dashboard

Dashboard disappears.

Always click:

```
Save Dashboard
```

---

## Forgot Apply

Changes are not committed.

Always click:

```
Apply
```

before saving.

---

## Wrong Visualization

Using:

- Gauge
- Stat
- Table

will fail validation.

---

## Wrong Query

Incorrect:

```promql
prediction
```

Correct:

```promql
prediction_accuracy
```

---

## Wrong Datasource

Selecting another datasource causes no data or validation failure.

Use the pre-provisioned Prometheus datasource.

---

# Real-World Use Cases

Production ML dashboards often monitor:

- Prediction Accuracy
- Precision
- Recall
- F1 Score
- Data Drift
- Feature Drift
- Model Latency
- Request Count
- Error Rate
- GPU Utilization
- CPU Usage
- Memory Usage

Grafana can display all of these on a single dashboard.

---

# Key Takeaways

- Grafana visualizes metrics; it does not store them.
- Prometheus collects and stores metrics from applications.
- Flask exposes metrics through the `/metrics` endpoint.
- `prediction_accuracy` is a **Gauge** metric because it can increase or decrease.
- A **Time Series** panel is the appropriate visualization for metrics that change over time.
- PromQL is the query language used to retrieve metrics from Prometheus.
- The required query for this lab is:

```promql
prediction_accuracy
```

- The dashboard must be saved after clicking **Apply**.
- The validator checks for a dashboard containing a panel of type `timeseries` with a Prometheus query referencing `prediction_accuracy`.

---

# Summary

This lab demonstrates the complete monitoring pipeline for an ML model:

1. The Flask application exposes the `prediction_accuracy` metric.
2. Prometheus periodically scrapes the metric and stores it as time-series data.
3. Grafana connects to Prometheus as a datasource.
4. A dashboard with a **Time Series** panel visualizes how prediction accuracy changes over time.
5. This enables ML engineers to detect model degradation, data drift, and other production issues early through continuous monitoring.

# Day 69 Notes
# Build a Grafana Table Panel for Per-Feature Data Drift

---

# Introduction

In any Machine Learning system deployed in production, monitoring is just as important as training the model.

Even if a model had 99% accuracy during training, its performance can degrade over time because the incoming data changes.

This phenomenon is called **Data Drift**.

A common question that ML engineers ask is:

> **"Which feature has changed compared to the training data?"**

Instead of looking at one feature at a time, we create a **Grafana Table Panel** that lists every feature alongside its drift score.

This gives engineers an immediate overview of the health of every feature.

---

# What is Data Drift?

Data Drift means the statistical distribution of input data has changed.

Example:

Suppose a fraud detection model was trained using transactions like:

| Feature | Training Average |
|----------|-----------------|
| amount | \$120 |
| hour | 2 PM |
| num_tx_past_day | 5 |

Months later, production traffic becomes:

| Feature | Production Average |
|----------|-------------------|
| amount | \$480 |
| hour | 11 PM |
| num_tx_past_day | 19 |

Now the production data is very different from training data.

This is **data drift**.

Even if the model code never changes, predictions may become worse.

---

# Why Monitor Per Feature?

Suppose only one feature changes.

Example:

```
amount → High drift

hour → Normal

num_tx_past_day → Normal
```

Without per-feature monitoring you only know:

```
Overall drift = High
```

But you don't know **why**.

With per-feature monitoring you instantly know:

```
The amount feature is responsible.
```

This helps data scientists investigate much faster.

---

# Understanding the Metric

The Flask application exports a Prometheus metric:

```text
data_drift_score
```

Unlike a normal metric, this metric has labels.

Example:

```text
data_drift_score{column="amount"}
```

Another one:

```text
data_drift_score{column="hour"}
```

Another:

```text
data_drift_score{column="num_tx_past_day"}
```

Notice something.

The metric name is the same.

Only the label changes.

---

# Understanding Labels

Prometheus labels are key-value pairs.

Example:

```text
column="amount"
```

Here

Key

```
column
```

Value

```
amount
```

Another series

```
column="hour"
```

Another

```
column="num_tx_past_day"
```

These labels allow Prometheus to store multiple related time series under one metric name.

Internally Prometheus stores:

```
Metric Name:
data_drift_score

Series 1
column=amount

Series 2
column=hour

Series 3
column=num_tx_past_day
```

This is much cleaner than creating metrics like:

```
amount_drift
hour_drift
num_tx_past_day_drift
```

---

# What Does Grafana Receive?

Prometheus returns something similar to:

```
data_drift_score{column="amount"} 0.39

data_drift_score{column="hour"} 0.07

data_drift_score{column="num_tx_past_day"} 0.15
```

Grafana receives three independent series.

Each series has

- Metric name
- Labels
- Timestamp
- Value

---

# Why Use a Table Panel?

Grafana supports many visualization types.

Examples:

- Time Series
- Bar Chart
- Pie Chart
- Gauge
- Heatmap
- Stat
- Table

For data drift we care about comparing features.

A table is ideal because every feature gets one row.

Example

| Feature | Drift |
|----------|------:|
| amount | 0.39 |
| hour | 0.07 |
| num_tx_past_day | 0.15 |

Much easier to scan than multiple graphs.

---

# Dashboard Creation

Create a new dashboard.

Click

```
Dashboards

↓

New Dashboard
```

Then

```
Add Visualization
```

Choose

```
Prometheus
```

because Prometheus stores the metrics.

---

# Writing the Query

The query is very simple.

```prometheus
data_drift_score
```

No aggregation is needed.

No filtering is needed.

No functions are required.

Prometheus automatically returns all series belonging to this metric.

---

# Why Not Use sum()?

Imagine using

```prometheus
sum(data_drift_score)
```

Output:

```
0.61
```

Now you've lost all feature information.

You only know the total.

You cannot identify which feature drifted.

Therefore avoid aggregation.

---

# Why Not Use avg()?

Example

```
amount = 0.5

hour = 0.1

transactions = 0.2
```

Average

```
0.26
```

Again,

you lose feature-level visibility.

---

# Why Use an Instant Query?

Prometheus stores history.

Suppose it collected

```
10:20

10:21

10:22

10:23

10:24
```

A Range query returns

```
10:20

10:21

10:22

10:23

10:24
```

Your table becomes

| Time | Value |
|------|------:|
|10:20|0.10|
|10:21|0.12|
|10:22|0.14|
|10:23|0.18|

This is not what we want.

We only want the latest value.

Instant Query returns

```
Latest only
```

Example

```
amount

0.39
```

This is perfect for a table.

---

# Time Series vs Instant

Range Query

```
amount

10:20

10:21

10:22

10:23
```

Instant Query

```
amount

0.39
```

The task specifically wants one row per feature.

Therefore

```
Instant Query
```

is the correct choice.

---

# Understanding Transformations

Grafana transformations reshape data.

They do not change Prometheus.

They only change how data appears.

Example

Prometheus returns

```
Series

amount

0.39
```

Transformation

```
Labels to Fields
```

becomes

| column | Value |
|---------|------:|
| amount |0.39|

Another series

```
hour
```

becomes

| column | Value |
|---------|------:|
| hour |0.08|

Combined result

| column | Value |
|---------|------:|
| amount |0.39|
| hour |0.08|
| num_tx_past_day |0.15|

Exactly what we need.

---

# Labels to Fields

This transformation converts labels into columns.

Input

```
column=amount

value=0.39
```

Output

| column | Value |
|---------|------:|
| amount |0.39|

Without it, Grafana simply displays

```
{column="amount"}
```

which is harder to read.

---

# Panel Title

A descriptive title is recommended.

Example

```
Feature Data Drift
```

Anyone opening the dashboard immediately understands its purpose.

---

# Saving the Dashboard

Always save your work.

Click

```
Apply
```

Then

```
Save Dashboard
```

Provide a meaningful dashboard name.

Example

```
Fraud Detection Monitoring
```

---

# Validation Requirements Explained

The lab validates several conditions.

## Requirement 1

```
GET /api/search?type=dash-db
```

This checks whether at least one user-created dashboard exists.

---

## Requirement 2

At least one panel must have

```
type = table
```

This confirms you created a Table visualization.

---

## Requirement 3

The panel query must reference

```prometheus
data_drift_score
```

This ensures the correct metric is being visualized.

---

## Requirement 4

The query must return multiple series containing the

```
column
```

label.

Example

```
column=amount

column=hour

column=num_tx_past_day
```

This proves the table can display one row for each feature.

---

# Expected Final Table

| column | Drift Score |
|----------|-----------:|
| amount |0.39|
| hour |0.08|
| num_tx_past_day |0.15|

Every row corresponds to one feature.

---

# Common Mistakes

## Mistake 1

Using Range Query.

Result

```
Many timestamps
```

instead of one latest value.

---

## Mistake 2

Using

```prometheus
sum(data_drift_score)
```

Feature information disappears.

---

## Mistake 3

Using

```prometheus
avg(data_drift_score)
```

All features are merged.

---

## Mistake 4

Creating a Time Series panel instead of a Table panel.

The lab specifically expects a Table visualization.

---

## Mistake 5

Forgetting to save the dashboard.

Unsaved dashboards are not returned by the Grafana API.

---

# Real-World Importance

Large organizations may have hundreds of input features.

Examples include:

- Banks
- Insurance companies
- E-commerce platforms
- Healthcare systems
- Recommendation engines

Without a table, engineers would have to inspect each feature separately.

A Table Panel provides a single overview showing every feature and its latest drift score.

This makes identifying problematic inputs much faster.

---

# Key Takeaways

- Data drift measures how production data differs from training data.
- Prometheus stores one time series per feature using labels.
- The `column` label identifies the feature name.
- Query `data_drift_score` directly without aggregation.
- Use an **Instant Query** to retrieve only the latest values.
- Use a **Table** visualization for easy comparison.
- Apply the **Labels to fields** transformation if needed to expose labels as columns.
- Save the dashboard after applying changes.
- The final table should display one row per feature and its latest drift score, enabling quick identification of which input features have drifted.

# Day 70: Enforcing ML Model Accuracy Gates with Evidently and Grafana

## Introduction

In a production machine learning system, deploying a model is not the end of the job. A model that performs well during development can degrade after deployment because of:

- Changes in incoming data
- Data quality problems
- Changes in user behavior
- Feature distribution changes
- Model drift

For this reason, ML platforms need continuous quality checks.

In this task, we implement quality gates for a fraud-detection model using two different layers:

1. **Evidently Quality Gates**
   - Used before deployment.
   - Acts as a CI/CD gate.
   - Prevents a bad model or bad data batch from moving to production.

2. **Grafana Alerting**
   - Used after deployment.
   - Monitors live model performance.
   - Alerts engineers when production accuracy decreases.

The important idea is:

> The same accuracy requirement is enforced before deployment and during production.

The accuracy threshold for this system is:

```
Accuracy > 0.80
```

---

# Part 1: Evidently Test Suite

## What is Evidently?

Evidently is an open-source ML monitoring and testing framework.

It helps teams monitor:

- Data quality
- Data drift
- Model performance
- Classification metrics
- Regression metrics

Evidently can generate reports and also run tests.

A normal metric answers:

"How good is my model?"

A test answers:

"Is this value acceptable?"

Example:

Metric:

```
Accuracy = 0.83
```

Test:

```
Accuracy must be greater than 0.80
```

Result:

```
SUCCESS
```

---

# Existing Project Structure

The monitoring project already contains:

```
/root/code/monitoring
|
├── tests
│   ├── test_suite.py
│   ├── current.csv
│   └── test_results.json
|
├── workspace
|
└── drift
    └── drift_scorer.py
```

The important files:

## current.csv

This is the production batch.

It contains:

- Features
- Target column:

```
is_fraud
```

- Model prediction:

```
prediction
```

---

## test_suite.py

The file already contains:

- Dataset loading
- Data definition
- Classification mapping
- Evidently report execution
- Workspace publishing

Only the quality gates are missing.

---

# Understanding Evidently Dataset Definition

The model is a binary classification model.

Example:

Actual value:

```
is_fraud = 1
```

Prediction:

```
prediction = 1
```

The model is correct.

Evidently needs to know:

- Which column is the target
- Which column contains predictions

This is configured using:

```python
BinaryClassification(
    target="is_fraud",
    prediction_labels="prediction"
)
```

---

# Understanding Test Metrics

The two required gates are:

1. Missing value gate
2. Accuracy gate

---

# Gate 1: Missing Values Check

## Problem

Bad input data can break model predictions.

Examples:

```
customer_age = NULL
transaction_amount = NULL
```

Too many missing values indicate a data pipeline problem.

The requirement:

```
Fail when missing values >= 10
```

Equivalent condition:

```
Missing values < 10
```

---

## Evidently Implementation

```python
DatasetMissingValueCount(
    tests=[lt(10)]
)
```

Explanation:

```
DatasetMissingValueCount
```

calculates the number of missing values.

The test:

```
lt(10)
```

means:

```
less than 10
```

If:

```
missing values = 5
```

Result:

```
SUCCESS
```

If:

```
missing values = 15
```

Result:

```
FAIL
```

---

# Gate 2: Accuracy Check

## Problem

A model can continue running but become inaccurate.

Example:

Before deployment:

```
Accuracy = 0.87
```

After deployment:

```
Accuracy = 0.72
```

The model should not be considered healthy.

Requirement:

```
Accuracy must be greater than 0.80
```

---

## Evidently Implementation

```python
Accuracy(
    tests=[gt(0.80)]
)
```

Explanation:

```
Accuracy
```

calculates classification accuracy.

The condition:

```
gt(0.80)
```

means:

```
greater than 0.80
```

Examples:

```
Accuracy = 0.85

SUCCESS
```

```
Accuracy = 0.75

FAIL
```

---

# Final METRICS Configuration

The completed section:

```python
METRICS = []

METRICS.append(
    DatasetMissingValueCount(
        tests=[lt(10)]
    )
)

METRICS.append(
    Accuracy(
        tests=[gt(0.80)]
    )
)
```

---

# Running the Evidently Test Suite

Execute:

```bash
python3 /root/code/monitoring/tests/test_suite.py
```

The script:

1. Reads the production batch
2. Creates an Evidently dataset
3. Runs the tests
4. Writes results
5. Publishes the report

---

# Output File

After successful execution:

```
/root/code/monitoring/tests/test_results.json
```

is created.

Example:

```json
{
  "tests": [
    {
      "name": "DatasetMissingValueCount",
      "status": "SUCCESS"
    },
    {
      "name": "Accuracy",
      "status": "SUCCESS"
    }
  ]
}
```

---

# Evidently UI Verification

The script publishes the result to:

```
Evidently Workspace
```

Open:

```
http://<host>:8000
```

Navigate:

```
fraud-detector quality gates

        |

     Reports
```

A snapshot should be visible.

The UI allows reviewers to inspect:

- Metrics
- Test results
- Pass/fail status

without opening code.

---

# Part 2: Grafana Production Alerting

## Why Grafana?

Evidently protects deployment.

But after deployment, production can still degrade.

Example:

Morning:

```
Accuracy = 0.86
```

After several hours:

```
Accuracy = 0.78
```

The model is now unhealthy.

Grafana continuously monitors this.

---

# Available Monitoring Metrics

The monitoring stack exposes:

## Prediction Accuracy

Metric:

```
prediction_accuracy
```

This is a gauge metric.

Example:

```
prediction_accuracy 0.85
```

---

## Other Metrics

Data drift:

```
data_drift_score
```

Drift percentage:

```
evidently_drift_share
```

API metrics:

```
flask_http_request_total
```

Latency:

```
model_inference_duration_seconds
```

---

# Why avg_over_time?

Raw accuracy values can fluctuate.

Example:

```
0.84
0.86
0.83
0.85
```

A moving average gives a smoother signal.

Prometheus function:

```promql
avg_over_time()
```

The required query:

```promql
avg_over_time(prediction_accuracy[1m])
```

Meaning:

"Calculate the average prediction accuracy over the last one minute."

---

# Creating Grafana Alert

Open:

```
http://<host>:3000
```

Login:

```
username:
admin

password:
grafana2026
```

---

Navigate:

```
Alerting

    |

Alert rules

    |

New alert rule
```

---

# Alert Name

Example:

```
Prediction Accuracy Alert
```

---

# Query

Datasource:

```
Prometheus
```

Query:

```promql
avg_over_time(prediction_accuracy[1m])
```

---

# Alert Condition

The alert should trigger when:

```
accuracy < 0.80
```

Configure:

```
WHEN QUERY

Is below

0.80
```

Equivalent PromQL logic:

```promql
avg_over_time(prediction_accuracy[1m]) < 0.80
```

---

# Folder and Evaluation

Create/select a folder:

```
ml-alerts
```

Evaluation:

```
Interval:
1m
```

The alert is checked every minute.

---

# Saving the Alert

After saving, verify:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/v1/provisioning/alert-rules
```

The response must contain:

- Alert rule
- prediction_accuracy reference
- threshold 0.80

---

# Complete System Flow

The final architecture:

```
              Developer pushes model
                       |
                       |
                 CI Pipeline
                       |
                       |
              Evidently Test Suite
                       |
          -----------------------------
          |                           |
   Missing values              Accuracy check
      < 10                      > 0.80
          |                           |
          -------- SUCCESS -----------
                       |
                Model Deployment
                       |
                       |
              Production Traffic
                       |
                       |
              Metric Exporter
                       |
                       |
                 Prometheus
                       |
                       |
                  Grafana
                       |
                       |
          avg_over_time accuracy < 0.80
                       |
                       |
                Alert On-call Team
```

---

# Final Validation Checklist

## Evidently

Check:

```bash
ls /root/code/monitoring/tests/test_results.json
```

Expected:

```
File exists
```

Tests:

```
DatasetMissingValueCount SUCCESS

Accuracy SUCCESS
```

---

## Evidently UI

Verify:

```
fraud-detector quality gates

Reports

Published snapshot exists
```

---

## Grafana

Verify:

```bash
curl -u admin:grafana2026 \
http://localhost:3000/api/v1/provisioning/alert-rules
```

Expected:

```
Non-empty array
```

The rule contains:

```
prediction_accuracy
```

and:

```
0.80
```

---

# Key Learning Points

## 1. Metrics are not enough

A metric tells us a value.

A test creates a decision.

Example:

Metric:

```
Accuracy = 0.82
```

Test:

```
Is accuracy acceptable?
```

Answer:

```
SUCCESS
```

---

## 2. Quality gates prevent bad deployments

Evidently stops poor models before production.

---

## 3. Monitoring protects production

Grafana catches problems after deployment.

---

## 4. Same threshold, two locations

The same business rule:

```
Accuracy > 0.80
```

exists in two places:

Before deployment:

```
Evidently CI test
```

After deployment:

```
Grafana alert
```

---

# Conclusion

The fraud-detection ML platform now has a complete production quality-control system.

Evidently ensures that only healthy models are deployed.

Grafana ensures that deployed models continue performing correctly.

Together they provide:

- Automated testing
- Continuous monitoring
- Faster incident detection
- Reliable ML operations

# Day 71 Notes
# Build a 4-Panel Model-Overview Grafana Dashboard

---

# Introduction

Modern Machine Learning systems require continuous monitoring after deployment. A model that performs well during training can gradually degrade in production due to changes in incoming data, infrastructure issues, or application bugs.

Grafana is one of the most widely used visualization platforms for monitoring systems. It allows engineers to visualize metrics collected by Prometheus and quickly identify performance issues.

In this project, we build a **Model Overview Dashboard** containing four essential metrics that help an on-call ML engineer understand the health of a deployed model in just a few seconds.

---

# Architecture

```
Application
      │
      │
      ▼
 Flask Metric Exporter
      │
      ▼
 Prometheus
      │
      ▼
 Grafana Dashboard

Evidently
      │
      ▼
 Drift Scores
      │
      ▼
 Flask Exporter
      │
      ▼
 Prometheus
      │
      ▼
 Grafana
```

---

# Components Used

## 1. Flask Metric Exporter

The Flask application exposes metrics in Prometheus format.

Example metrics:

- HTTP request count
- Model inference latency
- Prediction accuracy
- Feature drift scores

Prometheus periodically scrapes these metrics.

---

## 2. Prometheus

Prometheus is a monitoring database.

Its responsibilities are:

- Scrape metrics
- Store time-series data
- Execute PromQL queries
- Provide data to Grafana

Every metric becomes a time series inside Prometheus.

---

## 3. Grafana

Grafana is only a visualization layer.

Grafana does **not** collect metrics.

Instead, it:

- Connects to Prometheus
- Executes PromQL queries
- Draws graphs
- Creates dashboards
- Supports alerts

---

## 4. Evidently

Evidently monitors machine learning models.

It computes:

- Feature drift
- Target drift
- Prediction drift
- Data quality
- Model quality

In this project Evidently computes **Population Stability Index (PSI)** every few seconds and exports it.

---

# Understanding Each Metric

---

# 1. Request Rate

Metric

```
flask_http_request_total
```

Type

```
Counter
```

A Counter only increases.

Example

```
0
5
20
25
40
```

Since it always increases, we calculate requests per second using `rate()`.

Query

```promql
sum(rate(flask_http_request_total[5m]))
```

Explanation

```
rate()
```

Calculates how quickly the counter increases.

```
[5m]
```

Uses the last five minutes.

```
sum()
```

Adds together all HTTP requests.

Without `sum()` you would get one graph per endpoint.

---

Example

Requests

```
Login API

100
120
150

Predict API

500
550
600
```

Using

```promql
sum(rate(flask_http_request_total[5m]))
```

produces

```
Total Request Rate
```

instead of separate graphs.

---

# Why Time Series?

Request rate changes continuously.

A time-series graph shows

- spikes
- traffic increases
- traffic drops
- outages

which are impossible to understand from a single number.

---

# 2. Model Inference Latency

Metric

```
model_inference_duration_seconds_bucket
```

This is a Histogram.

---

# What is a Histogram?

Suppose inference takes

```
0.02 sec
0.05 sec
0.07 sec
0.10 sec
0.25 sec
0.50 sec
```

Prometheus stores them inside buckets.

Example

```
<=0.05

2 requests

<=0.10

4 requests

<=0.25

5 requests

<=0.50

6 requests
```

Each bucket has an **le** label.

Example

```
le="0.05"

le="0.10"

le="0.25"
```

---

# Why Buckets?

Instead of storing every request individually, Prometheus counts how many requests fall into each latency bucket.

This saves memory while allowing percentile calculations.

---

# Why P95?

Average latency can hide slow requests.

Example

Requests

```
10 ms
10 ms
10 ms
10 ms
900 ms
```

Average

```
188 ms
```

This average doesn't show that one request took almost one second.

P95 means:

95% of requests finished faster than this value.

Only the slowest 5% are excluded.

This is much more useful in production.

---

Query

```promql
histogram_quantile(
  0.95,
  sum by (le)(
    rate(model_inference_duration_seconds_bucket[5m])
  )
)
```

---

Breakdown

### rate()

Converts bucket counters into per-second values.

---

### sum by(le)

Groups buckets while preserving the bucket boundary.

Without

```
by(le)
```

Prometheus cannot compute percentiles.

---

### histogram_quantile()

Calculates

```
P50
P90
P95
P99
```

depending on the first argument.

Example

```
0.95
```

means

```
95th percentile
```

---

Why use a Stat visualization?

Latency is usually monitored as a single important value.

A large number is easier to notice than a graph.

---

# 3. Prediction Accuracy

Metric

```
prediction_accuracy
```

Type

```
Gauge
```

---

# Gauge

Unlike counters,

Gauge values can increase or decrease.

Example

```
0.91
0.92
0.90
0.89
0.93
```

No rate calculation is needed.

Query

```promql
prediction_accuracy
```

---

Why Gauge Visualization?

A Gauge instantly communicates whether the value is within an acceptable range.

Example

```
Green

0.95

Yellow

0.90

Red

0.75
```

It is easy for an operator to interpret.

---

# 4. Feature Drift

Metric

```
data_drift_score
```

Type

```
Gauge
```

Each feature has a label.

Example

```
column="amount"

column="hour"

column="merchant"

column="balance"
```

Prometheus stores

```
amount

0.18

hour

0.07

merchant

0.29
```

---

Query

```promql
data_drift_score
```

---

What is Feature Drift?

Suppose the model was trained on

```
Age

18-35
```

Today the incoming data becomes

```
55-80
```

The feature distribution has changed.

This is called feature drift.

---

Population Stability Index (PSI)

PSI measures how different two distributions are.

Reference data

↓

Production data

↓

Difference

↓

PSI Score

---

Typical Interpretation

| PSI | Meaning |
|------|----------|
| <0.1 | No drift |
| 0.1–0.2 | Moderate drift |
| >0.2 | Significant drift |

---

Why Bar Gauge?

Each feature has its own score.

Bars make it easy to compare features side by side.

Example

```
amount

█████████

0.24

hour

██

0.05

merchant

██████

0.17
```

The tallest bar indicates the feature with the most drift.

---

# PromQL Used

Request Rate

```promql
sum(rate(flask_http_request_total[5m]))
```

Latency

```promql
histogram_quantile(
0.95,
sum by(le)(
rate(model_inference_duration_seconds_bucket[5m])
)
)
```

Prediction Accuracy

```promql
prediction_accuracy
```

Feature Drift

```promql
data_drift_score
```

---

# Visualization Choices

| Metric | Visualization | Reason |
|---------|--------------|--------|
| Request Rate | Time Series | Shows traffic over time |
| Latency | Stat | Displays one important value |
| Prediction Accuracy | Gauge | Easy health indicator |
| Drift Score | Bar Gauge | Compare multiple features |

---

# Dashboard Layout

```
+----------------------+----------------------+
| Request Rate         | P95 Latency          |
+----------------------+----------------------+
| Prediction Accuracy  | Drift by Column      |
+----------------------+----------------------+
```

This layout allows an engineer to understand system health in seconds.

---

# Why These Four Metrics?

Together they answer four critical questions:

### Is traffic reaching the service?

Request Rate

---

### Is the model responding quickly?

P95 Latency

---

### Is the model making correct predictions?

Prediction Accuracy

---

### Has the incoming data changed?

Feature Drift

---

# Advantages of This Dashboard

- Quick health overview
- Detects traffic spikes
- Detects slow inference
- Detects accuracy degradation
- Detects feature drift
- Helps reduce incident response time
- Easy for on-call engineers to monitor

---

# Key Prometheus Concepts Learned

- Counter
- Gauge
- Histogram
- Histogram Buckets
- Labels
- Time Series
- PromQL
- `rate()`
- `sum()`
- `sum by()`
- `histogram_quantile()`

---

# Key Grafana Concepts Learned

- Dashboard
- Panel
- Visualization
- Time Series Panel
- Stat Panel
- Gauge Panel
- Bar Gauge Panel
- Prometheus Data Source
- Query Editor

---

# Key Machine Learning Monitoring Concepts Learned

- Model Monitoring
- Feature Drift
- Population Stability Index (PSI)
- Prediction Accuracy
- Inference Latency
- Request Rate
- Evidently AI Integration

---

# Summary

In this project, we created a comprehensive **Model Overview Dashboard** in Grafana using Prometheus as the data source. The dashboard combines infrastructure metrics and machine learning metrics into a single interface.

The dashboard displays:

- Request Rate (traffic monitoring)
- P95 Inference Latency (performance monitoring)
- Prediction Accuracy (model quality)
- Feature Drift (data quality)

By combining these four signals with multiple visualization types, an on-call engineer can quickly determine whether the deployed ML model is healthy, performant, accurate, and receiving data that matches its training distribution.

# Day 72 Notes
# Grafana Contact Points & Notification Policies (Complete Lecture Notes)

---

# Introduction

Grafana Alerting is used to monitor systems and notify people when something goes wrong.

Creating an alert rule alone **does not send notifications**.

Many beginners think:

```
Alert Rule
     ↓
Notification
```

This is incorrect.

The actual flow is

```
Metrics
   │
   ▼
Prometheus
   │
   ▼
Alert Rule
   │
   ▼
Notification Policy
   │
   ▼
Contact Point
   │
   ▼
Webhook / Slack / Email / PagerDuty
```

Every piece is required.

If even one piece is missing, no notification will ever be sent.

---

# The Alerting Pipeline

Grafana Alerting works in multiple stages.

```
Prometheus
      │
      ▼
Alert Rule evaluates metrics
      │
      ▼
Alert becomes "Firing"
      │
      ▼
Notification Policy checks labels
      │
      ▼
Policy selects Contact Point
      │
      ▼
Contact Point sends notification
```

Without a Notification Policy...

```
Alert Rule
      │
      ▼
Nothing happens
```

Without a Contact Point...

```
Policy
     │
     ▼
Nowhere to send notification
```

---

# Understanding Alert Labels

Every Grafana alert contains labels.

Example

```yaml
alertname: HighCPU
instance: node1
job: node-exporter
severity: high
team: ml
environment: production
```

Labels are simply key-value pairs.

```
severity = high
team = ml
environment = production
```

Notification Policies use these labels to decide where alerts should go.

Think of labels like tags attached to alerts.

---

# Real World Example

Suppose your company has different teams.

```
ML Team
Database Team
Security Team
Networking Team
```

If a database alert occurs,

it should not notify the ML team.

Instead,

Database alerts should go to Database engineers.

Example:

```
team=db
```

Another example

```
severity=critical
```

should page on-call engineers immediately.

Low severity alerts

```
severity=low
```

may only send an email.

This routing is performed by Notification Policies.

---

# Components of Grafana Alerting

There are four major components.

## 1. Data Source

Example

```
Prometheus
Loki
InfluxDB
Graphite
```

The datasource stores metrics.

---

## 2. Alert Rule

The Alert Rule asks

"WHEN should an alert fire?"

Example

```
CPU > 90%
```

or

```
Memory > 80%
```

Example

```
sum(rate(http_requests_total[5m])) > 100
```

The Alert Rule decides

```
Healthy

or

Firing
```

---

## 3. Notification Policy

Notification Policies ask

```
WHO should receive this alert?
```

Example

```
severity=critical
```

↓

```
PagerDuty
```

Example

```
team=backend
```

↓

```
Backend Slack Channel
```

Example

```
environment=dev
```

↓

```
Developer Email
```

---

## 4. Contact Point

A Contact Point answers

```
WHERE should the notification go?
```

Examples

```
Slack

Email

Webhook

Microsoft Teams

Discord

Telegram

PagerDuty

OpsGenie

VictorOps
```

Contact Points only define destinations.

They never decide which alert goes there.

---

# Difference Between Contact Point and Notification Policy

This is one of the most important interview questions.

---

## Contact Point

Stores

```
Destination
```

Example

```
Slack URL

Webhook URL

Email Address
```

---

## Notification Policy

Stores

```
Routing Logic
```

Example

```
severity=critical
```

↓

```
Slack
```

Example

```
team=security
```

↓

```
PagerDuty
```

---

Think of it like postal mail.

Contact Point

```
Address
```

Notification Policy

```
Who receives the letter
```

---

# Why We Need Both

Suppose you create only a Contact Point.

```
Webhook
```

Does Grafana know which alert should use it?

No.

Suppose you create only a Notification Policy.

```
severity=high
```

Does Grafana know where to send alerts?

No.

Therefore,

both are mandatory.

---

# Contact Point Types

Grafana supports many notification integrations.

Examples

```
Webhook

Email

Slack

Microsoft Teams

Discord

Telegram

PagerDuty

OpsGenie

VictorOps

Amazon SNS
```

All are Contact Points.

---

# Webhook

Webhook is one of the simplest notification methods.

Instead of sending an email,

Grafana performs an HTTP request.

Example

```
POST http://webhook-sink:5000/hook
```

The receiving application processes the alert.

---

# Why Companies Use Webhooks

Imagine a company has its own incident platform.

Instead of Slack,

Grafana sends

```
POST
```

to

```
https://company.com/alerts
```

The internal platform

- opens incidents
- creates tickets
- sends SMS
- triggers automation
- updates dashboards

Everything begins from the webhook.

---

# Notification Policy Matching

Suppose an alert contains

```yaml
severity: high
team: ml
environment: production
```

Policy

```
severity=high
```

Matches?

YES

---

Policy

```
team=db
```

Matches?

NO

---

Policy

```
environment=production
```

Matches?

YES

---

Policy

```
severity=critical
```

Matches?

NO

---

# Matchers

Notification Policies use matchers.

Example

```
severity = high
```

Operator

```
=
```

means exact match.

Other operators include

```
!=

=~

!~
```

Example

```
team =~ backend|frontend
```

Matches

```
backend

frontend
```

---

# Notification Policy Tree

Grafana stores policies as a tree.

Example

```
Default Policy
        │
        ├──────── severity=high
        │
        ├──────── team=security
        │
        ├──────── team=db
        │
        └──────── environment=dev
```

Each child is called a route.

---

# Default Policy

Every alert starts here.

```
Default Policy
```

If no child matches,

Grafana uses the default receiver.

---

# Route

A Route is simply a child policy.

Example

```
severity=high
```

↓

```
Webhook
```

---

Another route

```
team=database
```

↓

```
PagerDuty
```

---

# Lab Scenario

Requirement

```
severity=high
```

↓

```
Webhook
```

That's all.

Nothing else is needed.

---

# Contact Point Configuration

Name

```
high-severity-webhook
```

Type

```
Webhook
```

URL

```
http://webhook-sink:5000/hook
```

---

# Notification Policy

Matcher

```
severity = high
```

Receiver

```
high-severity-webhook
```

---

# API Verification

Grafana exposes Provisioning APIs.

Contact Points

```
GET

/api/v1/provisioning/contact-points
```

Example

```json
[
  {
    "name":"high-severity-webhook",
    "type":"webhook",
    "settings":{
      "url":"http://webhook-sink:5000/hook"
    }
  }
]
```

---

Notification Policies

```
GET

/api/v1/provisioning/policies
```

Example

```json
{
  "receiver":"empty",
  "routes":[
    {
      "receiver":"high-severity-webhook",
      "object_matchers":[
        [
          "severity",
          "=",
          "high"
        ]
      ]
    }
  ]
}
```

---

# Why API Verification Matters

The UI may show unsaved changes.

The API always shows the actual configuration stored in Grafana.

Labs usually validate using these APIs.

---

# Common Mistakes

## Forgetting to Save

Most common issue.

Always click

```
Save Policy
```

---

## Wrong Label

Correct

```
severity
```

Incorrect

```
Severity

SEVERITY

priority
```

Labels are case-sensitive.

---

## Wrong Value

Correct

```
high
```

Incorrect

```
High

HIGH
```

---

## Wrong Contact Point Type

Correct

```
Webhook
```

Incorrect

```
Email
```

---

## Wrong URL

Correct

```
http://webhook-sink:5000/hook
```

Incorrect

```
http://localhost:5000

http://webhook

https://webhook-sink
```

---

## No Notification Policy

Even if the Contact Point exists,

nothing will be sent.

---

## No Contact Point

Even if the Notification Policy exists,

there is nowhere to deliver alerts.

---

# Interview Questions

## What is a Contact Point?

A Contact Point defines **where notifications are delivered** (Webhook, Email, Slack, PagerDuty, Teams, etc.).

---

## What is a Notification Policy?

A Notification Policy defines **which alerts are routed to which Contact Point** based on alert labels.

---

## Can an Alert Rule send notifications directly?

No.

Alert Rules only determine **when** an alert fires. Notification Policies and Contact Points determine **how** and **where** notifications are sent.

---

## Why use labels?

Labels allow Grafana to categorize alerts and route them to the appropriate team or destination.

---

## What is a Webhook?

A Webhook is an HTTP endpoint that receives alert notifications as HTTP requests, enabling integration with custom applications or automation workflows.

---

# Key Takeaways

- Alert Rules decide **when** an alert fires.
- Contact Points define **where** notifications are sent.
- Notification Policies define **which** alerts go to **which** Contact Point.
- Labels are used for routing.
- `severity=high` matches alerts labeled with high severity.
- Webhooks allow Grafana to integrate with external systems using HTTP.
- The Notification Policy tree starts with the Default Policy, and child routes handle specific matching conditions.
- Always verify your configuration using Grafana Provisioning APIs, as many labs and automation tools rely on these endpoints.



# Day 73 Notes
# Champion/Challenger Model Promotion using MLflow Model Aliases

---

# Introduction

In any Machine Learning system, training a model once is **not enough**.

After deployment, the data entering the system changes over time.

This phenomenon is called **Data Drift**.

When data drift occurs, the existing production model may no longer perform well because it was trained on older data.

To solve this problem, organizations build **automated retraining pipelines**.

The pipeline periodically:

1. Detects drift
2. Retrains the model
3. Evaluates it
4. Registers the new model version

However...

There is a very dangerous mistake many beginners make.

They automatically deploy every newly trained model.

That is a terrible idea.

Why?

Because...

**A newly trained model is not guaranteed to be better.**

Sometimes it becomes worse.

Sometimes the new data contains noise.

Sometimes the retraining dataset is imbalanced.

Sometimes hyperparameters are poor.

If we blindly replace production with every new model, users may suddenly experience lower prediction quality.

That is why almost every ML platform follows the

> Champion / Challenger pattern.

---

# What is the Champion?

The Champion is the model currently serving production traffic.

Think of it as the "current winner."

Users are making predictions using this model.

Example

```
fraud-detector
Version 1
Alias: production
F1 Score = 0.71
```

This model is trusted.

It has already passed evaluation.

It is currently serving real users.

---

# What is the Challenger?

Whenever retraining finishes, a new model is created.

Example

```
fraud-detector
Version 2
```

This model is called the

**Challenger**

It challenges the production model.

But it is NOT production yet.

It must first prove itself.

Example

```
Version 2
F1 Score = 0.82
```

Only after comparison can it replace Version 1.

---

# Why Compare Models?

Imagine this situation.

Current Production

```
Accuracy = 95%
```

Retrained Model

```
Accuracy = 91%
```

If we deploy it automatically...

Users immediately receive worse predictions.

Nothing broke technically.

The deployment succeeded.

But business performance became worse.

This is called

**Model Regression**

Machine Learning engineers spend a lot of time preventing this.

---

# Champion vs Challenger Workflow

```
                 Production Model
                  (Champion)

                      │
                      │
             New Data Arrives
                      │
                      ▼
            Drift Detection System
                      │
          Drift exceeds threshold?
                      │
              Yes
                      │
                      ▼
             Retraining Pipeline
                      │
                      ▼
             Register Version 2
                      │
                      ▼
             Evaluate New Model
                      │
                      ▼
         Compare Champion vs Challenger
                      │
        ┌─────────────┴─────────────┐
        │                           │
 Better Model                 Worse Model
        │                           │
        ▼                           ▼
 Promote to Production        Reject Model
```

---

# What Changed in MLflow 3.x?

Older MLflow versions used

```
Stages
```

Like

```
None

Staging

Production

Archived
```

Example

```
Version 1

Stage = Production
```

Promotion meant changing stages.

Example

```
Staging
↓

Production
```

This system had limitations.

Only one stage existed.

It wasn't flexible.

---

# MLflow 3.x Introduced Aliases

Instead of stages,

MLflow now recommends

Aliases.

Aliases are simply names pointing to versions.

Example

```
production
```

points to

```
Version 1
```

Tomorrow,

the same alias can point to

```
Version 2
```

Nothing else changes.

The application always loads

```
models:/fraud-detector@production
```

Notice something important.

The application never loads

```
Version 1
```

or

```
Version 2
```

It loads the alias.

Therefore changing production is simply

moving the alias.

---

# Why are Aliases Better?

Suppose today

```
production → Version 1
```

Tomorrow

```
production → Version 2
```

Next week

```
production → Version 5
```

The application code never changes.

Only the alias changes.

This is much cleaner.

---

# Project Structure

```
monitoring/

│
├── retrain_pipeline.py
│
└── promote.py
```

---

# What retrain_pipeline.py Does

This script has already done its work.

It

Registered

```
Version 1
```

Then

Registered

```
Version 2
```

Each version has

```
Run ID

Metrics

Artifacts

Parameters
```

Our task is only promotion.

---

# What promote.py Does

This script decides

Should Version 2 become production?

Yes or No.

Nothing more.

---

# Understanding the Code

---

## Creating the Client

```python
client = MlflowClient(
    tracking_uri="http://localhost:5000"
)
```

This client communicates with the MLflow Tracking Server.

Every registry operation goes through this object.

Examples

Get model

Get run

Read metrics

Move aliases

Everything.

---

# Reading Metrics

```python
def f1_of(version):
```

This helper function returns the F1 score of a model version.

Inside it,

First

```
client.get_model_version()
```

returns metadata.

Example

```
Run ID

Version

Creation Time
```

Then

```
client.get_run()
```

opens the associated MLflow run.

From that run we read

```
metrics["f1_score"]
```

So

```
Version

↓

Run

↓

Metrics

↓

F1 Score
```

---

# Step 1

Find Current Production

```python
champion = client.get_model_version_by_alias(
    MODEL,
    PROD_ALIAS
)
```

This asks

"What version is production currently pointing to?"

Answer

```
Version 1
```

---

# Step 2

Read Champion Score

```python
champion_f1 = f1_of(champion.version)
```

Returns

```
0.71
```

---

# Step 3

Read Challenger Score

```python
challenger_f1 = f1_of("2")
```

Returns

```
0.82
```

---

# Step 4

Compare

```python
if challenger_f1 > champion_f1:
```

This is the promotion gate.

Only one condition exists.

Strictly greater.

Not greater than or equal.

Not latest version.

Only

Better model.

---

# Why Strictly Greater?

Suppose

```
Champion = 0.82

Challenger = 0.82
```

Should we replace it?

No.

There is no improvement.

Changing production introduces unnecessary deployment risk.

Therefore

```
>
```

is preferred.

Not

```
>=
```

---

# Step 5

Move Production Alias

```python
client.set_registered_model_alias(
    MODEL,
    "production",
    "2"
)
```

This is the actual deployment.

Notice

No files move.

No artifacts copy.

No retraining happens.

No model upload happens.

Only the alias changes.

Before

```
production

↓

Version 1
```

After

```
production

↓

Version 2
```

Done.

Deployment completed.

---

# Output

```
Champion: v1 (0.71)

Challenger: v2 (0.82)

Promoted Version 2
```

Exactly what happened.

---

# Visual Representation

Before

```
production

↓

Version 1
```

After

```
production

↓

Version 2
```

---

# APIs Used

## get_model_version()

Returns metadata about a specific version.

---

## get_run()

Returns metrics, parameters and artifacts associated with a run.

---

## get_model_version_by_alias()

Returns the version pointed to by an alias.

Example

```
production

↓

Version 1
```

---

## set_registered_model_alias()

Moves an alias.

Example

```
production

↓

Version 2
```

---

# Why Not Use Version Numbers?

Suppose your application loads

```
Version 1
```

Tomorrow

Version 7 becomes best.

You must modify code.

Redeploy application.

Restart services.

Instead,

Load

```
models:/fraud-detector@production
```

Now only the alias changes.

Applications never change.

---

# Real Industry Workflow

```
Users

      │

      ▼

Production Model

      │

Prediction Requests

      │

      ▼

Monitor Performance

      │

Detect Drift

      │

Retrain

      │

Evaluate

      │

Register Version

      │

Champion vs Challenger

      │

      ▼

Promote?

      │

   Yes / No

      │

      ▼

Production Alias Updated
```

---

# Best Practices

- Never deploy a retrained model without evaluation.
- Always compare the new model against the production model.
- Use aliases instead of hardcoded version numbers.
- Load models using aliases (for example, `models:/fraud-detector@production`).
- Keep production stable by promoting only when there is a measurable improvement.
- Store evaluation metrics in MLflow runs so they can be used for automated promotion decisions.
- Make promotion logic deterministic and reproducible.

---

# Key Takeaways

- Data drift can reduce model performance over time.
- Retraining creates a challenger model, not an automatic replacement.
- The production model is called the champion.
- MLflow 3.x recommends aliases instead of stages.
- Applications should load models using aliases, not version numbers.
- A Champion/Challenger gate prevents worse models from reaching production.
- In this lab, Version 2 (F1 = 0.82) correctly replaced Version 1 (F1 = 0.71) by moving the `production` alias after passing the evaluation gate.


# Day 74 — Custom Business Metric and Grafana Version Variable

## 1. What We Are Building

In this lab, we extend an existing ML monitoring stack with a custom business metric.

The monitoring stack already contains:

- Flask metric emitter
- Prometheus
- Grafana

The existing application exposes standard ML-serving metrics such as:

- HTTP request count
- Prediction accuracy
- Data drift
- Model inference latency

The requirement is to add one more business-level metric:

`fraud_amount_usd_total`

This metric tracks the cumulative dollar amount of fraudulent transactions.

The important requirement is that the metric must be separated by model version.

For example:

    fraud_amount_usd_total{version="v1"}
    fraud_amount_usd_total{version="v2"}

Grafana will then use a dashboard variable called `version` so that a user can select:

- `v1`
- `v2`
- `All`

The final flow is:

    Flask Metric Emitter
            |
            | fraud_amount_usd_total{version="v1"}
            | fraud_amount_usd_total{version="v2"}
            v
        Prometheus
            |
            | label_values(...)
            v
       Grafana Variable
            |
            | $version
            v
       Grafana Panel

This is a common monitoring pattern:

    Metric
      ↓
    Label
      ↓
    Prometheus
      ↓
    Grafana Variable
      ↓
    Dashboard Filter


# 2. Understanding the Existing Application

The metric emitter is located at:

    /root/code/monitoring/app/metric_emitter.py

The application uses the Prometheus Python client.

The important imports are:

    from prometheus_client import (
        CollectorRegistry,
        Counter,
        Gauge,
        Histogram,
        generate_latest,
    )

The application creates its own Prometheus registry:

    REGISTRY = CollectorRegistry()

Using a custom registry is important because the application explicitly controls which metrics are exposed.

The existing request counter is:

    REQUEST_TOTAL = Counter(
        "flask_http_request_total",
        "Total HTTP requests handled, labelled by model version.",
        labelnames=["version", "endpoint", "method"],
        registry=REGISTRY,
    )

The existing counter already demonstrates how labels work.

For example:

    REQUEST_TOTAL.labels(
        version="v1",
        endpoint="/predict",
        method="POST",
    ).inc()

This creates a series similar to:

    flask_http_request_total{
        version="v1",
        endpoint="/predict",
        method="POST"
    }


# 3. Understanding Prometheus Counters

A Counter represents a value that normally increases over time.

Examples include:

- Number of requests
- Number of errors
- Number of transactions
- Total amount of money processed

Our business metric is also cumulative, so a Counter is appropriate.

The metric is:

    fraud_amount_usd_total

The `_total` suffix is conventional for Prometheus counters.

The metric represents:

    Total fraudulent transaction amount in USD

It is not a current balance.

It is a cumulative amount that increases as fraudulent transactions are simulated.


# 4. Why We Need a Version Label

The ML platform contains multiple model versions.

In this lab there are:

    v1
    v2

If we created the metric without a label:

    fraud_amount_usd_total

we would only have one value.

That would make it impossible to answer:

    How much fraudulent transaction value was processed by v1?

or:

    How much fraudulent transaction value was processed by v2?

Instead, we create a labelled metric:

    fraud_amount_usd_total{version="v1"}

    fraud_amount_usd_total{version="v2"}

Prometheus treats these as separate time series.

This is one of the most important concepts in Prometheus.

The metric name is the same:

    fraud_amount_usd_total

The label value differentiates the series:

    version="v1"

and:

    version="v2"


# 5. Adding the Custom Counter

Add the following metric definition to the Python application:

    FRAUD_AMOUNT_USD_TOTAL = Counter(
        "fraud_amount_usd_total",
        "Total fraudulent transaction amount in USD, labelled by model version.",
        labelnames=["version"],
        registry=REGISTRY,
    )

There are several important parts.

## Metric name

    "fraud_amount_usd_total"

This is the name that Prometheus and Grafana will query.

## Description

    "Total fraudulent transaction amount in USD, labelled by model version."

This appears in the `/metrics` endpoint as the HELP text.

## Label

    labelnames=["version"]

This tells Prometheus that every series of this metric will have a `version` label.

## Registry

    registry=REGISTRY

This ensures that the metric is exposed through the application's custom registry.


# 6. Updating `_nudge_metrics()`

The application has a background function:

    def _nudge_metrics() -> None:

This function continuously simulates activity.

It runs:

    while True:

Inside the loop, requests are simulated for two model versions:

    for version in ("v1", "v1", "v1", "v2"):

There are three simulated requests for `v1` and one for `v2` during each iteration.

The existing request metric is incremented:

    REQUEST_TOTAL.labels(
        version=version,
        endpoint="/predict",
        method="POST",
    ).inc()

The inference latency is also recorded:

    INFERENCE_LATENCY.observe(random.uniform(0.005, 0.15))


# 7. Generating Fraudulent Transaction Amounts

We need a value to add to the fraudulent amount counter.

A simulated amount can be generated with:

    fraud_amount = random.uniform(25.0, 500.0)

This generates a random floating-point value between:

    $25.00

and:

    $500.00

The amount is then added to the counter for the current model version:

    FRAUD_AMOUNT_USD_TOTAL.labels(version=version).inc(fraud_amount)

This is the key line.

For example, if:

    version = "v1"

and:

    fraud_amount = 250.50

the operation is effectively:

    fraud_amount_usd_total{version="v1"} += 250.50

If the next simulated transaction is:

    version = "v2"

and:

    fraud_amount = 100.25

then:

    fraud_amount_usd_total{version="v2"} += 100.25

Because this happens inside the loop, every iteration advances the totals.


# 8. Complete Relevant `_nudge_metrics()` Logic

The important section looks like:

    def _nudge_metrics() -> None:
        random.seed(42)
        accuracy = 0.85
        drift = {
            "amount": 0.10,
            "hour": 0.12,
            "num_tx_past_day": 0.08,
        }

        while True:
            for version in ("v1", "v1", "v1", "v2"):
                REQUEST_TOTAL.labels(
                    version=version,
                    endpoint="/predict",
                    method="POST",
                ).inc()

                INFERENCE_LATENCY.observe(
                    random.uniform(0.005, 0.15)
                )

                fraud_amount = random.uniform(25.0, 500.0)
                FRAUD_AMOUNT_USD_TOTAL.labels(
                    version=version
                ).inc(fraud_amount)

            accuracy = max(
                0.70,
                min(
                    0.95,
                    accuracy + random.uniform(-0.02, 0.02),
                ),
            )

            PREDICTION_ACCURACY.set(accuracy)

            for column, base in drift.items():
                drift[column] = max(
                    0.01,
                    min(
                        0.60,
                        drift[column] + random.uniform(-0.02, 0.03),
                    ),
                )

                DATA_DRIFT_SCORE.labels(
                    column=column
                ).set(drift[column])

            time.sleep(5)

The important addition is:

    fraud_amount = random.uniform(25.0, 500.0)

followed by:

    FRAUD_AMOUNT_USD_TOTAL.labels(version=version).inc(fraud_amount)


# 9. Checking Python Syntax

After editing the file, it is good practice to check the Python syntax:

    python3 -m py_compile /root/code/monitoring/app/metric_emitter.py

If there is no output, the syntax check succeeded.

If Python reports an error, fix the error before restarting the container.


# 10. Understanding the Docker Setup

The monitoring containers are:

    mon-grafana
    mon-prometheus
    metric-emitter

The containers can be viewed with:

    docker ps

The metric emitter container is:

    metric-emitter

The ports are:

    Grafana      3000
    Prometheus   9090
    Flask        5000


# 11. Restarting the Metric Emitter

The metric emitter file is bind-mounted into the container.

This means changes made on the host are available to the container.

After changing the Python file, restart the container:

    docker restart metric-emitter

Then verify that it is running:

    docker ps

Check the logs:

    docker logs --tail 50 metric-emitter

The Flask application should start without a Python traceback.

The application normally reports that it is listening on:

    0.0.0.0:5000


# 12. Prometheus Scraping

Prometheus periodically requests:

    /metrics

from the metric emitter.

The logs showed requests such as:

    GET /metrics HTTP/1.1" 200

The `200` status means the metrics endpoint is responding successfully.

Repeated requests every few seconds indicate that Prometheus is scraping the application.

For example:

    172.18.0.3 - - [08/Aug/2026 04:30:09] "GET /metrics HTTP/1.1" 200 -
    172.18.0.3 - - [08/Aug/2026 04:30:14] "GET /metrics HTTP/1.1" 200 -
    172.18.0.3 - - [08/Aug/2026 04:30:19] "GET /metrics HTTP/1.1" 200 -

This confirms the scrape endpoint is working.


# 13. Checking the Metric Directly

Before checking Prometheus, verify that Flask itself exposes the metric.

Run:

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

The expected output looks like:

    # HELP fraud_amount_usd_total Total fraudulent transaction amount in USD, labelled by model version.
    # TYPE fraud_amount_usd_total counter
    fraud_amount_usd_total{version="v1"} 18876.64438329985
    fraud_amount_usd_total{version="v2"} 6047.097498702644

The exact values will change.

The important requirement is that there are two non-empty series:

    fraud_amount_usd_total{version="v1"}

and:

    fraud_amount_usd_total{version="v2"}


# 14. Why the Values Increase

The metric is a Counter.

Every five seconds, `_nudge_metrics()` runs another iteration.

For each version, a random fraudulent amount is generated.

The counter is incremented:

    .inc(fraud_amount)

Therefore, the values should continue increasing.

For example:

    v1 = 1000
    v2 = 500

After another iteration:

    v1 = 1700
    v2 = 850

After another iteration:

    v1 = 2500
    v2 = 1200

The exact values are not important.

What matters is that they are non-empty and cumulative.


# 15. Checking Prometheus

Prometheus is available on port `9090`.

A direct API query can be used:

    curl -s 'http://localhost:9090/api/v1/query?query=fraud_amount_usd_total'

A successful response contains the metric series.

The verified response contained:

    version="v1"

and:

    version="v2"

with:

    instance="metric-emitter:5000"

and:

    job="metric-emitter"

This confirms the complete path:

    Flask
      ↓
    /metrics
      ↓
    Prometheus scrape
      ↓
    Prometheus time series


# 16. Querying Prometheus Directly

The Prometheus expression for the metric is:

    fraud_amount_usd_total

Running this query in the Prometheus UI should return the two series:

    fraud_amount_usd_total{version="v1"}

    fraud_amount_usd_total{version="v2"}

The metric is therefore ready for Grafana.


# 17. Understanding Grafana Template Variables

A Grafana template variable allows dashboard users to dynamically choose values.

Instead of creating separate panels for:

    v1

and:

    v2

we create one panel and let the user choose the model version.

The variable is named:

    version

The user can then select:

    v1
    v2
    All

The panel query references:

    $version

This makes the dashboard dynamic.


# 18. Creating the Grafana Dashboard

Grafana is available on port:

    3000

The login credentials are:

    Username: admin
    Password: grafana2026

There was initially no dashboard, so a new dashboard was created.

The dashboard was named:

    ML Fraud Monitoring

The variable and panel must be on the same dashboard.


# 19. Creating the `version` Variable

Open the dashboard.

Go to:

    Dashboard Settings
        ↓
    Variables
        ↓
    Add variable

Choose:

    Query variable

Set the name:

    version

The datasource should be:

    Prometheus

The query type should be:

    Classic query


# 20. The Important Variable Query

The variable query is:

    label_values(fraud_amount_usd_total, version)

This query asks Prometheus:

    Give me all the values of the `version` label
    from the `fraud_amount_usd_total` metric.

Because Prometheus contains:

    fraud_amount_usd_total{version="v1"}

and:

    fraud_amount_usd_total{version="v2"}

the query returns:

    v1
    v2


# 21. Understanding `label_values()`

The syntax is:

    label_values(metric, label)

For this lab:

    label_values(fraud_amount_usd_total, version)

The first argument is the metric:

    fraud_amount_usd_total

The second argument is the label:

    version

Therefore, Grafana dynamically discovers the available model versions.

This is better than manually typing:

    v1
    v2

because if another model version is added later, such as:

    v3

the variable can discover it automatically.


# 22. Variable Preview

After entering:

    label_values(fraud_amount_usd_total, version)

the Grafana variable preview returned:

    v1
    v2

This confirms that the query is working.


# 23. Enabling the All Option

Under the variable's Selection options, enable:

    Include All value

This adds an option representing all available versions.

The variable can then be used as:

    v1
    v2
    All

The All option is useful because the same panel can display all model versions.


# 24. Creating the Fraud Amount Panel

Create a panel on the same dashboard.

The panel title can be:

    Fraud Amount USD

Select the Prometheus datasource.

The query should be:

    fraud_amount_usd_total{version=~"$version"}

This query is the most important Grafana query in the lab.


# 25. Understanding the Panel Query

The basic metric is:

    fraud_amount_usd_total

The label filter is:

    version=~"$version"

Therefore:

    fraud_amount_usd_total{version=~"$version"}

means:

    Return fraud_amount_usd_total where the version label
    matches the value selected in the Grafana version variable.


# 26. Why Use `=~` Instead of `=`

A simple equality matcher would be:

    version="$version"

A regular-expression matcher is:

    version=~"$version"

The regular-expression form is useful for Grafana's All selection because Grafana can expand the variable into a pattern that matches multiple values.

Therefore:

    fraud_amount_usd_total{version=~"$version"}

works well for:

    v1

    v2

    All


# 27. Testing the Dashboard

After saving the variable and panel, the dashboard should show a `version` dropdown.

Test:

    version = v1

The panel should show the v1 series.

Then test:

    version = v2

The panel should show the v2 series.

Finally test:

    version = All

The panel should show both versions.


# 28. Final Grafana Configuration

The variable should be:

    Name:
    version

    Type:
    Query

    Datasource:
    Prometheus

    Query:
    label_values(fraud_amount_usd_total, version)

The panel should use:

    fraud_amount_usd_total{version=~"$version"}


# 29. Complete End-to-End Flow

The complete system works like this:

    1. Flask simulates transactions.

    2. A model version is associated with each transaction.

    3. A fraudulent transaction amount is generated.

    4. The amount is added to the Counter:

       FRAUD_AMOUNT_USD_TOTAL.labels(
           version=version
       ).inc(fraud_amount)

    5. Flask exposes the metric at:

       /metrics

    6. Prometheus scrapes the endpoint.

    7. Prometheus stores separate series:

       fraud_amount_usd_total{version="v1"}

       fraud_amount_usd_total{version="v2"}

    8. Grafana queries the available version labels:

       label_values(fraud_amount_usd_total, version)

    9. Grafana creates the `$version` dashboard variable.

   10. The dashboard panel uses:

       fraud_amount_usd_total{version=~"$version"}

   11. The user selects v1, v2, or All.

   12. The panel dynamically displays the selected series.


# 30. Important Prometheus Concepts Learned

## Metric

A metric represents a measurable value.

Example:

    fraud_amount_usd_total

## Counter

A Counter represents a cumulative value that normally increases.

Example:

    fraud_amount_usd_total

## Label

A label provides dimensions for a metric.

Example:

    version="v1"

## Time Series

A metric combined with a unique set of labels represents a time series.

For example:

    fraud_amount_usd_total{version="v1"}

is one time series.

And:

    fraud_amount_usd_total{version="v2"}

is another time series.

## Scraping

Prometheus periodically requests a metrics endpoint.

In this lab:

    Flask
      ↓
    /metrics
      ↓
    Prometheus

## Template Variable

Grafana uses template variables to dynamically change dashboard queries.

Example:

    $version

## `label_values()`

Grafana uses:

    label_values(fraud_amount_usd_total, version)

to dynamically discover available versions.


# 31. Important Grafana Concepts Learned

## Query Variable

A Query variable gets its values from a datasource.

In this lab:

    Datasource = Prometheus

## Classic Query

The variable uses the classic query format:

    label_values(fraud_amount_usd_total, version)

## Include All

The Include All option allows the user to select all versions.

## Variable Reference

Grafana variables are referenced with:

    $version

## Dynamic Filtering

The panel uses:

    fraud_amount_usd_total{version=~"$version"}

This allows one panel to work for multiple versions.


# 32. Troubleshooting

## Problem: Metric does not appear

Check the application:

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

If nothing appears, check the Python file and restart:

    docker restart metric-emitter

Then check:

    docker logs --tail 50 metric-emitter


## Problem: Prometheus does not show the metric

First verify Flask:

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

Then query Prometheus:

    curl -s 'http://localhost:9090/api/v1/query?query=fraud_amount_usd_total'

If Flask exposes the metric but Prometheus does not, check the Prometheus target configuration and target health.


## Problem: Grafana variable preview shows zero values

Verify the query:

    label_values(fraud_amount_usd_total, version)

Make sure the datasource is:

    Prometheus

Then verify Prometheus directly with:

    fraud_amount_usd_total


## Problem: Grafana panel says No data

First test the metric without the variable:

    fraud_amount_usd_total

If that works, test:

    fraud_amount_usd_total{version="v1"}

Then:

    fraud_amount_usd_total{version="v2"}

Finally use:

    fraud_amount_usd_total{version=~"$version"}


# 33. Useful Commands

Check all containers:

    docker ps

Restart the metric emitter:

    docker restart metric-emitter

View emitter logs:

    docker logs --tail 50 metric-emitter

Check the Flask metrics:

    curl http://localhost:5000/metrics

Filter the new metric:

    curl http://localhost:5000/metrics | grep fraud_amount_usd_total

Query Prometheus:

    curl -s 'http://localhost:9090/api/v1/query?query=fraud_amount_usd_total'

Prometheus web interface:

    http://localhost:9090

Grafana web interface:

    http://localhost:3000


# 34. Final Validation Checklist

## Metric Emitter

- [x] `FRAUD_AMOUNT_USD_TOTAL` is defined.
- [x] Metric name is `fraud_amount_usd_total`.
- [x] Metric is a Counter.
- [x] Counter has a `version` label.
- [x] Counter is registered with the custom `REGISTRY`.
- [x] Counter is incremented inside `_nudge_metrics()`.
- [x] A fraudulent amount is generated for each simulated request.
- [x] The metric increases over time.
- [x] Flask exposes the metric at `/metrics`.

## Prometheus

- [x] Prometheus scrapes the metric emitter.
- [x] `/metrics` returns HTTP 200.
- [x] `fraud_amount_usd_total` exists in Prometheus.
- [x] `v1` series exists.
- [x] `v2` series exists.
- [x] Both series have non-empty values.
- [x] Each series contains the `version` label.

## Grafana

- [x] Grafana dashboard was created.
- [x] Dashboard is named `ML Fraud Monitoring`.
- [x] Variable is named `version`.
- [x] Variable type is Query.
- [x] Prometheus is the datasource.
- [x] Query type is Classic query.
- [x] Variable query is `label_values(fraud_amount_usd_total, version)`.
- [x] Variable preview returns `v1`.
- [x] Variable preview returns `v2`.
- [x] Include All option is enabled.
- [x] Fraud Amount USD panel exists.
- [x] Panel references `fraud_amount_usd_total`.
- [x] Panel uses `$version`.
- [x] Dashboard is saved.


# 35. Final Architecture Diagram

    +-----------------------------+
    |       Flask Application      |
    |     metric_emitter.py       |
    +-------------+---------------+
                  |
                  | /metrics
                  |
                  v
    +-----------------------------+
    |          Prometheus         |
    |                             |
    | fraud_amount_usd_total      |
    |   version="v1"              |
    |   version="v2"              |
    +-------------+---------------+
                  |
                  | label_values(...)
                  |
                  v
    +-----------------------------+
    |      Grafana Variable       |
    |                             |
    | Name: version               |
    | Values: v1, v2, All         |
    +-------------+---------------+
                  |
                  | $version
                  |
                  v
    +-----------------------------+
    |       Grafana Panel         |
    |                             |
    | fraud_amount_usd_total      |
    | {version=~"$version"}       |
    +-----------------------------+


# 36. Core Pattern to Remember

The most important concept from this lab is:

    Counter
        ↓
    Labelled Series
        ↓
    Prometheus
        ↓
    label_values(...)
        ↓
    Grafana Template Variable
        ↓
    $version
        ↓
    Dynamic Dashboard Panel


# 37. Why This Pattern Matters

This pattern is not limited to ML model versions.

The same approach can be used for:

- Model versions
- Tenants
- Regions
- Environments
- Services
- Applications
- Teams
- Customers
- Deployment versions

For example, instead of:

    version

we could have:

    tenant

and use:

    label_values(metric_name, tenant)

Or:

    region

with:

    label_values(metric_name, region)

The same Prometheus and Grafana design pattern can therefore support multi-tenant and multi-dimensional monitoring.


# 38. Final Result

The ML monitoring platform now supports a custom business metric:

    fraud_amount_usd_total

The metric is separated by model version:

    fraud_amount_usd_total{version="v1"}

    fraud_amount_usd_total{version="v2"}

Prometheus successfully collects the metric.

Grafana dynamically discovers the available versions using:

    label_values(fraud_amount_usd_total, version)

The dashboard variable is:

    version

The Grafana panel uses:

    fraud_amount_usd_total{version=~"$version"}

The user can therefore select:

    v1
    v2
    All

from a single Grafana dashboard panel.

This completes the Counter → labelled series → Prometheus → `label_values()` → Grafana variable → `$version` workflow.

# Day 76 — CI Pipeline for ML Code Linting and Testing

> **Topic:** Gitea Actions, CI workflows, Git branches, pull requests, automated linting, automated testing, self-hosted runners, and merging a validated feature into `main`.

---

# 1. What Is This Task About?

In this task, we are working on an ML repository called:

```text
fraud-detector
```

The goal is to build a **Continuous Integration (CI) pipeline**.

The CI pipeline must automatically run:

1. **Linting** using Ruff
2. **Unit tests** using pytest

whenever a pull request is opened against the `main` branch.

The complete development flow is:

```text
main
  │
  ├── create feature branch: add-ci
  │
  ├── create CI workflow
  │
  ├── commit changes
  │
  ├── push add-ci
  │
  ├── create Pull Request
  │
  ├── Gitea Actions runs
  │      ├── lint
  │      └── test
  │
  ├── both checks pass
  │
  └── merge add-ci → main
```

The important lesson is that **CI should validate changes before they are merged into the main branch**.

---

# 2. What Is CI?

CI stands for:

```text
Continuous Integration
```

Continuous Integration means developers frequently integrate their changes into a shared repository, while automated systems verify that the changes do not break the project.

A typical CI pipeline performs tasks such as:

```text
Checkout code
      ↓
Install dependencies
      ↓
Lint code
      ↓
Run tests
      ↓
Build/package application
      ↓
Report result
```

For this task, we only need:

```text
Checkout code
      ↓
Install Ruff
      ↓
Run Ruff
      ↓
Install pytest
      ↓
Run pytest
```

---

# 3. Why Do We Need CI?

Imagine a developer changes:

```text
src/train.py
```

and accidentally introduces invalid Python syntax or breaks an existing function.

Without CI, the broken code could be merged into `main`.

With CI:

```text
Developer creates PR
        ↓
CI automatically runs
        ↓
Lint fails / tests fail
        ↓
PR is not ready to merge
```

This gives the team an automated safety mechanism.

The important principle is:

> **Never rely only on a developer manually checking whether code works. Automate repeatable validation.**

---

# 4. What Is Linting?

Linting is static analysis of source code.

A linter looks for problems such as:

- Invalid syntax
- Unused imports
- Bad formatting
- Undefined names
- Common programming mistakes
- Code-quality issues

This project uses:

```text
Ruff
```

Ruff is a very fast Python linter and formatter.

The command used by this project is:

```bash
ruff check .
```

The `.` means:

```text
Check the current directory and project files.
```

---

# 5. What Is Testing?

Testing verifies that the application behaves as expected.

This project uses:

```text
pytest
```

The tests are located under:

```text
tests/
```

Specifically:

```text
tests/test_train.py
```

The project contains three unit tests.

Running:

```bash
pytest
```

executes the test suite.

Expected result:

```text
3 passed
```

---

# 6. Repository Structure

The repository initially looks approximately like this:

```text
fraud-detector/
├── src/
│   └── train.py
├── tests/
│   └── test_train.py
├── pyproject.toml
└── .gitea/
    └── workflows/
        └── ci.yml.template
```

Let's understand each part.

## `src/train.py`

This is the ML training code.

The task description says it is a deterministic synthetic training script.

---

## `tests/test_train.py`

This contains unit tests for the training code.

There are three passing tests.

---

## `pyproject.toml`

This contains Python project configuration.

It includes configuration for:

```text
Ruff
pytest
```

---

## `.gitea/workflows/ci.yml.template`

This is the CI workflow template.

It is intentionally named:

```text
ci.yml.template
```

rather than:

```text
ci.yml
```

because Gitea Actions only schedules files ending in:

```text
.yml
.yaml
```

Therefore:

```text
ci.yml.template
```

is inert.

It does not execute.

We must rename it to:

```text
ci.yml
```

to activate the workflow.

---

# 7. What Is Gitea?

Gitea is a self-hosted Git service.

It provides functionality similar to GitHub, including:

- Git repositories
- Branches
- Pull requests
- Issues
- Actions/CI
- Code review
- Repository management

The local Gitea server in this task is running on:

```text
http://localhost:3000
```

The repository is:

```text
http://localhost:3000/gitea-admin/fraud-detector
```

---

# 8. Gitea Credentials

The task provides:

```text
Username: gitea-admin
Password: gitea2026
```

These credentials are used to access the local Gitea server.

---

# 9. What Are Gitea Actions?

Gitea Actions is Gitea's automation system.

It is conceptually similar to:

```text
GitHub Actions
```

A workflow describes:

```text
When should CI run?
        ↓
What jobs should run?
        ↓
What commands should each job execute?
```

The workflow uses YAML syntax.

---

# 10. GitHub Actions Compatibility

One important lesson from this task is:

> Gitea Actions uses the same general workflow syntax as GitHub Actions.

For example:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
```

This type of workflow can also be used in a GitHub repository with appropriate GitHub Actions configuration.

Therefore, learning Gitea Actions workflow syntax is also useful for GitHub Actions.

---

# 11. Understanding the Workflow

The final workflow is:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install ruff
        run: pip install --break-system-packages ruff
      - name: Run ruff
        run: ruff check .

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install pytest
        run: pip install --break-system-packages pytest
      - name: Run pytest
        run: pytest
```

Let's understand every important part.

---

# 12. Workflow Name

```yaml
name: CI
```

This gives the workflow a human-readable name.

Gitea displays this name in the Actions interface.

---

# 13. Workflow Triggers

The workflow contains:

```yaml
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
```

This defines when the workflow runs.

There are two triggers.

## Pull Request Trigger

```yaml
pull_request:
  branches: [main]
```

This means the workflow runs for pull requests targeting:

```text
main
```

For example:

```text
add-ci → main
```

will trigger the workflow.

---

## Push Trigger

```yaml
push:
  branches: [main]
```

This means the workflow also runs when changes are pushed directly to `main`.

Therefore, the workflow provides validation for both:

```text
Pull Requests → main
```

and:

```text
Pushes → main
```

---

# 14. What Is a Job?

A job is a group of steps that executes on a runner.

This workflow has two jobs:

```yaml
jobs:
  lint:
```

and:

```yaml
jobs:
  test:
```

Therefore the task requires:

```text
lint job
test job
```

---

# 15. The Lint Job

The lint job is:

```yaml
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install ruff
      run: pip install --break-system-packages ruff
    - name: Run ruff
      run: ruff check .
```

It performs three major actions.

### Step 1 — Checkout

```yaml
- uses: actions/checkout@v4
```

This checks out the repository code into the runner.

Without checking out the repository, the runner would not have the source code to analyze.

---

### Step 2 — Install Ruff

```yaml
- name: Install ruff
  run: pip install --break-system-packages ruff
```

This installs Ruff into the runner environment.

---

### Step 3 — Run Ruff

```yaml
- name: Run ruff
  run: ruff check .
```

This runs the linter.

If Ruff finds a problem, the job fails.

If Ruff finds no problems, the job succeeds.

---

# 16. The Test Job

The test job is:

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install pytest
      run: pip install --break-system-packages pytest
    - name: Run pytest
      run: pytest
```

Again, there are three major steps.

### Step 1 — Checkout

```yaml
- uses: actions/checkout@v4
```

The repository is downloaded into the runner.

---

### Step 2 — Install pytest

```yaml
- name: Install pytest
  run: pip install --break-system-packages pytest
```

This installs pytest.

---

### Step 3 — Run pytest

```yaml
- name: Run pytest
  run: pytest
```

This executes the project's tests.

If any test fails:

```text
test job → failed
```

If all tests pass:

```text
test job → successful
```

---

# 17. What Is a Runner?

A runner is the machine that executes CI jobs.

The task provides a:

```text
self-hosted Actions runner
```

That runner is already registered with Gitea.

When a workflow is triggered:

```text
Gitea
  ↓
Creates workflow job
  ↓
Runner receives job
  ↓
Runner executes commands
  ↓
Runner reports result
```

The runner is therefore the machine where commands such as:

```bash
ruff check .
```

and:

```bash
pytest
```

actually execute.

---

# 18. Why Did the Checks Initially Say "Waiting to Run"?

After creating the PR, the Gitea page showed:

```text
CI / lint     Waiting to run
CI / test     Waiting to run
```

This is normal.

The workflow had been detected, but the Actions runner had not yet executed the jobs.

Eventually the runner picked them up.

The workflow then showed:

```text
lint
10s

test
11s
```

and the lint job showed:

```text
Success
```

The test job also eventually became successful.

---

# 19. Git Branches

Git branches allow us to develop changes independently.

The repository has:

```text
main
```

as the primary branch.

Instead of modifying `main` directly, we create:

```text
add-ci
```

The development flow becomes:

```text
main
  │
  └── add-ci
```

We make our changes on `add-ci`.

---

# 20. Why Use a Feature Branch?

Feature branches protect the main branch.

Instead of:

```text
Developer → directly modifies main
```

we use:

```text
Developer
   ↓
Feature branch
   ↓
Pull Request
   ↓
CI
   ↓
Review
   ↓
Merge
   ↓
main
```

This is a standard software engineering workflow.

---

# 21. Create the Feature Branch

Start from the repository:

```bash
cd /root/code/fraud-detector
```

Check the current state:

```bash
git status
```

Create the feature branch:

```bash
git checkout -b add-ci
```

The current branch should become:

```text
add-ci
```

You can verify:

```bash
git branch --show-current
```

Expected:

```text
add-ci
```

---

# 22. Rename the Workflow

The template is:

```text
.gitea/workflows/ci.yml.template
```

Rename it:

```bash
mv .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml
```

This is important because Gitea only recognizes:

```text
*.yml
*.yaml
```

as workflow files.

---

# 23. Edit the TODO Commands

The template already contains the workflow structure.

We do not need to write the workflow from scratch.

The two project-specific commands are:

```yaml
run: ruff check .
```

and:

```yaml
run: pytest
```

This demonstrates an important real-world CI engineering principle:

> CI engineers often inherit templates and customize only the commands that are specific to the project.

---

# 24. Validate Locally Before Pushing

Always run the same commands locally before depending on CI.

Run:

```bash
ruff check .
```

Then:

```bash
pytest
```

Expected:

```text
Ruff → successful
pytest → 3 passed
```

This prevents obvious failures from being pushed to the repository.

---

# 25. Stage the Changes

After renaming the file, Git may show:

```text
deleted: .gitea/workflows/ci.yml.template
```

This is because the old filename disappeared.

Stage both paths:

```bash
git add .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml
```

Then:

```bash
git status
```

Git should detect the rename.

Conceptually:

```text
ci.yml.template
       ↓
     rename
       ↓
ci.yml
```

---

# 26. Commit the Change

Create a Git commit:

```bash
git commit -m "Add CI pipeline for linting and tests"
```

A commit records the change in Git history.

The commit contains the new workflow:

```text
.gitea/workflows/ci.yml
```

---

# 27. Push the Feature Branch

Push the branch to the Gitea server:

```bash
git push -u origin add-ci
```

The branch now exists remotely:

```text
local add-ci
      ↓
remote add-ci
```

---

# 28. Verify the Push

Run:

```bash
git status
```

A clean result should look like:

```text
On branch add-ci
Your branch is up to date with 'origin/add-ci'.

nothing to commit, working tree clean
```

This means:

- The branch exists
- Changes have been committed
- Changes have been pushed
- There are no uncommitted changes

---

# 29. Create the Pull Request

Now open Gitea.

Repository:

```text
gitea-admin/fraud-detector
```

Create a new pull request.

The branches must be:

```text
Base: main
Head: add-ci
```

This means:

```text
add-ci → main
```

The direction matters.

We are proposing to merge the feature branch into the main branch.

---

# 30. What Is a Pull Request?

A Pull Request, or PR, is a request to merge changes from one branch into another.

In this task:

```text
Source:
add-ci
```

and:

```text
Target:
main
```

The PR allows CI to validate the proposed changes before they become part of `main`.

---

# 31. CI Runs Automatically

Once the PR is created, the workflow trigger:

```yaml
pull_request:
  branches: [main]
```

matches the PR.

Gitea therefore starts:

```text
CI / lint
CI / test
```

The Checks page may initially show:

```text
Waiting to run
```

This means the jobs are queued.

---

# 32. Successful Checks

Eventually the checks should show:

```text
CI / lint (pull_request) Successful
CI / test (pull_request) Successful
```

Both are required.

Do not merge if one of them has failed.

The desired state is:

```text
lint  → Success
test  → Success
```

---

# 33. Why Checks Matter

The checks prove that the code satisfies the automated quality gates.

For this project:

```text
Ruff
  ↓
Code quality validation

pytest
  ↓
Behavior validation
```

Together they provide two different forms of protection.

---

# 34. Merge the Pull Request

After both checks pass, merge the PR.

Click:

```text
Merge Pull Request
```

Then confirm the merge.

The PR should change from:

```text
Open
```

to:

```text
Merged
```

The page should say:

```text
Pull request successfully merged and closed
```

---

# 35. Why the Merge Step Is Critical

This was the most important failure point in the first attempt.

Having:

```text
PR created
```

and:

```text
CI successful
```

is not enough.

The task explicitly requires:

```text
PR merged into main
```

The grader checks the PR's merged state.

Conceptually, it expects:

```json
{
  "merged": true
}
```

Therefore:

```text
PR open + checks successful
```

is incomplete.

The final required state is:

```text
PR merged + checks successful
```

---

# 36. Merge Commit

After merging, Gitea creates a merge commit.

In the completed run, the merge commit was:

```text
3d3503d52d
```

The important point is that the changes from:

```text
add-ci
```

are now incorporated into:

```text
main
```

---

# 37. Verify Main Locally

After the merge:

```bash
git checkout main
git pull
```

Then:

```bash
git log --oneline -3
```

The merge commit should appear in the history.

This confirms that the local `main` branch has been updated.

---

# 38. API Verification

The task also specifies API-level requirements.

The repository status endpoint is:

```text
GET /api/v1/repos/gitea-admin/fraud-detector/commits/{sha}/status
```

The combined status for the PR's head commit must be:

```text
success
```

The PR API is:

```text
GET /api/v1/repos/gitea-admin/fraud-detector/pulls
```

The relevant PR must have:

```text
merged: true
```

These API checks are useful because they verify the actual repository state rather than relying only on the UI.

---

# 39. Example API Commands

To list pull requests:

```bash
curl -u gitea-admin:gitea2026 \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/pulls
```

After obtaining the PR head SHA:

```bash
curl -u gitea-admin:gitea2026 \
  http://localhost:3000/api/v1/repos/gitea-admin/fraud-detector/commits/<SHA>/status
```

The status should report a successful combined result.

---

# 40. Complete Git Workflow

The complete command-line portion is:

```bash
cd /root/code/fraud-detector

git status

git checkout -b add-ci

mv .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml

ruff check .
pytest

git add .gitea/workflows/ci.yml.template .gitea/workflows/ci.yml

git commit -m "Add CI pipeline for linting and tests"

git push -u origin add-ci
```

Then:

```text
Open Gitea
    ↓
Create PR
    ↓
add-ci → main
    ↓
Wait for CI
    ↓
lint successful
    ↓
test successful
    ↓
Merge PR
```

Finally:

```bash
git checkout main
git pull
```

---

# 41. Final Architecture

The final repository workflow looks like:

```text
                    ┌──────────────────┐
                    │   Developer      │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │    add-ci        │
                    │  feature branch  │
                    └────────┬─────────┘
                             │
                             │ Pull Request
                             ▼
                    ┌──────────────────┐
                    │      Gitea       │
                    └────────┬─────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
        ┌────────────────┐      ┌────────────────┐
        │     lint       │      │      test      │
        │  ruff check .  │      │     pytest     │
        └───────┬────────┘      └───────┬────────┘
                │                       │
                └───────────┬───────────┘
                            │
                       Both pass
                            │
                            ▼
                    ┌──────────────────┐
                    │  Merge Pull      │
                    │    Request       │
                    └────────┬─────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │       main       │
                    │   merged code    │
                    └──────────────────┘
```

---

# 42. Important Concepts Learned

## Continuous Integration

Automatically validates changes when developers push code or create pull requests.

## Linting

Static code analysis that detects potential code-quality problems.

Tool:

```text
Ruff
```

Command:

```bash
ruff check .
```

## Unit Testing

Tests individual pieces of application behavior.

Tool:

```text
pytest
```

Command:

```bash
pytest
```

## Workflow

A YAML file describing CI automation.

Location:

```text
.gitea/workflows/ci.yml
```

## Job

A collection of steps executed by a runner.

This workflow has:

```text
lint
test
```

## Runner

The machine that executes workflow jobs.

This task uses a self-hosted runner.

## Branch

An independent line of Git development.

Feature branch:

```text
add-ci
```

Main branch:

```text
main
```

## Pull Request

A request to merge changes from one branch into another.

```text
add-ci → main
```

## Checks

Automated CI results associated with a pull request.

Required:

```text
lint → success
test → success
```

## Merge

The operation that incorporates the feature branch into `main`.

Required final state:

```text
merged: true
```

---

# 43. Common Mistakes

## Mistake 1 — Leaving `.template`

Wrong:

```text
.gitea/workflows/ci.yml.template
```

Correct:

```text
.gitea/workflows/ci.yml
```

Gitea Actions will not schedule the `.template` file.

---

## Mistake 2 — Forgetting the `lint` job

The workflow must contain:

```yaml
jobs:
  lint:
```

---

## Mistake 3 — Forgetting the `test` job

The workflow must contain:

```yaml
jobs:
  test:
```

---

## Mistake 4 — Using the Wrong Ruff Command

Required:

```bash
ruff check .
```

---

## Mistake 5 — Using the Wrong Test Command

Required:

```bash
pytest
```

---

## Mistake 6 — Creating the PR in the Wrong Direction

Correct:

```text
add-ci → main
```

Not:

```text
main → add-ci
```

---

## Mistake 7 — Merging Before Checks Finish

Wait for:

```text
CI / lint → Successful
CI / test → Successful
```

---

## Mistake 8 — Forgetting to Merge

This is especially important.

The task is not complete when:

```text
PR created
```

or even when:

```text
CI successful
```

The final requirement is:

```text
PR merged into main
```

---

# 44. Final Task Checklist

```text
[✓] Repository cloned
[✓] Feature branch add-ci created
[✓] ci.yml.template renamed to ci.yml
[✓] lint job configured
[✓] test job configured
[✓] ruff check . configured
[✓] pytest configured
[✓] Ruff passes locally
[✓] Pytest passes locally
[✓] Changes committed
[✓] add-ci pushed to Gitea
[✓] Pull request created
[✓] PR target is main
[✓] PR head is add-ci
[✓] CI lint successful
[✓] CI test successful
[✓] Pull request merged
[✓] Merge commit exists on main
[✓] Final PR state is merged
```

---

# 45. Key Takeaway

The most important CI/CD lesson from this task is:

```text
Code
  ↓
Feature Branch
  ↓
Pull Request
  ↓
Automated CI
  ├── Lint
  └── Test
  ↓
Successful Checks
  ↓
Code Review / Approval
  ↓
Merge
  ↓
main
```

A CI pipeline creates a repeatable quality gate between development and the main branch.

For this ML project, the quality gate is simple:

```text
Ruff must pass
AND
pytest must pass
```

Only after both checks succeed should the pull request be merged.

The final state is not merely "CI works." The complete task is:

```text
CI workflow exists
+
CI checks succeed
+
Pull Request exists
+
Pull Request targets main
+
Pull Request is merged
```
# Day 77 Notes — Fixing a Failing Data-Quality Job in Gitea Actions

## 1. What This Task Is About

This task is about debugging a failed CI/CD pipeline in Gitea Actions.

The ML platform team wants data-schema validation tests to run automatically as a CI gate for every pull request. The purpose is to catch bad or unexpected training data before it reaches the machine-learning pipeline.

A pull request named **Add data-quality CI gate** was created in the `fraud-detector` repository.

The pull request contains three CI jobs:

1. `lint`
2. `test`
3. `data-quality`

The first two jobs are already passing.

The newly added `data-quality` job is failing.

The objective is to:

1. Open the failed CI run.
2. Read the actual failure log.
3. Determine why the job failed.
4. Fix the workflow.
5. Keep the `data-quality` job.
6. Make sure its pytest command references a real `.py` file.
7. Commit the fix.
8. Push the fix to the PR branch.
9. Verify that all three CI jobs become green.
10. Confirm that the PR's combined status is `success`.

---

# 2. Important Repository Information

Repository:

    fraud-detector

Repository URL:

    http://localhost:3000/gitea-admin/fraud-detector

Working clone:

    /root/code/fraud-detector

Branch:

    add-data-validation

Target branch:

    main

Pull request:

    Add data-quality CI gate

PR number:

    #1

Gitea username:

    gitea-admin

Gitea password:

    gitea2026

Gitea is running on port:

    3000

The workflow file is:

    .gitea/workflows/ci.yml

---

# 3. Understanding the Pull Request

The pull request is trying to add a new CI gate for data quality.

The PR initially contains one commit:

    ci: add data-quality job (pre-review -- may be buggy)

The important point is that the workflow itself can appear perfectly reasonable when reading it, but the CI runner can still fail when it actually executes the commands.

This is why the CI log is important.

Do not assume that a workflow is correct just because the YAML syntax looks valid.

---

# 4. What Is Gitea Actions?

Gitea Actions is Gitea's CI/CD system.

It allows repositories to automatically execute workflows when events occur.

For example:

- Pull requests
- Pushes
- Releases
- Other configured repository events

A workflow is normally stored inside the repository.

In this task, the workflow is:

    .gitea/workflows/ci.yml

The workflow defines jobs.

Each job contains steps.

For example:

    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install pytest
            run: pip install --break-system-packages pytest
          - name: Run pytest
            run: python3 -m pytest tests/test_train.py -v

The runner executes those steps in order.

If one step fails, the job fails.

---

# 5. Understanding the Workflow

The workflow begins with:

    name: CI

This gives the workflow its name.

The workflow is configured to run for pull requests targeting `main`:

    on:
      pull_request:
        branches: [main]

It also runs when code is pushed directly to `main`:

    push:
      branches: [main]

The workflow contains three jobs:

    jobs:
      lint:
      test:
      data-quality:

---

# 6. The Lint Job

The lint job checks code quality using Ruff.

It checks:

    src

and:

    tests

The relevant command is:

    ruff check src tests

This job was already passing.

Therefore, there was no need to modify the lint job.

General lesson:

When debugging CI, avoid changing jobs that are already working unless there is a clear reason.

---

# 7. The Test Job

The test job runs the normal training-related tests.

It installs pytest:

    pip install --break-system-packages pytest

Then runs:

    python3 -m pytest tests/test_train.py -v

This job was also already passing.

Again, there was no reason to modify it.

---

# 8. The Data-Quality Job

The new job is:

    data-quality:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Install pytest + pandas
          run: pip install --break-system-packages pytest pandas
        - name: Run data-quality tests
          run: python3 -m pytest tests/test_data_validation.py -v

This job is designed to execute the data-quality/schema tests.

It performs three important things.

First, it checks out the repository:

    actions/checkout@v4

Second, it installs the required Python packages:

    pytest
    pandas

Third, it runs the data-quality tests using pytest.

The problem is in the third step.

---

# 9. Reading the CI Failure

The pull request Checks page showed:

    CI / lint — Successful

    CI / test — Successful

    CI / data-quality — Failing

The important debugging rule is:

> Do not stop at the red status. Open the failed job and read the log.

A red CI job only tells us that something failed.

The actual log tells us what failed.

---

# 10. Identifying the Root Cause

The data-quality job attempted to execute:

    tests/test_data_validation.py

However, the branch contains:

    tests/test_data_quality.py

The workflow therefore points pytest at the wrong file.

The incorrect command is:

    python3 -m pytest tests/test_data_validation.py -v

The correct command is:

    python3 -m pytest tests/test_data_quality.py -v

This is the root cause of the failure.

---

# 11. Why the Error Matters

Pytest needs the path supplied to it to refer to a real test file or test location.

The workflow tells pytest:

    tests/test_data_validation.py

But that file is not present on the branch.

The actual test file is:

    tests/test_data_quality.py

Therefore, the runner cannot execute the intended data-quality tests.

This is a classic CI failure caused by a mismatch between:

- the filename referenced by the workflow
- the filename that actually exists in the repository

---

# 12. The Required Fix

Only the incorrect test path needs to be changed.

Before:

    run: python3 -m pytest tests/test_data_validation.py -v

After:

    run: python3 -m pytest tests/test_data_quality.py -v

The job itself must NOT be deleted.

The requirement explicitly says that the `data-quality` job must remain declared.

Therefore, do not solve the problem by removing:

    data-quality:

Instead, fix the command inside the existing job.

---

# 13. Correct Data-Quality Job

The corrected job is:

    data-quality:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v4
        - name: Install pytest + pandas
          run: pip install --break-system-packages pytest pandas
        - name: Run data-quality tests
          run: python3 -m pytest tests/test_data_quality.py -v

Notice that everything else remains unchanged.

Only the test filename was corrected.

---

# 14. Correct Complete Workflow

The final `.gitea/workflows/ci.yml` should be:

    name: CI

    on:
      pull_request:
        branches: [main]
      push:
        branches: [main]

    jobs:
      lint:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install ruff
            run: pip install --break-system-packages ruff
          - name: Run ruff
            run: ruff check src tests

      test:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install pytest
            run: pip install --break-system-packages pytest
          - name: Run pytest
            run: python3 -m pytest tests/test_train.py -v

      data-quality:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install pytest + pandas
            run: pip install --break-system-packages pytest pandas
          - name: Run data-quality tests
            run: python3 -m pytest tests/test_data_quality.py -v

---

# 15. Making the Change Locally

The working clone is already available at:

    /root/code/fraud-detector

Move into the repository:

    cd /root/code/fraud-detector

Check the current branch:

    git branch --show-current

The expected branch is:

    add-data-validation

You can inspect the workflow with:

    cat .gitea/workflows/ci.yml

---

# 16. Editing the Workflow

The simplest change is to replace:

    tests/test_data_validation.py

with:

    tests/test_data_quality.py

For example:

    sed -i 's#tests/test_data_validation.py#tests/test_data_quality.py#' .gitea/workflows/ci.yml

Then inspect the file:

    cat .gitea/workflows/ci.yml

Confirm that the data-quality step now contains:

    run: python3 -m pytest tests/test_data_quality.py -v

---

# 17. Verify the Test File Exists

Before committing, verify that the referenced file actually exists.

Run:

    ls tests/

You should see:

    test_data_quality.py

You can also check directly:

    test -f tests/test_data_quality.py && echo "File exists"

Expected output:

    File exists

This is an important habit.

Whenever a CI command references a file, verify that the file exists in the branch.

---

# 18. Check the Git Diff

Before committing, inspect exactly what changed:

    git diff -- .gitea/workflows/ci.yml

The important diff should look like:

    - run: python3 -m pytest tests/test_data_validation.py -v
    + run: python3 -m pytest tests/test_data_quality.py -v

Ideally, there should be no unnecessary changes.

This is good CI debugging practice because small, focused changes are easier to understand and less likely to introduce new problems.

---

# 19. Commit the Fix

Stage the workflow:

    git add .gitea/workflows/ci.yml

Commit it:

    git commit -m "fix: correct data-quality test path"

The commit records the fix in Git.

---

# 20. Push the Fix

Push the branch:

    git push origin add-data-validation

The new commit is now pushed to the branch used by the pull request.

Because the PR is based on `add-data-validation`, pushing to this branch updates the PR.

The Gitea Actions workflow should automatically run again because the pull request has received a new commit.

---

# 21. Verify the New CI Run

Return to the pull request in Gitea.

Open:

    Add data-quality CI gate

Then open the:

    Checks

section.

The new workflow run should appear.

There should be three jobs:

    CI / lint
    CI / test
    CI / data-quality

Wait for all jobs to finish.

---

# 22. Expected Final Result

The desired result is:

    CI / lint          Successful
    CI / test          Successful
    CI / data-quality  Successful

The pull request should then show:

    All checks have passed

or an equivalent successful combined status.

The key requirement is that the PR head commit's combined status reaches:

    success

---

# 23. What Must Be True at the End

The final state must satisfy all of these conditions.

### Requirement 1 — Keep the data-quality job

The workflow must still contain:

    data-quality:

Do not delete it.

### Requirement 2 — Use an existing Python file

The pytest step must reference:

    tests/test_data_quality.py

This file exists on the `add-data-validation` branch.

### Requirement 3 — Push the fix

The corrected workflow must be committed and pushed to:

    add-data-validation

### Requirement 4 — All CI jobs pass

The final PR must show:

    lint — green
    test — green
    data-quality — green

### Requirement 5 — Combined status is successful

The PR's latest head commit must have:

    success

---

# 24. Why We Did Not Change the Other Jobs

It is tempting to modify the entire workflow when debugging CI.

That is unnecessary here.

The evidence shows:

    lint → passed
    test → passed
    data-quality → failed

Therefore, the investigation should focus on `data-quality`.

Changing working jobs creates unnecessary risk.

A good debugging approach is:

1. Identify the failing component.
2. Read its log.
3. Find the smallest cause.
4. Make the smallest safe correction.
5. Run CI again.
6. Verify the complete result.

---

# 25. Why the CI Log Is Important

Static inspection can tell you what the workflow is supposed to do.

The runtime log tells you what actually happened.

For example, a workflow can contain:

    python3 -m pytest tests/test_data_validation.py -v

and look syntactically correct.

YAML syntax is not the problem.

The command itself is valid shell syntax.

But the referenced file does not exist.

Therefore, the workflow can be syntactically valid while still failing at runtime.

This is why CI debugging requires looking at the execution log.

---

# 26. Important Git Concepts Used in This Task

## Working Tree

The files currently present in your local repository.

## Branch

A separate line of development.

The relevant branch is:

    add-data-validation

## Commit

A recorded snapshot of changes.

Example:

    git commit -m "fix: correct data-quality test path"

## Push

Uploads your local commit to the remote repository.

Example:

    git push origin add-data-validation

## Pull Request

A request to merge changes from one branch into another.

Here:

    add-data-validation → main

## CI

Continuous Integration.

CI automatically builds, tests, and checks code when repository events occur.

---

# 27. Important Git Commands

Check current directory:

    pwd

Move to repository:

    cd /root/code/fraud-detector

Check branch:

    git branch --show-current

Check status:

    git status

View workflow:

    cat .gitea/workflows/ci.yml

View changed files:

    git diff

Stage workflow:

    git add .gitea/workflows/ci.yml

Commit:

    git commit -m "fix: correct data-quality test path"

Push:

    git push origin add-data-validation

View recent commits:

    git log --oneline -5

---

# 28. One-Line Fix

The entire technical fix can be summarized as:

    - run: python3 -m pytest tests/test_data_validation.py -v
    + run: python3 -m pytest tests/test_data_quality.py -v

Everything else in the workflow can remain unchanged.

---

# 29. Complete Troubleshooting Thought Process

When you encounter a similar CI failure, think through it systematically.

### Step 1 — Find the failing job

Look at the PR Checks page.

Example:

    lint → green
    test → green
    data-quality → red

Focus on the red job.

### Step 2 — Open the job details

Do not guess from the workflow file.

Read the actual runtime output.

### Step 3 — Identify the failing command

Find the command that returned a failure.

Here it was the pytest command.

### Step 4 — Inspect the referenced resource

The command referenced:

    tests/test_data_validation.py

Check whether the file exists.

### Step 5 — Compare with the repository

The actual file is:

    tests/test_data_quality.py

### Step 6 — Make the smallest correction

Change only the incorrect filename.

### Step 7 — Commit and push

Use Git to record and publish the correction.

### Step 8 — Re-run CI

The push should trigger a new PR workflow.

### Step 9 — Verify every job

Do not stop when `data-quality` becomes green.

Confirm all three jobs are green.

### Step 10 — Confirm the PR status

The final combined status must be:

    success

---

# 30. Common Mistakes to Avoid

## Mistake 1 — Deleting the failing job

Do not remove:

    data-quality:

The purpose of the task is to fix the job, not eliminate the CI gate.

## Mistake 2 — Changing the wrong branch

The PR comes from:

    add-data-validation

Make sure the fix is pushed to that branch.

## Mistake 3 — Guessing the failure

Always inspect the job log first.

## Mistake 4 — Using a nonexistent test file

Always verify the test path:

    test -f tests/test_data_quality.py

## Mistake 5 — Changing unrelated jobs

`lint` and `test` were already successful.

Do not introduce unnecessary modifications.

## Mistake 6 — Forgetting to push

A local commit does not update the remote PR.

You need:

    git push origin add-data-validation

## Mistake 7 — Not waiting for the new run

After pushing, Gitea Actions needs to execute the workflow again.

Check the latest run rather than relying on the old failed run.

---

# 31. Final Checklist

Before considering Day 77 complete, verify:

- [ ] Repository is `fraud-detector`
- [ ] Branch is `add-data-validation`
- [ ] Workflow is `.gitea/workflows/ci.yml`
- [ ] `data-quality` job still exists
- [ ] `tests/test_data_quality.py` exists
- [ ] pytest references `tests/test_data_quality.py`
- [ ] Workflow change is committed
- [ ] Commit is pushed to `add-data-validation`
- [ ] New CI run has completed
- [ ] `CI / lint` is green
- [ ] `CI / test` is green
- [ ] `CI / data-quality` is green
- [ ] PR combined status is `success`

---

# 32. Final Takeaway

The central lesson from Day 77 is:

> When CI fails, investigate the runtime log instead of assuming the workflow is correct.

The workflow was structurally valid, and two jobs were working correctly.

The failure was caused by a simple filename mismatch:

    tests/test_data_validation.py

versus the actual file:

    tests/test_data_quality.py

The correct solution was a minimal one-line workflow change.

After committing and pushing that change, Gitea Actions should rerun the PR checks and all three jobs should pass.

Final expected state:

    lint          ✅
    test          ✅
    data-quality  ✅
    PR status     success

# Day 78 Notes: Parallelise Tests via a Gitea Actions Matrix Strategy

## 1. What This Task Is About

This task is about improving a CI/CD pipeline by running multiple test suites in parallel instead of running them one after another.

The repository is:

```text
fraud-detector
```

The CI workflow is:

```text
.gitea/workflows/ci.yml
```

The branch used for the task is:

```text
add-test-matrix
```

There are three separate test suites:

```text
tests/test_train.py
tests/test_data_quality.py
tests/test_model_contract.py
```

Originally, all three test suites were executed inside one `test` job.

The goal is to use a Gitea Actions matrix strategy so that each test suite gets its own CI job.

---

# 2. Why Do We Need a Matrix?

Imagine we have three test suites:

```text
test_train.py
test_data_quality.py
test_model_contract.py
```

If we run:

```bash
python3 -m pytest tests -v
```

pytest executes all the tests together.

From a CI perspective, this means:

```text
                 TEST JOB
                    |
          ---------------------
          |         |         |
        train    data       model
                  quality    contract
```

But they are all inside the same job and therefore execute serially.

For example:

```text
Start
  |
  +--> train tests
  |
  +--> data quality tests
  |
  +--> model contract tests
  |
Finish
```

If each suite takes 2 minutes, the total can approach:

```text
2 + 2 + 2 = 6 minutes
```

As the test suites grow, this becomes a CI bottleneck.

---

# 3. What Is a Matrix Strategy?

A matrix strategy allows us to define one job and execute it multiple times with different values.

Think of a matrix as a loop in CI.

For example, in a programming language:

```python
for suite in ["train", "data_quality", "model_contract"]:
    run_tests(suite)
```

The CI equivalent is:

```yaml
strategy:
  matrix:
    suite:
      - train
      - data_quality
      - model_contract
```

Gitea Actions expands this into separate job executions.

Conceptually:

```text
                 test job
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      train     data_quality  model_contract
        |           |           |
        v           v           v
   test_train   test_data    test_model
      .py       _quality.py   _contract.py
```

These matrix jobs can run independently and in parallel.

---

# 4. The Existing CI Workflow

Before the change, the workflow looked like this:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install ruff
        run: pip install --break-system-packages ruff
      - name: Run ruff
        run: ruff check src tests

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install pytest + runtime deps
        run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
      - name: Run all tests
        run: python3 -m pytest tests -v
```

There are two jobs:

```text
jobs:
├── lint
└── test
```

The `lint` job checks code quality using Ruff.

The `test` job runs the Python test suite.

---

# 5. Understanding the Lint Job

The lint job is:

```yaml
lint:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install ruff
      run: pip install --break-system-packages ruff
    - name: Run ruff
      run: ruff check src tests
```

## `runs-on`

```yaml
runs-on: ubuntu-latest
```

This tells Gitea Actions to run the job on an Ubuntu runner.

## Checkout

```yaml
- uses: actions/checkout@v4
```

This checks out the repository code so the runner can access the source files.

## Install Ruff

```yaml
- name: Install ruff
  run: pip install --break-system-packages ruff
```

Ruff is installed using pip.

## Run Ruff

```yaml
- name: Run ruff
  run: ruff check src tests
```

Ruff checks the `src` and `tests` directories.

The important requirement for this task is:

```text
DO NOT CHANGE THE LINT JOB
```

Only the `test` job needs to be converted to a matrix.

---

# 6. Understanding the Original Test Job

The original test job was:

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install pytest + runtime deps
      run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
    - name: Run all tests
      run: python3 -m pytest tests -v
```

Let's understand each part.

## Job name

```yaml
test:
```

This defines the CI job called `test`.

## Runner

```yaml
runs-on: ubuntu-latest
```

The test job runs on an Ubuntu runner.

## Checkout

```yaml
- uses: actions/checkout@v4
```

The repository is checked out onto the runner.

## Install dependencies

```yaml
- name: Install pytest + runtime deps
  run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
```

This installs:

```text
pytest
pandas
numpy
scikit-learn
joblib
```

These packages are required to run the tests.

## Run all tests

```yaml
- name: Run all tests
  run: python3 -m pytest tests -v
```

This command tells pytest to discover and run tests throughout:

```text
tests/
```

The problem is that all three suites are executed together.

---

# 7. The Three Test Files

The repository contains:

```text
tests/
├── test_train.py
├── test_data_quality.py
└── test_model_contract.py
```

The matrix values need to correspond exactly to these files.

The required mapping is:

```text
train
    |
    v
tests/test_train.py
```

```text
data_quality
    |
    v
tests/test_data_quality.py
```

```text
model_contract
    |
    v
tests/test_model_contract.py
```

This is why the matrix values are:

```yaml
- train
- data_quality
- model_contract
```

---

# 8. Adding the Matrix

The first change is to add:

```yaml
strategy:
  matrix:
    suite:
      - train
      - data_quality
      - model_contract
```

inside the `test` job.

The resulting structure is:

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      suite:
        - train
        - data_quality
        - model_contract
```

---

# 9. Understanding `strategy`

The keyword:

```yaml
strategy:
```

controls how a job is executed.

One of the most useful features of `strategy` is:

```yaml
matrix:
```

A matrix allows us to create multiple versions of the same job.

---

# 10. Understanding `matrix`

The matrix is:

```yaml
matrix:
  suite:
    - train
    - data_quality
    - model_contract
```

Here:

```text
suite
```

is the matrix variable.

Its possible values are:

```text
train
data_quality
model_contract
```

Gitea Actions creates one job for each value.

Therefore:

```text
suite = train
```

creates:

```text
test (train)
```

Then:

```text
suite = data_quality
```

creates:

```text
test (data_quality)
```

And:

```text
suite = model_contract
```

creates:

```text
test (model_contract)
```

---

# 11. Matrix as a Loop

A very useful way to remember this is:

```yaml
matrix:
  suite:
    - train
    - data_quality
    - model_contract
```

is conceptually similar to:

```python
for suite in ["train", "data_quality", "model_contract"]:
    ...
```

The CI system takes the same job definition and changes the value of:

```text
matrix.suite
```

for every execution.

---

# 12. Accessing the Matrix Value

Inside the job, we can access the current matrix value with:

```text
${{ matrix.suite }}
```

For example:

```yaml
- name: Run ${{ matrix.suite }} tests
```

When the current matrix value is:

```text
train
```

the step name becomes conceptually:

```text
Run train tests
```

When the value is:

```text
data_quality
```

it becomes:

```text
Run data_quality tests
```

When the value is:

```text
model_contract
```

it becomes:

```text
Run model_contract tests
```

---

# 13. Dynamically Selecting the Test File

The most important command is:

```yaml
run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

This combines a fixed filename pattern with the matrix value.

The pattern is:

```text
tests/test_<matrix-value>.py
```

## For `train`

The matrix value is:

```text
train
```

So the command becomes:

```bash
python3 -m pytest "tests/test_train.py" -v
```

## For `data_quality`

The matrix value is:

```text
data_quality
```

So the command becomes:

```bash
python3 -m pytest "tests/test_data_quality.py" -v
```

## For `model_contract`

The matrix value is:

```text
model_contract
```

So the command becomes:

```bash
python3 -m pytest "tests/test_model_contract.py" -v
```

This is why it is important that the matrix values match the test filenames.

---

# 14. Why We Do Not Duplicate the Job

A bad approach would be:

```yaml
test_train:
  ...

test_data_quality:
  ...

test_model_contract:
  ...
```

This duplicates the same configuration three times.

If we later need to change:

```yaml
pip install ...
```

we would need to change it in three places.

A matrix avoids this duplication.

We define the job once:

```yaml
test:
```

and define the variable values:

```yaml
matrix:
  suite:
    - train
    - data_quality
    - model_contract
```

This gives us three executions without copying the entire job.

---

# 15. Parallel Execution

The main benefit is that matrix jobs are independent.

Conceptually, before:

```text
                 test
                  |
              train tests
                  |
          data quality tests
                  |
         model contract tests
```

After:

```text
              test matrix
             /     |      \
            /      |       \
        train    data      model
                 quality   contract
```

The matrix cells can execute concurrently, subject to the available runners and CI configuration.

If the suites take roughly:

```text
train = 2 minutes
data_quality = 2 minutes
model_contract = 2 minutes
```

Serial execution could take approximately:

```text
2 + 2 + 2 = 6 minutes
```

Parallel execution can approach:

```text
max(2, 2, 2) = 2 minutes
```

There is some setup and scheduling overhead, so the actual time will not necessarily be exactly 2 minutes.

The important idea is that the suites no longer have to wait for each other.

---

# 16. Final Test Job

The completed test job is:

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      suite:
        - train
        - data_quality
        - model_contract

  steps:
    - uses: actions/checkout@v4
    - name: Install pytest + runtime deps
      run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
    - name: Run ${{ matrix.suite }} tests
      run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

Notice that the setup is still shared.

Each matrix cell:

1. Gets its own runner/job execution.
2. Checks out the repository.
3. Installs the required dependencies.
4. Runs only its assigned test suite.

---

# 17. Complete Final Workflow

The complete `.gitea/workflows/ci.yml` is:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install ruff
        run: pip install --break-system-packages ruff
      - name: Run ruff
        run: ruff check src tests

  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        suite:
          - train
          - data_quality
          - model_contract

    steps:
      - uses: actions/checkout@v4
      - name: Install pytest + runtime deps
        run: pip install --break-system-packages pytest pandas numpy scikit-learn joblib
      - name: Run ${{ matrix.suite }} tests
        run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

---

# 18. Important YAML Indentation

YAML indentation is significant.

The correct hierarchy is:

```text
jobs
└── test
    ├── runs-on
    ├── strategy
    │   └── matrix
    │       └── suite
    │           ├── train
    │           ├── data_quality
    │           └── model_contract
    └── steps
```

Therefore:

```yaml
test:
  runs-on: ubuntu-latest
  strategy:
    matrix:
      suite:
        - train
        - data_quality
        - model_contract
```

is correct.

Do not accidentally put `strategy` outside the `test` job.

---

# 19. Common Mistakes

## Mistake 1: Using the wrong matrix values

Incorrect:

```yaml
matrix:
  suite:
    - test_train
    - test_data
    - test_model
```

This would produce incorrect filenames when combined with:

```text
tests/test_${{ matrix.suite }}.py
```

The required values are:

```yaml
- train
- data_quality
- model_contract
```

---

## Mistake 2: Running the entire tests directory

If we keep:

```yaml
run: python3 -m pytest tests -v
```

every matrix cell will run all tests.

That defeats the purpose of the matrix.

Instead use:

```yaml
run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

Each matrix cell then runs only one suite.

---

## Mistake 3: Changing the lint job

The task specifically requires the `lint` job to remain unchanged.

Do not modify:

```yaml
lint:
```

The matrix is only for:

```yaml
test:
```

---

## Mistake 4: Creating three separate jobs

Avoid manually creating:

```yaml
test_train:
test_data_quality:
test_model_contract:
```

The point of the task is to demonstrate the matrix strategy.

Use:

```yaml
test:
  strategy:
    matrix:
      suite:
        - train
        - data_quality
        - model_contract
```

---

## Mistake 5: Incorrect filename

The actual files are:

```text
tests/test_train.py
tests/test_data_quality.py
tests/test_model_contract.py
```

The command must generate exactly these filenames.

---

# 20. Git Workflow Used

After modifying the workflow:

```bash
cd /root/code/fraud-detector
```

Check the current branch:

```bash
git branch --show-current
```

Expected:

```text
add-test-matrix
```

Check the changes:

```bash
git diff -- .gitea/workflows/ci.yml
```

Stage the workflow:

```bash
git add .gitea/workflows/ci.yml
```

Commit:

```bash
git commit -m "Convert test job to matrix strategy"
```

Push:

```bash
git push origin add-test-matrix
```

---

# 21. Commit Created

The workflow change was committed as:

```text
ca36d40
```

Commit message:

```text
Convert test job to matrix strategy
```

The branch was successfully pushed:

```text
add-test-matrix -> origin/add-test-matrix
```

---

# 22. Pull Request

The existing PR was:

```text
Convert test job to matrix strategy
```

The latest workflow run was triggered by commit:

```text
ca36d40
```

The workflow run was:

```text
#4
```

This confirmed that the new workflow was picked up by Gitea Actions.

---

# 23. CI Verification

The latest Gitea Actions run created these jobs:

```text
lint
test (data_quality)
test (model_contract)
test (train)
```

All four jobs were successful.

The result was:

```text
lint                    SUCCESS
test (data_quality)     SUCCESS
test (model_contract)   SUCCESS
test (train)            SUCCESS
```

This satisfies the requirement that the latest PR head commit reports a combined successful status with at least three `test` status entries.

---

# 24. How to Read the Gitea Actions Result

When opening the workflow run, we should not just check whether the workflow exists.

We need to confirm that the matrix actually expanded.

Look for:

```text
test (data_quality)
test (model_contract)
test (train)
```

If there is only:

```text
test
```

then the matrix is not working as expected.

The presence of the three separate entries proves that Gitea created three matrix cells.

---

# 25. Why Matrix Testing Is Useful in Real Projects

As a project grows, the number of test suites can increase.

For example:

```text
unit tests
integration tests
API tests
data quality tests
model tests
security tests
contract tests
```

Running everything serially can make CI very slow.

A matrix can separate these workloads:

```yaml
strategy:
  matrix:
    suite:
      - unit
      - integration
      - api
      - data_quality
      - model
      - security
      - contract
```

One job definition can then handle all of them.

This makes the workflow easier to maintain.

---

# 26. Matrix Can Be Used for More Than Test Suites

Matrix strategies are not limited to tests.

A common example is Python versions:

```yaml
strategy:
  matrix:
    python-version:
      - "3.10"
      - "3.11"
      - "3.12"
```

This creates separate jobs for each Python version.

Conceptually:

```text
test Python 3.10
test Python 3.11
test Python 3.12
```

Another example is operating systems:

```yaml
strategy:
  matrix:
    os:
      - ubuntu-latest
      - windows-latest
      - macos-latest
```

This tests the project across multiple operating systems.

The same principle applies:

```text
one job definition
        +
multiple matrix values
        =
multiple independent jobs
```

---

# 27. Matrix Variables

A matrix can contain different dimensions.

For example:

```yaml
strategy:
  matrix:
    python:
      - "3.10"
      - "3.11"
    os:
      - ubuntu
      - windows
```

This can produce combinations such as:

```text
Python 3.10 + Ubuntu
Python 3.10 + Windows
Python 3.11 + Ubuntu
Python 3.11 + Windows
```

This is called a multi-dimensional matrix.

For this task, only one dimension is required:

```yaml
suite:
```

---

# 28. Important GitHub/Gitea Actions Expression Syntax

The matrix value is accessed using:

```text
${{ matrix.suite }}
```

The `${{ }}` syntax tells the Actions workflow engine to evaluate an expression.

For example:

```yaml
run: echo "${{ matrix.suite }}"
```

could produce:

```text
train
```

in one matrix execution.

The value changes for each matrix cell.

---

# 29. Why Quotes Are Used Around the Test Path

The command is:

```yaml
run: python3 -m pytest "tests/test_${{ matrix.suite }}.py" -v
```

The quotes make the generated path a single shell argument.

For these filenames, quotes are not strictly necessary because they do not contain spaces, but they make the command explicit and safe.

The important part is:

```text
tests/test_
```

plus:

```text
${{ matrix.suite }}
```

plus:

```text
.py
```

Together:

```text
tests/test_${{ matrix.suite }}.py
```

---

# 30. What Happens Internally

When Gitea reads:

```yaml
strategy:
  matrix:
    suite:
      - train
      - data_quality
      - model_contract
```

it effectively creates three executions.

### Execution 1

```text
matrix.suite = train
```

Runs:

```bash
python3 -m pytest tests/test_train.py -v
```

### Execution 2

```text
matrix.suite = data_quality
```

Runs:

```bash
python3 -m pytest tests/test_data_quality.py -v
```

### Execution 3

```text
matrix.suite = model_contract
```

Runs:

```bash
python3 -m pytest tests/test_model_contract.py -v
```

The job definition itself is still only written once.

---

# 31. Definition of a Matrix Cell

A matrix cell means one particular combination of matrix variables.

In this task, there is only one matrix variable:

```text
suite
```

Therefore the cells are:

```text
suite=train
suite=data_quality
suite=model_contract
```

Gitea displays these as:

```text
test (train)
test (data_quality)
test (model_contract)
```

---

# 32. CI Status Behavior

Each matrix cell has its own status.

For example:

```text
test (train)            SUCCESS
test (data_quality)     SUCCESS
test (model_contract)   SUCCESS
```

If one cell fails:

```text
test (train)            SUCCESS
test (data_quality)     FAILURE
test (model_contract)   SUCCESS
```

the overall test workflow/PR status can fail.

This is useful because we can immediately see which suite failed.

Instead of one large test job saying:

```text
test FAILED
```

we get:

```text
test (data_quality) FAILED
```

which gives much more useful information.

---

# 33. Benefits of the Matrix Approach

## Faster CI

Independent suites can run simultaneously.

## Less Duplication

One job definition handles all suites.

## Easier Maintenance

Dependencies and setup steps are written once.

## Better Failure Visibility

Each suite gets a separate status.

## Easy Expansion

Adding another test suite is simple.

For example:

```yaml
- security
```

would create:

```text
test (security)
```

assuming:

```text
tests/test_security.py
```

exists.

---

# 34. Requirements Checklist

The task required the following:

### Requirement 1

The `test` job must declare:

```yaml
strategy:
  matrix:
```

Status:

```text
DONE
```

### Requirement 2

The matrix must contain:

```text
train
data_quality
model_contract
```

Status:

```text
DONE
```

### Requirement 3

Each value must map to an existing test file.

Mappings:

```text
train          -> tests/test_train.py
data_quality   -> tests/test_data_quality.py
model_contract -> tests/test_model_contract.py
```

Status:

```text
DONE
```

### Requirement 4

The lint job must remain unchanged.

Status:

```text
DONE
```

### Requirement 5

The latest PR head commit must report successful CI.

Commit:

```text
ca36d40
```

Status:

```text
SUCCESS
```

### Requirement 6

There must be at least three test status entries.

Observed:

```text
test (data_quality)
test (model_contract)
test (train)
```

Status:

```text
DONE
```

---

# 35. Final Architecture

The final CI pipeline is:

```text
                         CI
                          |
              +-----------+-----------+
              |                       |
             lint                    test
              |                       |
         Ruff checks             Matrix strategy
                                      |
                    +-----------------+-----------------+
                    |                 |                 |
                    v                 v                 v
              test (train)    test (data_quality)  test (model_contract)
                    |                 |                 |
                    v                 v                 v
             test_train.py     test_data_quality.py  test_model_contract.py
```

This is the desired architecture.

---

# 36. Practical Pattern to Remember

When you see multiple similar CI jobs, ask:

> Can these jobs be represented as different values of a matrix variable?

Instead of:

```text
job A
job B
job C
```

consider:

```yaml
job:
  strategy:
    matrix:
      value:
        - A
        - B
        - C
```

Then reference the value using:

```text
${{ matrix.value }}
```

This is one of the most useful patterns for reducing CI configuration duplication.

---

# 37. Quick Revision

### What is a matrix?

A matrix is a way to run the same CI job multiple times with different parameter values.

### What is the matrix variable in this task?

```text
suite
```

### What are its values?

```text
train
data_quality
model_contract
```

### How do we access the current value?

```text
${{ matrix.suite }}
```

### How do we select the test file?

```text
tests/test_${{ matrix.suite }}.py
```

### What does `train` produce?

```text
tests/test_train.py
```

### What does `data_quality` produce?

```text
tests/test_data_quality.py
```

### What does `model_contract` produce?

```text
tests/test_model_contract.py
```

### Why use a matrix?

To avoid duplicated CI configuration and allow independent jobs to run in parallel.

### What stayed unchanged?

The `lint` job.

### What commit contains the solution?

```text
ca36d40
```

### Did CI pass?

Yes.

### What jobs were created?

```text
lint
test (data_quality)
test (model_contract)
test (train)
```

---

# 38. Final Result

The `fraud-detector` CI pipeline was successfully converted from a serial test job into a matrix-based test strategy.

The final matrix is:

```yaml
strategy:
  matrix:
    suite:
      - train
      - data_quality
      - model_contract
```

Each matrix value maps directly to an existing test file:

```text
train          -> tests/test_train.py
data_quality   -> tests/test_data_quality.py
model_contract -> tests/test_model_contract.py
```

The latest CI run successfully produced three separate test jobs and one lint job.

```text
lint                    SUCCESS
test (train)            SUCCESS
test (data_quality)     SUCCESS
test (model_contract)   SUCCESS
```

## Day 78 Status

**COMPLETED SUCCESSFULLY** ✅

The key lesson is:

> A CI matrix lets you define one job once and execute it multiple times with different parameters, making CI pipelines more parallel, maintainable, and scalable.
---

