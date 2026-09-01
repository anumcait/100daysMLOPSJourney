# Day 92: Fix a Service targetPort Mismatch on a Kubernetes Deployment
The xFusionCorp Industries ML platform team has deployed a Kubernetes Deployment of the fraud-detector model server. This deployment uses an nginx:alpine container listening on port 80. Please note that this task focuses on Kubernetes Service routing, rather than the model itself. Currently, the Deployment is operating with two replicas, and executing kubectl get deploy fraud-detector confirms that the status shows READY 2/2. However, any requests made to the Service at fraud-detector-svc:8080 result in a timeout. Your task is to diagnose the issue preventing the Service from routing traffic to its backing pods and to correct the manifest accordingly.


A kind cluster named mlops is pre-provisioned and kubectl is configured to reach it. The Deployment and Service manifests live at /root/code/k8s/deployment.yaml and /root/code/k8s/service.yaml, and both have already been applied — the fraud-detector Deployment reports READY 2/2, but requests to fraud-detector-svc:8080 time out. Inspect the Service and its endpoints to see why traffic never reaches the pods:

kubectl describe svc fraud-detector-svc

The end state must include:

Deployment fraud-detector is Available.
Service fraud-detector-svc exists; clients still dial it on 8080.
Endpoints/fraud-detector-svc routes to the port the container is actually listening on (80).
An in-cluster HTTP GET http://fraud-detector-svc:8080/ returns the nginx default page.
In a Kubernetes Service, port is what clients dial and targetPort is what kube-proxy forwards that traffic to on the backing pods. They can legitimately differ (e.g. external 8080 → internal 80), but targetPort must match a real listening port on the container.

## Task

The xFusionCorp Industries ML platform team deployed a Kubernetes Deployment named `fraud-detector`. The Deployment uses an `nginx:alpine` container listening on port `80`.

The Service `fraud-detector-svc` is exposed on port `8080`, but requests to:

```text
http://fraud-detector-svc:8080/
```

were timing out.

The goal was to diagnose the Service routing issue and correct the manifest without changing the client-facing Service port.

## Environment

- Kind cluster: `mlops`
- Deployment: `fraud-detector`
- Service: `fraud-detector-svc`
- Deployment manifest: `/root/code/k8s/deployment.yaml`
- Service manifest: `/root/code/k8s/service.yaml`
- Deployment replicas: `2`
- Container listening port: `80`
- Service client port: `8080`

## Diagnosis

First, inspect the Service:

```bash
kubectl describe svc fraud-detector-svc
```

The initial configuration showed:

```text
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
Endpoints:                10.244.0.5:8080,10.244.0.6:8080
```

The Service was listening on `8080`, but it was also forwarding traffic to port `8080` on the pods.

However, the `nginx:alpine` containers listen on port `80`.

The endpoints confirmed that Kubernetes had selected the correct pods, but was targeting the wrong port:

```bash
kubectl get endpoints fraud-detector-svc -o yaml
```

Initial result:

```yaml
ports:
  - port: 8080
    protocol: TCP
```

Therefore, the problem was not the Deployment or Service selector. The problem was the Service's `targetPort`.

## Kubernetes Port Mapping

A Kubernetes Service can expose one port to clients and forward traffic to a different port on the pods.

In this case, the desired routing is:

```text
Client
  |
  | HTTP :8080
  v
Service: fraud-detector-svc
  |
  | targetPort: 80
  v
Pod
  |
  | nginx listening on :80
  v
nginx
```

The Service's:

- `port` must remain `8080` because clients connect to `fraud-detector-svc:8080`.
- `targetPort` must be `80` because nginx is listening on port `80`.

## Fix

Edit the Service manifest:

```bash
vi /root/code/k8s/service.yaml
```

Change:

```yaml
targetPort: 8080
```

to:

```yaml
targetPort: 80
```

The relevant Service configuration should be:

```yaml
ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
```

Apply the updated manifest:

```bash
kubectl apply -f /root/code/k8s/service.yaml
```

Expected output:

```text
service/fraud-detector-svc configured
```

## Verification

Check the Service again:

```bash
kubectl describe svc fraud-detector-svc
```

The corrected configuration showed:

```text
Port:                     <unset>  8080/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30092/TCP
Endpoints:                10.244.0.5:80,10.244.0.6:80
```

This confirms that the Service still accepts traffic on `8080`, but now forwards it to port `80` on the backing pods.

Check the Deployment:

```bash
kubectl get deploy fraud-detector
```

Final result:

```text
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
fraud-detector   2/2     2            2           5m29s
```

The Deployment is healthy and has both replicas available.

## Final HTTP Test

Test the Service from inside the cluster:

```bash
kubectl run curl-test --rm -i --restart=Never \
  --image=curlimages/curl -- \
  curl -s http://fraud-detector-svc:8080/
```

The request should return the nginx default page, containing:

```html
<title>Welcome to nginx!</title>
```

## Final State

The required end state is:

```text
Deployment fraud-detector
  READY:     2/2
  AVAILABLE: 2

Service fraud-detector-svc
  Client port: 8080/TCP
  Target port: 80/TCP

Endpoints
  10.244.0.5:80
  10.244.0.6:80
```

The Service routing is therefore:

```text
fraud-detector-svc:8080
        |
        v
      :80
        |
        v
nginx:alpine pods
```

## Root Cause

The Service had a `targetPort` mismatch.

It was configured as:

```yaml
port: 8080
targetPort: 8080
```

but the containers were listening on:

```text
80/TCP
```

The corrected configuration is:

```yaml
port: 8080
targetPort: 80
```

The key Kubernetes concept is that `port` is the port exposed by the Service to clients, while `targetPort` is the port on the selected pods where traffic is forwarded.

**Final fix: `8080 -> 80`.**

### Screenshots

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/3db4d437-8c47-4ace-a8a5-627abf7d44f6" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/97893974-2afb-4b93-b410-1abca1742a40" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/31cc6314-944a-4c18-a915-820b5e5d00c7" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/8336735c-c14e-474f-8de4-e8f3e824d328" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/2fd1dab9-ff2c-4f51-b42f-235e9aede792" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/516161ba-606d-4f48-a96d-4767fe761a75" />
<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/b3165b92-dbe7-4f6e-a595-0e040f841885" />







