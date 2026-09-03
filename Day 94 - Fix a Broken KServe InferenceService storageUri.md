# Day 94: Fix a Broken KServe InferenceService `storageUri`

The xFusionCorp Industries ML platform team has established KServe on their Kind cluster and deployed a fraud-detector InferenceService, which is backed by a PVC-mounted Scikit-learn model. However, the InferenceService is not reaching a Ready state, as the predictor pod remains in Pending. Please investigate the cause of this issue and rectify the manifest located at /root/code/k8s/inference-service.yaml to ensure that the InferenceService transitions to Ready. After making the necessary adjustments, confirm that the predictor can successfully serve a prediction.


KServe is installed on a kind cluster, and a fraud-detector InferenceService has been deployed — but it never reaches Ready and its predictor pod stays Pending. Inspect the predictor pod to see what is blocking it:

kubectl describe pod -l serving.kserve.io/inferenceservice=fraud-detector

The end state must include:

kubectl get isvc fraud-detector shows the InferenceService.
spec.predictor.model.storageUri references a PVC that exists in the namespace.
.status.conditions[?(@.type=="Ready")].status == True (tests poll up to 360 s).
A prediction request to the predictor's /v1/models/fraud-detector:predict returns a JSON predictions array.
KServe's pvc:// storage scheme mounts the named PVC into the predictor pod at /mnt/models and lets the runtime read model artefacts from there. The reference must name a PVC that exists in the InferenceService's namespace.

## Objective

Fix the `fraud-detector` KServe `InferenceService` because its predictor pod was stuck in `Pending`.

The root cause was an incorrect PVC name in the `storageUri`.

## Initial Manifest

The original `/root/code/k8s/inference-service.yaml` contained:

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: fraud-detector
  annotations:
    serving.kserve.io/deploymentMode: "RawDeployment"
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 1
    model:
      modelFormat:
        name: sklearn
      # storageUri bug: the real PVC is named `model-storage`.
      storageUri: "pvc://models-storage/"
```

The problem was:

```yaml
storageUri: "pvc://models-storage/"
```

The actual PVC is named:

```text
model-storage
```

KServe's `pvc://` storage URI must reference a PVC that exists in the same namespace as the `InferenceService`.

## Fix

Changed the `storageUri` to:

```yaml
storageUri: "pvc://model-storage/"
```

The corrected manifest is:

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: fraud-detector
  annotations:
    serving.kserve.io/deploymentMode: "RawDeployment"
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 1
    model:
      modelFormat:
        name: sklearn
      storageUri: "pvc://model-storage/"
```

## Apply the Fix

```bash
kubectl apply -f /root/code/k8s/inference-service.yaml
```

Output:

```text
inferenceservice.serving.kserve.io/fraud-detector configured
```

## Verify InferenceService Readiness

```bash
kubectl get isvc fraud-detector -w
```

Final status:

```text
NAME             URL   READY   PREV   LATEST   PREVROLLOUTEDREVISION   LATESTREADYREVISION   AGE
fraud-detector         False                                                                 4m4s
fraud-detector   http://fraud-detector-default.example.com   True                                                                  4m17s
fraud-detector   http://fraud-detector-default.example.com   True                                                                  4m17s
```

The important result is:

```text
READY=True
```

The `InferenceService` successfully transitioned to Ready.

## Verify Predictor Pod

```bash
kubectl get pods -l serving.kserve.io/inferenceservice=fraud-detector
```

Output:

```text
NAME                                        READY   STATUS    RESTARTS   AGE
fraud-detector-predictor-7dc8977dc9-5smhs   1/1     Running   0          107s
```

The predictor pod is now:

- `Running`
- `1/1` containers ready
- `0` restarts

This confirms that the corrected PVC reference allowed the predictor to start successfully.

## Verify Predictor Service

```bash
kubectl get svc | grep fraud-detector
```

Output:

```text
fraud-detector-predictor   ClusterIP   10.96.76.3   <none>        80/TCP    5m52s
```

## Test the Prediction Endpoint

Port-forward the predictor service:

```bash
kubectl port-forward svc/fraud-detector-predictor 8080:80
```

Then, from another terminal, send a prediction request:

```bash
curl -X POST \
  http://localhost:8080/v1/models/fraud-detector:predict \
  -H 'Content-Type: application/json' \
  -d '{"instances":[[1,2,3,4]]}'
```

The expected response is JSON containing a `predictions` array:

```json
{"predictions":[...]}
```

## Final Verification

Check the InferenceService:

```bash
kubectl get isvc fraud-detector
```

Check the PVC:

```bash
kubectl get pvc model-storage
```

The final state should satisfy:

- `fraud-detector` exists as a KServe `InferenceService`.
- `spec.predictor.model.storageUri` references `pvc://model-storage/`.
- The `model-storage` PVC exists in the namespace.
- The predictor pod is `Running`.
- `InferenceService` condition `Ready=True`.
- `/v1/models/fraud-detector:predict` returns a JSON response containing a `predictions` array.

## Root Cause

The predictor remained pending because the manifest referenced a nonexistent PVC:

```text
pvc://models-storage/
```

The correct PVC was:

```text
model-storage
```

Therefore, the required fix was simply:

```diff
- storageUri: "pvc://models-storage/"
+ storageUri: "pvc://model-storage/"
```

After applying the correction, KServe successfully mounted the PVC, started the predictor, and transitioned the `InferenceService` to `Ready=True`.

### Screenshots
