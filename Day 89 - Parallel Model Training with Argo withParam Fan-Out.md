# Day 89: Parallel Model Training with Argo withParam Fan-Out
The xFusionCorp Industries ML platform team aims to train multiple model variants simultaneously and select the best performing model. The train-parallel-variants WorkflowTemplate has been created but is incomplete: the train-variant and pick-best step templates are defined, yet the main template lacks its withParam fan-out and the pick-best fan-in step. Your task is to complete both components: fan out the train-variant step over the estimators_list parameter using withParam, and incorporate the pick-best reducer as a second step. After applying the template, submit it twice through the Argo UI: first, with a deliberately faulty entry (ensuring one branch fails), and then with a valid list (confirming both the fan-out and reducer complete successfully).


The scaffolded template is at /root/code/argo/train-parallel-variants.yaml. It is NOT applied yet — the Argo UI's Workflow Templates list (port 5000) stays empty until the template is applied to the argo namespace. The train-variant template validates its n_estimators input as a positive integer and exits 1 otherwise.

Two TODO-marked pieces are missing from the main template:

TODO 1 — fan-out: the train step runs only once instead of fanning out over the estimators_list parameter, so it never becomes the N parallel pods the sweep needs.
TODO 2 — fan-in: the reducer step group that runs after the parallel branches finish is absent, so nothing picks the best result once the variants complete.
With the template applied and appearing in the UI, submitting it twice demonstrates the fan-out and its failure mode: an estimators_list with one obviously-bad entry alongside valid positive integers red-lights the bad branch while the others go green, leaves pick-best Omitted, and finishes Failed; a clean list of three valid positive integers turns every train-variant branch green, runs pick-best, and finishes Succeeded.

The end state must include:

GET /api/v1/workflow-templates/argo/train-parallel-variants returns the template, whose main step fans out over estimators_list with withParam and includes a pick-best fan-in step.
At least two workflows exist with spec.workflowTemplateRef.name == train-parallel-variants.
The most recent workflow's status.phase == Succeeded, with ≥3 train-variant Pod nodes all Succeeded plus one pick-best node Succeeded.
withParam is Argo's fan-out primitive—one step definition, N parallel pods, one template per input value. Because every item runs independently, one bad value does not stop the others; it only blocks the fan-in reducer (pick-best) from receiving a complete set. That isolation is both the pattern's value (a 99-of-100 sweep still gives you 99 models) and its failure mode (one bad row in the input list red-lights the release).

## Objective

Complete the `train-parallel-variants` Argo WorkflowTemplate so that:

- `train-variant` fans out over `estimators_list` using `withParam`.
- Each list item is passed to `train-variant` as `n_estimators`.
- `pick-best` runs as a second step group after the fan-out.
- A faulty input demonstrates isolated branch failure.
- A valid input demonstrates successful fan-out and fan-in.

## Environment

- Argo Workflows: v4.0.4
- Namespace: `argo`
- WorkflowTemplate: `train-parallel-variants`
- Manifest: `/root/code/argo/train-parallel-variants.yaml`

## Changes Made

### TODO 1 — Fan-out

The `train` step was changed from:

```yaml
- - name: train
    template: train-variant
```

to:

```yaml
- - name: train
    template: train-variant
    arguments:
      parameters:
        - name: n_estimators
          value: "{{item}}"
    withParam: "{{workflow.parameters.estimators_list}}"
```

`withParam` is Argo's fan-out primitive. It takes the JSON array from `estimators_list` and creates one parallel execution for each item.

For example:

```json
["10","50","100"]
```

creates three `train-variant` executions.

### TODO 2 — Fan-in

The reducer was added as a second step group:

```yaml
- - name: pick-best
    template: pick-best
```

Because it is the second step group, Argo waits for the parallel `train` branches before executing `pick-best`.

## Completed Main Template

The relevant `main` template is:

```yaml
- name: main
  steps:
    - - name: train
        template: train-variant
        arguments:
          parameters:
            - name: n_estimators
              value: "{{item}}"
        withParam: "{{workflow.parameters.estimators_list}}"

    - - name: pick-best
        template: pick-best
```

## Apply the Template

The completed WorkflowTemplate was applied with:

```bash
kubectl apply -f /root/code/argo/train-parallel-variants.yaml -n argo
```

Output:

```text
workflowtemplate.argoproj.io/train-parallel-variants created
```

Verification:

```bash
kubectl get workflowtemplate train-parallel-variants -n argo
```

Output confirmed:

```text
NAME                      AGE
train-parallel-variants   ...
```

The applied YAML also confirmed:

```yaml
withParam: '{{workflow.parameters.estimators_list}}'
```

and:

```yaml
- - name: pick-best
    template: pick-best
```

## Faulty Submission

The first workflow was submitted with:

```json
["10","bad","100"]
```

The Argo UI showed three independent fan-out branches:

```text
train(0:10)   ✓
train(1:bad)  ✗
train(2:100)  ✓
```

This confirmed that `withParam` successfully created three parallel executions.

The invalid `bad` value caused its branch to fail because `train-variant` validates `n_estimators` as numeric.

The expected behavior for this run is:

- Valid branches succeed.
- Invalid branch fails.
- `pick-best` is omitted.
- Overall workflow finishes `Failed`.

## Successful Submission

The second workflow was submitted with:

```json
["10","50","100"]
```

The Argo UI showed:

```text
train(0:10)   ✓
train(1:50)   ✓
train(2:100)  ✓
pick-best     running
```

The three training branches all succeeded, demonstrating the fan-out.

The `pick-best` reducer then ran after the parallel branches, demonstrating the fan-in.

The reducer template executes:

```yaml
- name: pick-best
  container:
    image: alpine:3.19
    command: [sh, -c]
    args: ["echo '[pick_best] selected: model-n100'"]
```

After completion, the expected final state is:

```text
train(0:10)    ✓
train(1:50)    ✓
train(2:100)   ✓
pick-best      ✓
```

with the overall workflow status:

```text
Succeeded
```

## Key Concept

`withParam` allows one step definition to fan out into N parallel pods, one for each input value.

For example:

```yaml
withParam: "{{workflow.parameters.estimators_list}}"
```

with:

```json
["10","50","100"]
```

creates three independent training branches.

A failure in one branch does not prevent the other branches from running. However, the failed branch prevents the reducer from receiving a complete successful set, so `pick-best` is omitted and the workflow fails.

This demonstrates both the value and failure mode of parallel model sweeps:

- A `99-of-100` sweep can still produce 99 successful models.
- A single bad input can cause the overall workflow release to fail.
- The reducer only runs successfully when the required fan-out branches complete successfully.

## Final Expected End State

The WorkflowTemplate should be available as:

```text
train-parallel-variants
```

At least two workflows should reference:

```yaml
spec:
  workflowTemplateRef:
    name: train-parallel-variants
```

The most recent workflow should have:

```text
status.phase = Succeeded
```

and its nodes should include:

- At least three `train-variant` nodes — `Succeeded`
- One `pick-best` node — `Succeeded`

The completed exercise demonstrates:

**Argo `withParam` fan-out → parallel model training → reducer fan-in → best model selection**
