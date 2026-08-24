# Day 86 — Argo Workflow DAG Dependency Fix
The xFusionCorp Industries ML platform team has submitted an initial training workflow to Argo — a three-step Directed Acyclic Graph (DAG), data-prep → train → evaluate, whose steps share a workspace volume. This workflow is triggered on every push; however, the current execution is marked red because one of the steps races ahead of its upstream dependency. Your task is to review the broken DAG displayed in the Argo UI, submit a corrected workflow using the YAML editor in the UI, and observe the new execution transition to a successful green Succeeded status.


The Argo UI button at the top of the lab opens the Workflows page on port 5000. The Workflows list holds the pre-submitted run (training-pipeline-<suffix>) in a Failed state. Open that run and click the red node to read its logs and the DAG graph — that is where the failure surfaces.

The workflow is a dag template of three tasks that each run in their own pod and share a workspace volume; the three tasks do not run in the order the pipeline needs. The same spec is staged at /root/code/pipelines/training-workflow.yaml.

After the corrected workflow is submitted through the UI's YAML editor, the DAG should run data-prep → train → evaluate in order, all three nodes turning green with the workflow phase Succeeded.

The end state must include:

At least two workflows in namespace argo (the original broken run plus the fixed resubmit).
The corrected workflow's main DAG gives the evaluate task dependencies: [train], so the three steps run in order rather than in parallel.
The most recent workflow's status.phase == Succeeded (tests wait up to 240 s).
An Argo dag template starts each task the moment its dependencies are satisfied, and a task with no dependencies runs immediately. Declaring the right dependencies is what turns independently-scheduled pods into an ordered pipeline — without them, a task races its inputs, which is a classic cause of a red DAG run.

## Objective

Fix the broken Argo training workflow so that the three DAG tasks execute in the correct order:

`data-prep → train → evaluate`

The workflow uses a shared workspace volume.

## Problem

The original workflow allowed tasks to run without the required upstream dependencies. This caused `evaluate` to potentially run before `train` had created `/workdir/model.pkl`.

The correct dependency chain is:

- `data-prep` has no dependency and starts immediately.
- `train` depends on `data-prep`.
- `evaluate` depends on `train`.

## Corrected Workflow

The important DAG section is:

    tasks:
      - name: data-prep
        template: data-prep

      - name: train
        template: train
        dependencies:
          - data-prep

      - name: evaluate
        template: evaluate
        dependencies:
          - train

## Complete Workflow YAML

    apiVersion: argoproj.io/v1alpha1
    kind: Workflow
    metadata:
      generateName: training-pipeline-
      namespace: argo
      labels:
        lab: day86
    spec:
      entrypoint: main
      volumeClaimTemplates:
        - metadata:
            name: workdir
          spec:
            accessModes: ["ReadWriteOnce"]
            resources:
              requests:
                storage: 64Mi

      templates:
        - name: main
          dag:
            tasks:
              - name: data-prep
                template: data-prep

              - name: train
                template: train
                dependencies:
                  - data-prep

              - name: evaluate
                template: evaluate
                dependencies:
                  - train

        - name: data-prep
          container:
            image: alpine:3.19
            command: [sh, -c]
            args:
              - |
                echo "[data-prep] preparing data"
                sleep 2
                echo 'rows=100' > /workdir/data.txt
                echo "[data-prep] done"
            volumeMounts:
              - name: workdir
                mountPath: /workdir

        - name: train
          container:
            image: alpine:3.19
            command: [sh, -c]
            args:
              - |
                echo "[train] training model"
                sleep 5
                echo 'model-v1' > /workdir/model.pkl
                echo "[train] done"
            volumeMounts:
              - name: workdir
                mountPath: /workdir

        - name: evaluate
          container:
            image: alpine:3.19
            command: [sh, -c]
            args:
              - |
                echo "[evaluate] checking model"
                if [ ! -f /workdir/model.pkl ]; then
                  echo "[evaluate] ERROR: model.pkl not found"
                  exit 1
                fi
                cat /workdir/model.pkl
                echo "[evaluate] done"
            volumeMounts:
              - name: workdir
                mountPath: /workdir

## Git Attempt

The workflow file is located at:

    /root/code/pipelines/training-workflow.yaml

The attempted commands were:

    cd /root/code/pipelines
    git add training-workflow.yaml
    git commit -m "Fix training workflow DAG dependencies"

Git returned:

    fatal: not a git repository (or any of the parent directories): .git

The `/root/code/pipelines` directory is not a Git repository in this environment. Do not run `git init` unless the lab specifically instructs you to create a repository.

## Argo UI Submission

1. Open the Argo Workflows UI from the lab button.
2. Open the original `training-pipeline-<suffix>` workflow.
3. Confirm that the original workflow is in `Failed` state.
4. Open the YAML editor for submitting a new workflow.
5. Paste the corrected workflow YAML.
6. Submit the workflow.
7. Watch the DAG execution.

Expected order:

    data-prep → train → evaluate

## Expected Final State

The `argo` namespace should contain at least two workflows:

1. The original broken workflow in `Failed` state.
2. The corrected workflow in `Succeeded` state.

The corrected workflow must have:

    train → depends on data-prep
    evaluate → depends on train

The most recent workflow should have:

    status.phase == Succeeded

All three DAG nodes should turn green:

    data-prep: Succeeded
    train: Succeeded
    evaluate: Succeeded

## Key Concept

An Argo DAG task with no dependencies starts immediately.

A task with dependencies starts only after its dependencies are satisfied.

Therefore:

    data-prep → train → evaluate

creates the required ordered ML pipeline.

Without these dependencies, the pods can be scheduled independently. `evaluate` may run before `train` creates `/workdir/model.pkl`, causing the workflow to fail.

## Final Verification

In the Argo UI, verify:

    Workflow: Succeeded
    data-prep: Succeeded
    train: Succeeded
    evaluate: Succeeded

The final DAG should show all three nodes in green and in the correct dependency order.

### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a212e26b-07f1-4ddd-ba8f-b0a3dfc64691" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/457e1d8c-102d-4307-8302-885e6c5cd6c5" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/c50367ae-6d5b-4d62-81b7-34318667ac39" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/8fb0a571-3626-4f8a-867e-6a18c595bfc2" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/7e12aadf-0f50-4ea2-9995-f76961367727" />




