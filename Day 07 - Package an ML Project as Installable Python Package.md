# Day 7: Package an ML Project as Installable Python Package

## Task Description
The xFusionCorp Industries ML team is ready to distribute and deploy the fraud-detection model. To make the project portable and easily installable in various target environments, we must package the project as a standard, installable Python distribution.

The goal is to create a PEP 517-compliant `pyproject.toml` file at the root of the project `/root/code/fraud-detection/` and build the `fraud_detection` package.

The configuration must meet the following specifications:
1. Declare `setuptools>=61.0` and `wheel` as the build system requirements, using `setuptools.build_meta` as the backend.
2. Define project metadata with the name `fraud_detection` and version `0.1.0`.
3. Require Python version `>=3.10`.
4. Include core runtime dependencies: `scikit-learn`, `pandas`, and `numpy`.
5. Specify that source files are located in the `src/` directory.

Once configured, we must build the package to generate both a source distribution (`.tar.gz`) and a built distribution (`.whl`) under the `dist/` directory.

---

## Solution

To achieve this, we configure `pyproject.toml` at the project root and execute the build tool.

### 1. Configure `pyproject.toml`
Create or update `/root/code/fraud-detection/pyproject.toml` with the build system settings, package metadata, dependencies, and code quality configurations (retaining Ruff and Black settings from Day 6):

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

[tool.black]
line-length = 120

[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "W", "I"]
```

### 2. Build the Package
Navigate to the project root directory and build the package using the Python `build` module:

```bash
cd /root/code/fraud-detection

# Generate source and built distributions
python3 -m build
```

Running this command creates a `dist/` directory containing two archives:
- A source archive: `fraud_detection-0.1.0.tar.gz`
- A wheel binary: `fraud_detection-0.1.0-py3-none-any.whl`

---

## Verification

To verify the distribution files and validate the package installation:

### 1. Verify Distribution Outputs
Check that both build artifacts exist under `dist/`:

```bash
ls -l dist/
```

### 2. Install the Package Locally
To ensure the package installs correctly, activate your environment and install the generated wheel file:

```bash
pip install dist/fraud_detection-0.1.0-py3-none-any.whl
```

### 3. Verify Import and Metadata
Test that the package can be imported from any location and that its version is read correctly:

```bash
python -c "import fraud_detection; print(fraud_detection.__name__, 'installed successfully!')"
```

Once completed, the ML model package is ready to be published to a private registry (like PyPI, AWS CodeArtifact, or Nexus) or directly bundled into target production deployment environments.

### Screenshots
<img width="600" height="590" alt="image" src="https://github.com/user-attachments/assets/1518051f-3e91-40f7-87cd-ad7cafed6c8c" />
<img width="600" height="590" alt="image" src="https://github.com/user-attachments/assets/b82c118b-3ef9-4808-a047-a5f231be066d" />
<img width="600" height="591" alt="image" src="https://github.com/user-attachments/assets/2580aa3b-24f0-4961-b017-b2a05632c0ff" />
<img width="600" height="591" alt="image" src="https://github.com/user-attachments/assets/3cf78b04-b9a2-4919-8319-c813274e3dd8" />





