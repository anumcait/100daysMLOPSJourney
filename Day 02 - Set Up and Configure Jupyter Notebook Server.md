# Day 2: Set Up and Configure Jupyter Notebook Server

## Task Description
The xFusionCorp Industries data science team needs a properly configured JupyterLab server. An existing configuration exists, but it has several issues preventing it from working correctly within the lab environment.

### Requirements:
1. **Port**: The server must listen on port `8888`.
2. **Binding**: The server must bind to `0.0.0.0` (to be reachable via proxy).
3. **Root Directory**: The notebook root directory must be `/root/notebooks/`.
4. **Interface**: The Jupyter UI button must open the classic notebook interface (`/tree`).
5. **Automation**: Ensure the root directory exists and start the server using the corrected configuration.

## Solution

### 1. Diagnosis and Configuration Fixes
We need to modify `/root/code/jupyter_lab_config.py`. Below are the necessary corrections:

| Requirement | Setting in `jupyter_lab_config.py` | Correct Value |
| :--- | :--- | :--- |
| Bind IP | `c.ServerApp.ip` | `'0.0.0.0'` |
| Port | `c.ServerApp.port` | `8888` |
| Root Directory | `c.ServerApp.root_dir` | `'/root/notebooks/'` |
| Default URL | `c.ServerApp.default_url` | `'/tree'` |

### 2. Implementation Steps

```bash
# 1. Create the missing notebooks directory
mkdir -p /root/notebooks/

# 2. Activate the virtual environment
source /root/code/ml-env/bin/activate

# 3. Start JupyterLab with the configuration file
jupyter lab --config=/root/code/jupyter_lab_config.py --allow-root --no-browser &
```

### 3. Verification
After starting the server, verify it is listening on the correct port:
```bash
netstat -tuln | grep 8888
```

The server should now be accessible, and clicking the Jupyter logo should redirect to the `/tree` endpoint.
