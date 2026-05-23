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

