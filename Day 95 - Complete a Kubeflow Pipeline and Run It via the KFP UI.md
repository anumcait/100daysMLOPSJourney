# Day 95: Complete a Kubeflow Pipeline and Run It via the KFP UI

The xFusionCorp Industries ML platform team is piloting Kubeflow Pipelines on their kind cluster, with the KFP web UI exposed on port 5000. A two-component pipeline source (prep_data → train) is staged at /root/code/kfp/pipeline.py, but its pipeline function wires only the first component. Your task is to complete the DAG so train runs after prep_data, compile it to an IR YAML with the KFP SDK, upload it through the KFP UI as fraud-training, then create and run it from the Default experiment and confirm the run reaches Succeeded.


Kubeflow Pipelines is running on the kind cluster and its UI is exposed via the KFP UI button at the top of the lab (port 5000). A two-component pipeline source is staged at /root/code/kfp/pipeline.py: the @dsl.component functions prep_data and train are written, but fraud_training_pipeline wires only prep_data — inspect the pipeline function to see what is missing.

Compile the source to an IR YAML (the KFP SDK is installed):

cd /root/code/kfp && python3 pipeline.py

The KFP UI's file picker reads from your local machine, not the lab container, so download the compiled pipeline.yaml from the VS Code Explorer before uploading it through the UI.

The end state must include:

The KFP UI is reachable on :5000.
/root/code/kfp/pipeline.py's pipeline function wires the train component after prep_data.
GET /apis/v2beta1/pipelines returns a pipeline named fraud-training.
At least one run from that pipeline reaches state SUCCEEDED (tests poll up to 420 s).
KFP compiles each @dsl.component into one container-per-step; the @dsl.pipeline function is the DAG that wires them, and python3 pipeline.py runs the compiler to produce the IR YAML the KFP UI executes. Components run in parallel unless an explicit ordering edge declares a dependency.

## Objective

Complete the two-component Kubeflow Pipelines (KFP) DAG so that `train` runs after `prep_data`, compile the pipeline to an IR YAML file, upload it to the KFP UI as `fraud-training`, and run it from the **Default** experiment until the run reaches **Succeeded**.

## Environment

- Kubeflow Pipelines running on a kind cluster
- KFP UI exposed on port `5000`
- Source file: `/root/code/kfp/pipeline.py`
- Compiled artifact: `/root/code/kfp/pipeline.yaml`
- Pipeline name: `fraud-training`
- Experiment: `Default`

## 1. Inspect the Pipeline Source

The source already defines two components:

- `prep_data`
- `train`

The pipeline function originally wired only `prep_data`:

```python
@dsl.pipeline(
    name=PIPELINE_NAME,
    description="Synthetic two-step training pipeline for the KFP lab.",
)
def fraud_training_pipeline():
    prep = prep_data()
    # TODO: Complete the DAG.
```

Because the components do not have input/output parameters, the dependency is declared explicitly with `.after(prep)`.

## 2. Complete the DAG

The completed pipeline function is:

```python
@dsl.pipeline(
    name=PIPELINE_NAME,
    description="Synthetic two-step training pipeline for the KFP lab.",
)
def fraud_training_pipeline():
    prep = prep_data()
    train().after(prep)
```

This creates the required execution order:

```text
prep_data -> train
```

The `.after(prep)` call is important because KFP components otherwise have no ordering relationship and may run in parallel.

## 3. Compile the Pipeline

Run:

```bash
cd /root/code/kfp
python3 pipeline.py
```

Expected output:

```text
Wrote pipeline.yaml -- upload this file via the KFP UI.
```

The compiler creates:

```text
/root/code/kfp/pipeline.yaml
```

## 4. Download the Compiled YAML

The KFP UI file picker reads files from the local machine rather than directly from the lab container.

Download `/root/code/kfp/pipeline.yaml` through the VS Code Explorer to the local computer.

## 5. Upload the Pipeline in KFP UI

Open the KFP UI on port `5000`.

Go to **Pipelines** and select **Upload pipeline**.

Upload the downloaded `pipeline.yaml` and register it with the name:

```text
fraud-training
```

Confirm that `fraud-training` appears in the pipeline list.

## 6. Create and Run from the Default Experiment

Go to **Experiments** and select the **Default** experiment.

Create a new run using the `fraud-training` pipeline and start it.

The execution graph should show:

```text
prep_data -> train
```

Wait for the run to finish successfully.

## 7. Verify the Pipeline Registration

The KFP API should return a pipeline named `fraud-training`:

```bash
curl -s http://localhost:5000/apis/v2beta1/pipelines
```

The response should contain `fraud-training`.

## 8. Verify the Run

The final run state must be:

```text
SUCCEEDED
```

The lab tests may poll for up to 420 seconds, so allow sufficient time for both component pods to complete.

## Final Checklist

- [x] KFP UI reachable on port `5000`
- [x] `fraud_training_pipeline` wires `train` after `prep_data`
- [x] Pipeline compiled to `pipeline.yaml`
- [x] Pipeline uploaded as `fraud-training`
- [x] Pipeline available in the `Default` experiment
- [x] A run was created from `fraud-training`
- [x] Run reaches `SUCCEEDED`