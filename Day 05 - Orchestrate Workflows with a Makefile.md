# Day 5: Orchestrate Workflows with a Makefile

## Task Description
The xFusionCorp Industries ML team uses a `Makefile` to orchestrate common tasks such as environment setup, data processing, training, testing, and cleanup. A draft `Makefile` exists at `/root/code/fraud-detection/Makefile`, but `make all` does not complete successfully.

The goal is to bring the `Makefile` in line with the team's standard by creating or correcting it to support six key targets:
1. **setup**: Creates a Python virtual environment at `mlops-venv/` and installs dependencies from `requirements.txt`.
2. **data**: Runs the data processing script using `python src/data/process_data.py`.
3. **train**: Runs the model training script using `python src/models/train.py`.
4. **test**: Executes the test suite using `pytest tests/`.
5. **clean**: Recursively removes all `__pycache__` directories, removes `.pytest_cache`, and clears the contents of the `models/` directory.
6. **all**: Orchestrates the entire workflow by running `setup`, `data`, `train`, and `test` in that exact order.

### Requirements:
- Declare all six target names as `.PHONY` so that Make never confuses them with files of the same name.
- Makefile recipes **must** be indented with a real tab character, not spaces.
- `make all` must complete without error.

---

## Solution

To achieve this, change into the project directory and update the `Makefile`:

```bash
cd /root/code/fraud-detection
```

### The Corrected Makefile
Below is the standard, production-ready `Makefile`. Note that all recipe lines are strictly indented with tab characters.

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

---

## Target Breakdown

- **`.PHONY`**: Informs Make that these targets are not physical files, preventing issues if directories/files with names like `clean` or `test` are created.
- **`all`**: Chains the execution of the pipeline. Since `setup` builds the virtual environment and installs dependencies, subsequent targets (`data`, `train`, `test`) can safely execute within the defined environment.
- **`clean`**: Keeps the workspace clean by removing compiled Python bytecode (`__pycache__`), testing artifacts (`.pytest_cache`), and clearing previous model weights from `models/`.

---

## Verification

To clean the workspace and run the full end-to-end pipeline, run:

```bash
# Clean up previous runs
make clean

# Execute the entire workflow
make all
```

This should run the environment setup, process raw data, train the fraud detection model, and run unit tests successfully!

### Screenshots
