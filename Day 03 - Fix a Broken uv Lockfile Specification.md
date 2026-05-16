# Day 3: Fix a Broken uv Lockfile Specification

## Task Description
The xFusionCorp Industries ML team utilizes `uv` for lightning-fast and reproducible dependency management. A teammate left a broken `requirements.in` file that needs to be corrected and compiled into a pinned lockfile.

### Requirements:
1. **Core Packages**: The specification must include exactly:
   - `scikit-learn`
   - `mlflow`
   - `pandas`
   - `numpy`
2. **Version Constraints**: Every package must have a version constraint that can be satisfied on PyPI.
3. **Compilation**: Use `uv` to compile the `.in` file into a pinned `requirements.txt` (lockfile) including all transitive dependencies.

## Solution

### 1. Correcting `requirements.in`
The file at `/root/code/fraud-detection/requirements.in` was updated to ensure valid versioning:

```text
scikit-learn>=1.3.0
mlflow>=2.0.0
pandas>=2.1.0
numpy>=1.26.0
```

### 2. Compiling the Lockfile
Navigating to the project directory and running the compilation command:

```bash
# Navigate to project directory
cd /root/code/fraud-detection/

# Compile the pinned lockfile
uv pip compile requirements.in -o requirements.txt
```

### 3. Verification
The resulting `requirements.txt` now contains exact versions (e.g., `pandas==2.1.1`) and all necessary sub-dependencies, ensuring a reproducible environment across all team machines.

```bash
# Preview the generated lockfile
cat requirements.txt
```
