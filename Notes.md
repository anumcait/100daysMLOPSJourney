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
