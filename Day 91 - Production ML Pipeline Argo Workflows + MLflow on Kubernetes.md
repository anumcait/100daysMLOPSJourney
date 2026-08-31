# Day 91: Production ML Pipeline — Argo Workflows + MLflow on Kubernetes
The xFusionCorp Industries ML platform team is cutting the first production release of the fraud-detector pipeline. A WorkflowTemplate named fraud-training-pipeline trains a model and registers it in MLflow; a CronWorkflow named fraud-retraining re-runs the template every minute. Argo and an in-cluster MLflow are running, but the release is broken on three fronts. Your capstone task is to fix all three bugs entirely through the Argo UI and confirm that a new version of fraud-detector appears on the MLflow Models page.


Two surfaces are exposed: the Argo UI (port 5000) — Workflows list, Workflow Templates, and Cron Workflows — and the MLflow UI (port 5001), whose Models page is empty since no version of fraud-detector has been registered yet.

Three independent wiring issues sit between the run logs, the fraud-training-pipeline WorkflowTemplate spec, and the fraud-retraining CronWorkflow spec. They surface progressively: the first submission of the template is rejected outright with a parameter-resolution error (a bad output-parameter reference); once that is fixed a node fails at runtime, visible in its logs; and the Cron Workflows page reveals what fraud-retraining is failing to do over the last few minutes. Each fix is a single value change made through the Argo UI's YAML editors (the template's or the cron's Edit view).

With all three fixed, a fresh submission of fraud-training-pipeline should run green end-to-end, fraud-retraining should spawn a green child workflow within a minute, and the MLflow Models page should show one or more versions of fraud-detector.

The end state must include:

A manual submission of the fraud-training-pipeline template runs end-to-end to Succeeded (train and register both green on the DAG).
GET /api/2.0/mlflow/registered-models/get?name=fraud-detector returns at least one version (tests poll up to 300 s).
The fraud-retraining CronWorkflow spawns at least one child Workflow that completes successfully — the Cron Workflows page shows it in the resource's Workflows panel (tests look for the owner label workflows.argoproj.io/cron-workflow=fraud-retraining).
Production orchestration breaks across boundaries — a typo or a stale reference can survive a reviewer's read of any single resource. The capstone is reading them as symptoms on a running system, fixing them in place, and confirming the full pipe is back to passing.

## Objective

Fix the three wiring issues in the production fraud-detector ML pipeline using only the Argo UI, then verify that:

- `fraud-training-pipeline` runs successfully end-to-end.
- `train` and `register` both complete successfully.
- `fraud-detector` is registered in MLflow.
- `fraud-retraining` successfully references the correct WorkflowTemplate.
- The CronWorkflow spawns successful child workflows.

## Environment

- Argo UI: port `5000`
- MLflow UI: port `5001`
- Argo namespace: `argo`
- WorkflowTemplate: `fraud-training-pipeline`
- CronWorkflow: `fraud-retraining`
- MLflow registered model: `fraud-detector`

## Bugs Found and Fixed

### Bug 1 — Incorrect output parameter reference

The `train` template defines this output parameter:

```yaml
outputs:
  parameters:
    - name: run_id
      valueFrom:
        path: /tmp/run_id
```

However, the `register` step originally referenced:

```yaml
{{steps.train.outputs.parameters.runid}}
```

Argo rejected the WorkflowTemplate with:

```text
templates.main.steps failed to resolve {{steps.train.outputs.parameters.runid}}
```

### Fix

Changed:

```yaml
value: "{{steps.train.outputs.parameters.runid}}"
```

to:

```yaml
value: "{{steps.train.outputs.parameters.run_id}}"
```

The parameter name now matches the output declared by the `train` template.

---

### Bug 2 — Incorrect MLflow service hostname

The original MLflow tracking URI was:

```yaml
http://mlflow.default.svc.cluster.local:5000
```

The training container failed with:

```text
NameResolutionError: Failed to resolve 'mlflow.default.svc.cluster.local'
```

The first attempted correction to:

```yaml
http://mlflow.argo.svc.cluster.local:5000
```

also failed with:

```text
NameResolutionError: Failed to resolve 'mlflow.argo.svc.cluster.local'
```

The working MLflow service address was:

```yaml
http://mlflow.mlflow.svc.cluster.local:5000
```

### Fix

Updated `MLFLOW_TRACKING_URI` in both the `train` and `register` templates:

```yaml
env:
  - name: MLFLOW_TRACKING_URI
    value: http://mlflow.mlflow.svc.cluster.local:5000
```

This allowed the training container to connect successfully to MLflow.

---

### Bug 3 — CronWorkflow referenced the wrong WorkflowTemplate

The `fraud-retraining` CronWorkflow originally contained:

```yaml
workflowTemplateRef:
  name: training-pipeline
```

But the actual WorkflowTemplate is:

```yaml
metadata:
  name: fraud-training-pipeline
```

The CronWorkflow showed:

```text
Last Scheduled Time: -
No completed cron workflows
```

### Fix

Changed:

```yaml
workflowTemplateRef:
  name: training-pipeline
```

to:

```yaml
workflowTemplateRef:
  name: fraud-training-pipeline
```

The CronWorkflow schedule remained:

```yaml
schedules:
  - "* * * * *"
```

so it runs every minute.

## Final Working Configuration

The important WorkflowTemplate wiring is:

```yaml
steps:
  - - name: train
      template: train
  - - name: register
      template: register
      arguments:
        parameters:
          - name: run_id
            value: "{{steps.train.outputs.parameters.run_id}}"
```

The `train` template outputs:

```yaml
outputs:
  parameters:
    - name: run_id
      valueFrom:
        path: /tmp/run_id
```

Both templates use:

```yaml
MLFLOW_TRACKING_URI=http://mlflow.mlflow.svc.cluster.local:5000
```

The CronWorkflow references:

```yaml
workflowTemplateRef:
  name: fraud-training-pipeline
```

## Verification

After applying the three fixes through the Argo UI:

- Manual `fraud-training-pipeline` execution completed successfully.
- The training step successfully connected to MLflow.
- The registration step completed successfully.
- `fraud-detector` was registered in MLflow.
- `fraud-retraining` was corrected to reference `fraud-training-pipeline`.
- The pipeline is now wired correctly across Argo Workflows, the CronWorkflow, and MLflow.

## Key Lesson

Production workflow failures can occur across resource boundaries:

1. Argo parameter names must exactly match declared output parameters.
2. Kubernetes service DNS names must use the correct namespace.
3. CronWorkflows must reference the exact name of the WorkflowTemplate.

A resource can look correct in isolation while still failing because of a stale or incorrect reference to another resource.

## Result

**Day 91 completed successfully.**

The production fraud-detector pipeline is now functioning end-to-end:

```text
CronWorkflow
    ↓
fraud-training-pipeline
    ↓
train
    ↓
MLflow run + model artifact
    ↓
register
    ↓
fraud-detector model version
```
### Screenshots

