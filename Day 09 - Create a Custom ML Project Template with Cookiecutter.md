# Day 09 — Create a Custom ML Project Template with Cookiecutter

## 📋 Task Summary

Fix a broken Cookiecutter template at `/root/code/mlops-template/` and use it to generate a project at `/root/code/churn-model/`.

---

## 🔍 Step 1: Inspect the Existing Template

```bash
# View the folder structure
find /root/code/mlops-template/ -type f | sort
cat /root/code/mlops-template/cookiecutter.json
```

Common issues in broken Cookiecutter templates:
- Malformed `cookiecutter.json` (missing keys, wrong types, bad JSON)
- Incorrect template directory name (must be `{{cookiecutter.project_name}}`)
- Jinja2 syntax errors in template files (`{{ }}` vs `{% %}`)
- Missing directories or files inside the template folder

---

## 🛠️ Step 2: Fix `cookiecutter.json`

The root `cookiecutter.json` must declare **four variables** with these exact defaults and choices:

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

> **Key Point:** `ml_framework` uses a **JSON array** so Cookiecutter presents it as a choice selector. The first item (`sklearn`) is the default.

---

## 🛠️ Step 3: Fix the Template Directory Name

The template directory **must** be named exactly `{{cookiecutter.project_name}}`:

```bash
# Check what the template directory is currently named
ls /root/code/mlops-template/

# If it's wrong (e.g., {{project_name}}, or project_name, etc.), rename it:
# Example fix (adjust based on what you find):
cd /root/code/mlops-template/
# Remove any incorrectly named directory and create the correct one
mv "$(ls -d */ | head -1)" "{{cookiecutter.project_name}}" 2>/dev/null

# OR if it doesn't exist at all:
mkdir -p "{{cookiecutter.project_name}}"
```

---

## 🛠️ Step 4: Create Required Directories Inside the Template

```bash
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/data"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/models"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/src"
mkdir -p "/root/code/mlops-template/{{cookiecutter.project_name}}/tests"
```

> **Note:** Cookiecutter only copies directories that have files. Add `.gitkeep` files to keep empty dirs:

```bash
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/data/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/models/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/src/.gitkeep"
touch "/root/code/mlops-template/{{cookiecutter.project_name}}/tests/.gitkeep"
```

---

## 🛠️ Step 5: Fix `README.md` Template

The README must reference **both** `project_name` and `author`:

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

---

## 🛠️ Step 6: Fix `requirements.txt` Template

The `requirements.txt` must use **Jinja2 conditionals** to select the correct package:

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

> **Key Points:**
> - `sklearn` → `scikit-learn` (the PyPI package name)
> - `pytorch` → `torch` (the PyPI package name)
> - `tensorflow` → `tensorflow`
> - The `-%}` strips trailing whitespace/newlines for clean output

---

## 🛠️ Step 7: Verify the Template Structure

```bash
find /root/code/mlops-template/ -not -path '*/.git/*' | sort
```

Expected output:
```
/root/code/mlops-template/
/root/code/mlops-template/cookiecutter.json
/root/code/mlops-template/{{cookiecutter.project_name}}/
/root/code/mlops-template/{{cookiecutter.project_name}}/README.md
/root/code/mlops-template/{{cookiecutter.project_name}}/requirements.txt
/root/code/mlops-template/{{cookiecutter.project_name}}/data/
/root/code/mlops-template/{{cookiecutter.project_name}}/data/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/models/
/root/code/mlops-template/{{cookiecutter.project_name}}/models/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/src/
/root/code/mlops-template/{{cookiecutter.project_name}}/src/.gitkeep
/root/code/mlops-template/{{cookiecutter.project_name}}/tests/
/root/code/mlops-template/{{cookiecutter.project_name}}/tests/.gitkeep
```

---

## 🚀 Step 8: Generate the Project

```bash
cookiecutter /root/code/mlops-template/ -o /root/code/ --no-input project_name=churn-model ml_framework=sklearn
```

---

## ✅ Step 9: Verify the Generated Project

```bash
# Check directory structure
find /root/code/churn-model/ | sort

# Verify requirements.txt contains scikit-learn
cat /root/code/churn-model/requirements.txt

# Verify README.md mentions xFusionCorp
cat /root/code/churn-model/README.md
grep "xFusionCorp" /root/code/churn-model/requirements.txt 2>/dev/null
grep "xFusionCorp" /root/code/churn-model/README.md
grep "scikit-learn" /root/code/churn-model/requirements.txt
```

Expected results:
- `requirements.txt` should contain `scikit-learn` (since `ml_framework=sklearn`)
- `README.md` should mention `churn-model` (project_name) and `xFusionCorp` (default author)
- Directories `data/`, `models/`, `src/`, `tests/` should exist

---

## 🧠 Key Concepts Learned

| Concept | Detail |
|---------|--------|
| `cookiecutter.json` | Defines template variables and their defaults |
| Choice variables | Use JSON arrays: `["option1", "option2"]` |
| Template dir name | Must be `{{cookiecutter.project_name}}` exactly |
| Jinja2 in templates | Use `{{ var }}` for substitution, `{% if %}` for logic |
| `--no-input` flag | Skips interactive prompts, uses defaults or CLI overrides |
| `-o` flag | Specifies output directory for generated project |

---

## ⚠️ Common Pitfalls

1. **Wrong package names**: `sklearn` ≠ `scikit-learn`, `pytorch` ≠ `torch`
2. **Jinja2 whitespace**: Use `-%}` to strip trailing newlines
3. **Empty directories**: Cookiecutter skips empty dirs — add `.gitkeep` files
4. **JSON syntax**: Trailing commas in `cookiecutter.json` break parsing
5. **Template dir typos**: Even one wrong character in `{{cookiecutter.project_name}}` will fail
