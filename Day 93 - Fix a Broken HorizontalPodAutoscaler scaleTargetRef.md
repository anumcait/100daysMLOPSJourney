# Day 93: Fix a Broken HorizontalPodAutoscaler `scaleTargetRef`
The xFusionCorp Industries ML platform team has autoscaled their fraud-server Deployment, which serves as a stand-in for the fraud-detector model using nginx:alpine. This task focuses on the configuration of the HorizontalPodAutoscaler (HPA), rather than the model itself. Currently, executing kubectl get hpa results in TARGETS <unknown>/70%, indicating that the HPA is unable to find a metric to measure, and as a consequence, it is not performing any scaling operations. Please investigate the issue and rectify the HPA manifest located at /root/code/k8s/hpa.yaml.


A kind cluster is pre-provisioned and kubectl is configured to reach it. metrics-server is installed and patched for kind's kubelet, the fraud-server Deployment is running with CPU requests and limits, and the HPA manifest at /root/code/k8s/hpa.yaml has already been applied — its TARGETS column currently reads <unknown>/70%. Inspect the HPA to see why it can't read a metric:

kubectl describe hpa fraud-server-hpa

The end state must include:

The fraud-server Deployment is Available with 2 or more replicas.
HPA fraud-server-hpa exists, and its scaleTargetRef points at a Deployment that exists in the cluster.
HPA.status.currentMetrics[].resource.current.averageUtilization (or averageValue) is populated — no longer <unknown> (tests poll up to 180 s).
An HPA is a reference plus a metric target — nothing works if the reference points at a resource that no longer exists. Silent break modes like this are why CI gates on kubectl apply --dry-run=server (which validates the scaleTargetRef against live resources) are a useful safety net in pipelines that rename workloads.

## Task

The xFusionCorp Industries ML platform team has autoscaled their `fraud-server` Deployment, which serves as a stand-in for the fraud-detector model using `nginx:alpine`.

The HorizontalPodAutoscaler (HPA) was showing:

```text
TARGETS <unknown>/70%
```

This indicated that the HPA could not find the resource it was configured to scale, so it could not calculate CPU utilization or perform scaling.

The task was to investigate and fix:

```text
/root/code/k8s/hpa.yaml
```

## Existing Deployment

The `fraud-server` Deployment was correctly configured with CPU requests and limits:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fraud-server
  labels:
    app: fraud-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: fraud-server
  template:
    metadata:
      labels:
        app: fraud-server
    spec:
      containers:
        - name: server
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
```

The important detail is the Deployment name:

```yaml
name: fraud-server
```

## Broken HPA

The HPA was configured with the following `scaleTargetRef`:

```yaml
scaleTargetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: fraud-serving
```

The problem was that the HPA referenced:

```text
fraud-serving
```

but the actual Deployment was:

```text
fraud-server
```

Therefore, the HPA could not find the target Deployment.

## Fix

The HPA manifest was corrected to reference the existing Deployment:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: fraud-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: fraud-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

The key change was:

```diff
- name: fraud-serving
+ name: fraud-server
```

## Apply the Fix

Apply the corrected HPA manifest:

```bash
kubectl apply -f /root/code/k8s/hpa.yaml
```

Output:

```text
horizontalpodautoscaler.autoscaling/fraud-server-hpa configured
```

## Verification

Check the Deployment:

```bash
kubectl get deployment fraud-server
```

Result:

```text
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
fraud-server   2/2     2            2           4m52s
```

The Deployment is available with 2 replicas.

Check the HPA:

```bash
kubectl get hpa fraud-server-hpa
```

Result:

```text
NAME               REFERENCE                 TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
fraud-server-hpa   Deployment/fraud-server   cpu: 0%/70%   2         10        2          4m59s
```

The important parts are:

```text
REFERENCE: Deployment/fraud-server
TARGETS:   cpu: 0%/70%
```

The HPA is now successfully reading CPU utilization instead of showing `<unknown>`.

## Detailed HPA Verification

Run:

```bash
kubectl describe hpa fraud-server-hpa
```

Relevant output:

```text
Reference:                                             Deployment/fraud-server
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (0) / 70%
Min replicas:                                          2
Max replicas:                                          10
Deployment pods:                                       2 current / 2 desired
Conditions:
  Type            Status  Reason               Message
  ----            ------  ------               -------
  AbleToScale     True    ScaleDownStabilized  recent recommendations were higher than current one, applying the highest recent recommendation
  ScalingActive   True    ValidMetricFound     the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  False   DesiredWithinRange   the desired count is within the acceptable range
```

The critical condition is:

```text
ScalingActive   True    ValidMetricFound
```

This confirms that the HPA successfully calculated CPU resource utilization.

## Historical Warning

The `describe` output still contained an older warning:

```text
Warning  FailedGetScale  deployments/scale.apps "fraud-serving" not found
```

This was generated while the HPA still had the broken target reference.

It is a historical Event and does not indicate that the current HPA configuration is broken.

The current HPA reference is:

```text
Deployment/fraud-server
```

and the current metric is successfully populated:

```text
cpu: 0%/70%
```

## Final Validation

The live HPA can be inspected with:

```bash
kubectl get hpa fraud-server-hpa -o yaml
```

The expected `scaleTargetRef` is:

```yaml
scaleTargetRef:
  apiVersion: apps/v1
  kind: Deployment
  name: fraud-server
```

The HPA status should contain a populated CPU metric rather than `<unknown>`.

## Final State

All required conditions are satisfied:

- `fraud-server` Deployment exists.
- `fraud-server` Deployment is Available with 2 replicas.
- `fraud-server-hpa` exists.
- `fraud-server-hpa.spec.scaleTargetRef` points to `Deployment/fraud-server`.
- The target Deployment exists in the cluster.
- The HPA successfully reads CPU utilization.
- `ScalingActive=True`.
- `ValidMetricFound` is reported.
- HPA target is `70%` CPU utilization.
- Minimum replicas are `2`.
- Maximum replicas are `10`.

## Key Lesson

An HPA depends on its `scaleTargetRef` pointing to a real, existing scalable workload.

A mismatch such as:

```yaml
name: fraud-serving
```

when the actual Deployment is:

```yaml
name: fraud-server
```

causes the HPA to fail to retrieve the target's scale information and can result in:

```text
TARGETS <unknown>/70%
```

Correcting the reference restores HPA functionality.

This is also why validating Kubernetes manifests with:

```bash
kubectl apply --dry-run=server
```

can be useful in CI pipelines: it helps catch invalid references against live cluster resources before changes are applied.

### Screenshots

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5c917e5e-bd0f-4c5d-bc8a-29b873280c7c" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/82508c3a-42e7-4cf9-a83f-b523e3f77f09" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/9fcbddf0-26d4-4679-854c-b02e8e767d0d" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/0893e633-cfe2-4799-a5dd-fca66dc3ab20" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/f3142f77-9a73-4303-b87b-1a7b07dc0559" />





