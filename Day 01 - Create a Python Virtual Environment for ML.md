# Day 1: Create a Python Virtual Environment for ML

## Task Description
The xFusionCorp Industries data science team needs a standardized Python environment for their new ML project. The goal is to set up a virtual environment with the required ML libraries on the controlplane host.

### Requirements:
1. Create a Python virtual environment named `ml-env` under `/root/code/` using `python3 -m venv`.
2. Activate the environment.
3. Install the following packages: `numpy`, `pandas`, `scikit-learn`, and `matplotlib`.
4. Generate a `requirements.txt` file using `pip freeze` and save it at `/root/code/requirements.txt`.

## Solution

### Run these commands on the controlplane host:

```bash
# Create the directory if it doesn't exist
mkdir -p /root/code/

# Create the virtual environment
python3 -m venv /root/code/ml-env

# Activate the virtual environment
source /root/code/ml-env/bin/activate

# Install required ML packages
pip install numpy pandas scikit-learn matplotlib

# Generate requirements.txt
pip freeze > /root/code/requirements.txt
```
### To verify:

```bash
cat /root/code/requirements.txt
```

### You should see entries for:

- numpy
- pandas
- scikit-learn
- matplotlib

### Screenshots
<img width="1575" height="778" alt="image" src="https://github.com/user-attachments/assets/328b292b-da8a-47ae-b4c9-5180f9f60e9b" />
<img width="1588" height="771" alt="image" src="https://github.com/user-attachments/assets/e01044f4-3094-46f5-be3a-a19addf7610d" />


