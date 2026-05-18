# Day 4: Create a Standard ML Project Structure

## Task Description
A colleague has started a new ML project at `/root/code/fraud-detection/`, but the layout does not match the xFusionCorp Industries standard. Bring the project in line with the team's conventions.

The final layout must match the tree below exactly:

```text
fraud-detection/
├── data/
│   ├── raw/
│   └── processed/
├── models/
├── notebooks/
├── src/
│   ├── data/
│   ├── features/
│   ├── models/
│   └── utils/
├── tests/
├── configs/
├── requirements.txt
└── README.md
```

Every subdirectory under `src/` must contain an `__init__.py` file so that Python recognises it as a package.

`requirements.txt` must list the following dependencies, one per line: `scikit-learn`, `pandas`, `numpy`, and `mlflow`. The canonical PyPI name for the scikit-learn package is `scikit-learn`.

`README.md` must begin with the heading `# fraud-detection`.

Review the existing project and correct everything that does not match the requirements above.

## Solution

To bring the project into the required structure, the following commands were run:

```bash
cd /root/code/fraud-detection

# Create required directories
mkdir -p data/raw
mkdir -p data/processed
mkdir -p models
mkdir -p notebooks
mkdir -p src/data
mkdir -p src/features
mkdir -p src/models
mkdir -p src/utils
mkdir -p tests
mkdir -p configs

# Create __init__.py files
touch src/data/__init__.py
touch src/features/__init__.py
touch src/models/__init__.py
touch src/utils/__init__.py

# Create/update requirements.txt
cat > requirements.txt <<EOF
scikit-learn
pandas
numpy
mlflow
EOF

# Create/update README.md
cat > README.md <<EOF
# fraud-detection
EOF
```

### Verification
You can verify the final structure with:

```bash
tree /root/code/fraud-detection
```
