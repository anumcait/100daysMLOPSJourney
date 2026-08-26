# Day 88: Fix a Missing @task Decorator in a Prefect Flow
The xFusionCorp Industries ML platform team is piloting Prefect 3.x as a second orchestrator alongside Argo Workflows. A teammate wired up a fraud-pipeline deployment with three steps—prep, train, evaluate—but on every run the Prefect Flow Run graph only shows two tracked nodes (prep and train); evaluate is missing. Your task is to fix the flow source, redeploy, trigger a new run from the Deployments page, and confirm the 3-node DAG.


The Prefect UI button at the top of the lab opens the UI on port 5000. Its Deployments page lists fraud-pipeline. Trigger a Quick Run and open the resulting Flow Run: its DAG renders only two nodes, and comparing that graph against the flow source is where the gap shows up.

The flow source is at /root/code/prefect/fraud_pipeline.py; the evaluate function runs as part of the flow but does not surface in the run graph the way the other two functions do. A shipped Makefile in that directory wraps the kill + restart cycle needed for the serve loop to pick up the new source (/var/log/prefect-serve.log confirms the new process is up).

After redeploying and triggering a fresh Quick Run, the Flow Run's DAG should render three nodes — prep → train → evaluate — each reaching Completed.

The end state must include:

Prefect's /api/deployments/name/fraud-pipeline/fraud-pipeline returns the deployment.
At least one Completed flow run under that deployment has three task runs whose names are exactly prep, train, and evaluate — Prefect's run graph now records evaluate alongside the other two.
Prefect's flow-run graph is built from the task-run records its orchestrator emits during execution. A function that runs as part of the flow but is not registered as a task disappears from the run graph entirely — it executes, but the orchestrator has no record of it.

## Issue

The `evaluate` function was executing as part of the Prefect flow but was not appearing in the Flow Run graph because it was not registered as a Prefect task.

A duplicate `evaluate()` definition also caused the decorated version to be overwritten.

## Fix

The final `fraud_pipeline.py` uses `@task(name="evaluate")` and contains only one `evaluate()` definition:

```python
@task(name="evaluate")
def evaluate(model: str) -> float:
    print(f"[evaluate] scoring model {model}")
    return 0.75
```

## Redeploy

From `/root/code/prefect`:

```bash
make
```

The Makefile redeploys the `fraud-pipeline` deployment and restarts the serve process.

## Verification

Open the Prefect UI on port 5000:

1. Go to **Deployments → fraud-pipeline**.
2. Trigger **Quick Run**.
3. Open the newest Flow Run.
4. Confirm the DAG contains:
   `prep → train → evaluate`
5. Confirm all three task runs are **Completed**.

The final deployment should have a completed Flow Run containing exactly these task runs:

- `prep`
- `train`
- `evaluate`

### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/b0f15317-4255-4643-a464-cb7a9800cfdf" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/6ab7e9b4-3e64-4a49-af78-6ec71897ad49" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/16536bd2-a923-47a9-8401-cffb2ce86c31" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/89092113-c788-49cc-bf61-bc4b026641f3" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/828a97e7-11e4-4853-b5f3-2fe0ed56e5b8" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/d37d8e90-a4b5-4462-bb59-c81ee9696e63" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a2e16d89-06fc-473a-9e8e-d01af8212265" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/9abe7e22-4986-481d-9ba2-1f69f8faddd8" />








