# Day 87: Pass Data Between Argo Steps with Output Parameters and Branching
The xFusionCorp Industries ML platform team requires a reusable training pipeline that promotes a model to the registry only when it passes a configurable quality gate. This pipeline should utilize the same WorkflowTemplate with differing min_score values for each run. Currently, a train-and-maybe-register WorkflowTemplate is available on disk; however, it is not yet functional. The evaluate step computes a synthetic score of 0.75 but does not publish it, and the register step lacks a quality gate, causing it to execute unconditionally. Your task is to complete the template by publishing the score as an output parameter and implementing a when: expression to gate the register step, ensuring that it compares the score to min_score. Once you have applied these changes, submit the workflow twice from the Argo UI to demonstrate the functionality of the gate: first with a threshold that prevents register from executing, and then with a threshold that allows it to execute.


The scaffolded template is at /root/code/argo/train-and-maybe-register.yaml. It is NOT applied yet — the Argo UI's Workflow Templates list (port 5000) stays empty until the template is applied to the argo namespace as a cluster resource.

Two TODO-marked pieces are missing from the YAML:

TODO 1 — evaluate template: the script writes a synthetic score to /tmp/score.txt, but the template does not yet expose that value to the rest of the workflow as an output parameter.
TODO 2 — register step in main: the register step currently runs unconditionally, rather than only when the score clears the min_score threshold.
With the template applied and appearing in the UI, submitting it twice demonstrates the gate: a run with min_score above the evaluate score (e.g. 0.99) reaches Succeeded with the register node Skipped (the when: evaluated false), while a run with min_score below the score (e.g. 0.5) has register Succeeded.

The end state must include:

GET /api/v1/workflow-templates/argo/train-and-maybe-register returns the template, whose evaluate template emits a score output parameter and whose register step carries a when: gate referencing both the evaluate score and min_score.
At least two workflows exist whose spec.workflowTemplateRef.name == train-and-maybe-register.
One with min_score > 0.75 has its register node Skipped/Omitted; another with min_score <= 0.75 has register Succeeded.
This is the canonical CI/CT gate: every commit runs train + evaluate, but only commits that clear the configurable threshold promote the model. Passing the score between steps as an output parameter—and gating promotion with when:—is what lets one template serve dev (min_score=0.5), staging (0.75), and prod (0.9) without a single rewrite.

## Objective

Complete the reusable Argo `WorkflowTemplate` so that:

- The `evaluate` step publishes its synthetic score as an output parameter.
- The `register` step runs only when the score meets the configurable `min_score`.
- The same `WorkflowTemplate` can be submitted with different quality thresholds.
- A run with `min_score=0.99` skips registration.
- A run with `min_score=0.5` successfully registers the model.

## Starting Point

The scaffolded template was located at:

`/root/code/argo/train-and-maybe-register.yaml`

The template contained two TODOs.

### TODO 1 — Publish the evaluation score

The `evaluate` script writes:

`0.75`

to:

`/tmp/score.txt`

The file needed to be exposed as an Argo output parameter named `score`.

### TODO 2 — Add the quality gate

The `register` step originally ran unconditionally.

It needed a `when:` expression comparing:

`steps.evaluate.outputs.parameters.score`

against:

`workflow.parameters.min_score`

## Completed YAML

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: train-and-maybe-register
  namespace: argo
spec:
  entrypoint: main
  arguments:
    parameters:
      - name: min_score
        value: "0.80"
  templates:
    - name: main
      steps:
        - - name: train
            template: train
        - - name: evaluate
            template: evaluate
        - - name: register
            template: register
            when: "{{steps.evaluate.outputs.parameters.score}} >= {{workflow.parameters.min_score}}"

    - name: train
      container:
        image: alpine:3.19
        command: [sh, -c]
        args: ["echo '[train] fitting model' && sleep 2"]

    - name: evaluate
      script:
        image: alpine:3.19
        command: [sh]
        source: |
          # Deterministic score -- MLOps-not-ML. Teaching point is the
          # parameter plumbing, not the score itself.
          echo "0.75" > /tmp/score.txt
          echo "[evaluate] score=0.75"
      outputs:
        parameters:
          - name: score
            valueFrom:
              path: /tmp/score.txt

    - name: register
      container:
        image: alpine:3.19
        command: [sh, -c]
        args: ["echo '[register] promoting model to registry'"]
```

## Key Changes

### Output Parameter

The `evaluate` template now contains:

```yaml
outputs:
  parameters:
    - name: score
      valueFrom:
        path: /tmp/score.txt
```

This makes the score available to subsequent steps as:

`steps.evaluate.outputs.parameters.score`

### Conditional Registration

The `register` step now contains:

```yaml
when: "{{steps.evaluate.outputs.parameters.score}} >= {{workflow.parameters.min_score}}"
```

The workflow therefore evaluates the quality gate dynamically for every run.

## Apply the Template

The template was applied with:

```bash
kubectl apply -f /root/code/argo/train-and-maybe-register.yaml -n argo
```

Result:

```text
workflowtemplate.argoproj.io/train-and-maybe-register created
```

The template was then verified:

```bash
kubectl get workflowtemplate -n argo train-and-maybe-register
```

Result:

```text
NAME                       AGE
train-and-maybe-register   ...
```

The live resource confirmed both required changes:

```yaml
outputs:
  parameters:
    - name: score
      valueFrom:
        path: /tmp/score.txt
```

and:

```yaml
when: '{{steps.evaluate.outputs.parameters.score}} >= {{workflow.parameters.min_score}}'
```

## Test Run 1 — Quality Gate Fails

The WorkflowTemplate was submitted from the Argo UI with:

```text
min_score = 0.99
```

The evaluate step produced:

```text
0.75
```

The `register` node showed:

```text
TYPE
Skipped

PHASE
Skipped

MESSAGE
when '0.75 >= 0.99' evaluated false
```

This confirms that the quality gate prevented model registration.

Expected result:

```text
train       Succeeded
evaluate    Succeeded
register    Skipped
workflow    Succeeded
```

## Test Run 2 — Quality Gate Passes

The same WorkflowTemplate was submitted again from the Argo UI with:

```text
min_score = 0.5
```

The evaluate step again produced:

```text
0.75
```

The `register` node showed:

```text
TYPE
Pod

PHASE
Succeeded
```

The workflow graph showed all three steps succeeding:

```text
train       Succeeded
evaluate    Succeeded
register    Succeeded
```

The gate therefore evaluated successfully:

```text
0.75 >= 0.5
```

## Final Behavior

The reusable pipeline now behaves as a configurable CI/CT promotion gate:

```text
                ┌──────────┐
                │  train   │
                └────┬─────┘
                     │
                     ▼
                ┌──────────┐
                │ evaluate │
                │  score   │
                │   0.75   │
                └────┬─────┘
                     │
                     ▼
          score >= min_score?
                /           \
              yes            no
               │              │
               ▼              ▼
          ┌──────────┐     SKIPPED
          │ register │
          └──────────┘
```

Examples:

```text
min_score = 0.50
0.75 >= 0.50 → true
register → Succeeded
```

```text
min_score = 0.75
0.75 >= 0.75 → true
register → Succeeded
```

```text
min_score = 0.90
0.75 >= 0.90 → false
register → Skipped
```

```text
min_score = 0.99
0.75 >= 0.99 → false
register → Skipped
```

## Verification Checklist

- [x] `WorkflowTemplate` created in the `argo` namespace.
- [x] `evaluate` publishes the score as an output parameter.
- [x] Output parameter is named `score`.
- [x] Output parameter reads from `/tmp/score.txt`.
- [x] `register` has a `when:` expression.
- [x] `when:` references `steps.evaluate.outputs.parameters.score`.
- [x] `when:` references `workflow.parameters.min_score`.
- [x] Run with `min_score=0.99` completed with `register` skipped.
- [x] Run with `min_score=0.5` completed with `register` succeeded.
- [x] The same WorkflowTemplate was reused for both runs.

## Key Takeaway

Argo output parameters allow data generated by one step to be passed to later steps. The `when:` expression then uses that value together with a workflow parameter to implement a configurable quality gate.

This allows a single reusable template to support different environments without rewriting the pipeline:

```text
Development → min_score=0.5
Staging     → min_score=0.75
Production  → min_score=0.9
```

Every commit can therefore run the training and evaluation stages, while only models that clear the configured quality threshold are promoted to the registry.

### Screenshots
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/3f3ea84b-cc34-4102-93a5-29dd5cc058bd" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/126e98ac-4065-4093-990f-ca12bd590f54" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/4f78749c-6dba-4a98-be84-d702fb246b82" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/54fd8b72-68e3-45d9-ab85-a41938d13ca3" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a526f37e-6503-450d-b286-5ec0aeae2044" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/3a6bca65-6c16-4200-a62c-cc1a3ba39df9" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/0a99fdbe-232a-466c-ba9f-e26f819636a8" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5d88eb41-1948-46c2-85b9-5c6adc0b97dc" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/6a91ff5c-6843-4e78-a949-e1bc9436526b" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/89020717-39cb-4407-997c-4a82d5da50fd" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/78b5888c-0062-4c4d-9da0-a599f75603b4" />










