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


