# Day 85: Submit Your First Argo Workflow
The xFusionCorp Industries ML platform team has established a new Kubernetes cluster for pipeline orchestration. Argo Workflows v4.0.4 is currently operational in the argo namespace, with both the workflow-controller and argo-server Deployments marked as Available. Additionally, the Argo web UI is accessible via port-forwarding in the lab. Your task is to create your first Argo Workflow and submit it using the Argo UI's + Submit New Workflow form. This should be a single-step container that simulates a training step and achieves a status of Succeeded in the Workflows list.


The Argo UI button at the top of the lab opens the Workflows page on port 5000. The UI has no auth (quick-start install) and the Workflows list is empty on first open; new workflows are authored through its + Submit New Workflow form, whose in-browser YAML editor is the canonical authoring surface for this section.

The workflow to author is a kind: Workflow in namespace argo that declares a spec.entrypoint pointing at a template under spec.templates. That template runs a single container whose command/args stand in for a training step — an echo is fine, since this section teaches orchestration, not model quality. Once submitted, the single node should progress Pending → Running → Succeeded.

The end state must include:

GET http://localhost:5000/ returns 200 (Argo UI reachable).
workflow-controller and argo-server Deployments in namespace argo are Available.
At least one Workflow exists in namespace argo, and the most recent one is genuinely authored: it declares spec.entrypoint and a template whose container runs a command/args.
That most recent workflow reaches status.phase == Succeeded (tests wait up to 180 s for a terminal phase).
Argo's + Submit New Workflow flow is how every future lab in this section starts. The UI's YAML editor is the canonical authoring surface—not kubectl apply -f file.yaml from a terminal. Authoring the Workflow spec by hand here—entrypoint, templates, container—is the foundation every WorkflowTemplate, CronWorkflow, and parameterised pipeline in the coming labs builds on.

## Objective

Create and submit your first Argo Workflow using the Argo UI. The workflow should be a single-step container that simulates a training step and reaches `Succeeded`.

## Prerequisites

- Argo Workflows v4.0.4 is installed in the `argo` namespace.
- `workflow-controller` Deployment is Available.
- `argo-server` Deployment is Available.
- The Argo web UI is accessible through port `5000`.
- The Workflows list is initially empty.

## 1. Open the Argo UI

Open the lab's **Argo UI** button.

The UI should be available at:

`http://localhost:5000/`

Confirm that the Workflows page loads successfully.

## 2. Submit the Workflow

Click:

**+ Submit New Workflow**

Use the YAML editor in the UI and enter the following Workflow definition:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: ml-training-
  namespace: argo
spec:
  entrypoint: training
  templates:
    - name: training
      container:
        image: alpine:3.19
        command: [sh, -c]
        args:
          - echo "Simulating ML training step"
```
Click Submit.

## 3. Verify the Workflow
The workflow should progress through:

Pending → Running → Succeeded

Wait until the workflow status is Succeeded.

## 4. Final Verification
- Argo UI returns HTTP 200.
- workflow-controller in namespace argo is Available.
- argo-server in namespace argo is Available.
- At least one Workflow exists in namespace argo.
- The most recent Workflow declares spec.entrypoint.
- The Workflow contains a template under spec.templates.
- The template contains a container with a command and args.
- The most recent Workflow reaches status.phase == Succeeded.

### Important

Do not use kubectl apply -f file.yaml for this exercise.

The canonical authoring method is:

### Argo UI → + Submit New Workflow → YAML editor → Submit

This workflow structure provides the foundation for later Argo concepts such as WorkflowTemplates, CronWorkflows, parameters, and multi-step pipelines.

### Screenshots

