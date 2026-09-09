# Day 99: Capstone (3/4) — GitOps Continuous Deployment with ArgoCD
The xFusionCorp Industries MLOps team is building the GitOps deployment layer for their fraud-detector model server (an nginx stand-in — the rollout loop is image-agnostic). The server's Kubernetes manifests live in a Gitea repo, and ArgoCD should reconcile the cluster against that repo. The kind cluster, in-cluster Gitea with the seeded mlops-deploy repo, and ArgoCD are all in place, but no Application exists yet. Your task is to wire and drive the loop: complete application.yaml so ArgoCD tracks the mlops-deploy manifests and apply it, sync it to deploy the server, then roll a new image version by bumping the tag in the Gitea repo and syncing again.


The Gitea UI (port 3000) and the ArgoCD UI (port 5000) buttons at the top of the lab open the relevant UIs. Both accept ArgoCD admin / adminadmin, Gitea gitops-admin / adminadmin. Pre-staged state:

Gitea repo gitops-admin/mlops-deploy contains manifests/deployment.yaml (image nginx:1.25-alpine) and manifests/service.yaml (NodePort 30080, exposed on host :8085).
ArgoCD is installed with the mlops-deploy repository already registered, but no Application exists yet — you create it.
/root/code/application.yaml is a scaffold with three TODOs. Fill them so the Application tracks the repo:
repo URL: http://gitea-http.gitea.svc.cluster.local:3000/gitops-admin/mlops-deploy.git
manifests path: manifests
destination namespace: default
The target version is nginx:1.27-alpine. The end state must include:

An ArgoCD Application fraud-detector exists and is Synced + Healthy (tests poll up to 240 s).
manifests/deployment.yaml in the Gitea repo references nginx:1.27-alpine.
The fraud-detector Deployment in default runs image nginx:1.27-alpine.
http://localhost:8085/ returns HTTP 200 from the running pod.
The source of truth is the Gitea repo — the rollout runs through the Gitea web editor and the ArgoCD UI. Reference manifests also live under /root/code/manifests/ for transparency, but edits to those local files are not tracked and will not roll out.

## Objective

Build a GitOps deployment workflow for the fraud-detector model server using Gitea as the source of truth and ArgoCD for Kubernetes reconciliation.

The final deployment must:

- Have an ArgoCD Application named `fraud-detector`
- Be `Synced` and `Healthy`
- Track the Gitea `mlops-deploy` repository
- Deploy manifests from the `manifests` directory
- Deploy into the `default` namespace
- Use `nginx:1.27-alpine`
- Serve HTTP 200 through `http://localhost:8085/`

## Environment

- Gitea UI: port `3000`
- ArgoCD UI: port `5000`
- Gitea username: `gitops-admin`
- Gitea password: `adminadmin`
- ArgoCD username: `admin`
- ArgoCD password: `adminadmin`

Repository:

```text
gitops-admin/mlops-deploy
```

Repository URL:

```text
http://gitea-http.gitea.svc.cluster.local:3000/gitops-admin/mlops-deploy.git
```

The repository contains:

```text
manifests/
├── deployment.yaml
└── service.yaml
```

The initial deployment uses:

```yaml
image: nginx:1.25-alpine
```

The target image is:

```yaml
image: nginx:1.27-alpine
```

## Step 1: Complete application.yaml

The scaffold at `/root/code/application.yaml` contains three TODOs.

The completed file is:

```yaml
# ArgoCD Application for the fraud-detector model server.
#
# Fill the three TODOs (see the task for the repo URL, manifests path,
# and target namespace), then apply it so ArgoCD reconciles the cluster
# against the mlops-deploy repo:
#   kubectl apply -n argocd -f /root/code/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fraud-detector
  namespace: argocd
spec:
  project: default
  source:
    repoURL: "http://gitea-http.gitea.svc.cluster.local:3000/gitops-admin/mlops-deploy.git"
    targetRevision: HEAD
    path: "manifests"
  destination:
    server: https://kubernetes.default.svc
    namespace: "default"
```

The three required values are:

```text
repoURL: http://gitea-http.gitea.svc.cluster.local:3000/gitops-admin/mlops-deploy.git
path: manifests
namespace: default
```

## Step 2: Apply the ArgoCD Application

Run:

```bash
kubectl apply -n argocd -f /root/code/application.yaml
```

Expected:

```text
application.argoproj.io/fraud-detector created
```

Check the Application:

```bash
kubectl -n argocd get application fraud-detector
```

Initially it may show:

```text
NAME             SYNC STATUS   HEALTH STATUS
fraud-detector   OutOfSync     Missing
```

This is expected because the Application exists but the resources have not been synchronized yet.

## Step 3: Initial ArgoCD Sync

Open the ArgoCD UI from the lab.

Log in with:

```text
Username: admin
Password: adminadmin
```

Open the `fraud-detector` Application.

Choose:

```text
Sync → Synchronize
```

Wait for the Application to become:

```text
Synced
Healthy
```

Verify from the terminal:

```bash
kubectl -n argocd get application fraud-detector
```

Check the Deployment:

```bash
kubectl get deployment fraud-detector -n default
```

Check the pods:

```bash
kubectl get pods -n default
```

The initial image should be:

```bash
kubectl get deployment fraud-detector -n default \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected:

```text
nginx:1.25-alpine
```

## Step 4: Update the Image in Gitea

The Gitea repository is the GitOps source of truth.

Do **not** modify:

```text
/root/code/manifests/deployment.yaml
```

Local reference manifests are not tracked for this rollout.

Instead, open the Gitea UI and log in:

```text
Username: gitops-admin
Password: adminadmin
```

Navigate to:

```text
gitops-admin/mlops-deploy
└── manifests
    └── deployment.yaml
```

Open `deployment.yaml` using the web editor.

Change:

```yaml
image: nginx:1.25-alpine
```

to:

```yaml
image: nginx:1.27-alpine
```

Commit the change **directly to the `main` branch**.

## Step 5: Refresh ArgoCD

Return to the ArgoCD UI.

Open the `fraud-detector` Application.

Click **Refresh** so ArgoCD detects the new Git commit.

The Application should become:

```text
OutOfSync
```

This is expected because the Git repository now specifies `nginx:1.27-alpine`, while the Kubernetes Deployment is still running `nginx:1.25-alpine`.

## Step 6: Sync the New Version

In ArgoCD, choose:

```text
Sync → Synchronize
```

Wait for the rollout to complete.

The Application should return to:

```text
Synced
Healthy
```

## Step 7: Verify the New Image

Check the Deployment image:

```bash
kubectl get deployment fraud-detector -n default \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected:

```text
nginx:1.27-alpine
```

Check the pods:

```bash
kubectl get pods -n default
```

The fraud-detector pod should be:

```text
1/1     Running
```

## Step 8: Verify HTTP

The Kubernetes Service is a NodePort exposed on host port `8085`.

Run:

```bash
curl -i http://localhost:8085/
```

The response must contain:

```text
HTTP/1.1 200 OK
```

A concise status check can be performed with:

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8085/
```

Expected:

```text
200
```

## Final Verification

Run:

```bash
kubectl -n argocd get application fraud-detector
```

Expected:

```text
NAME             SYNC STATUS   HEALTH STATUS
fraud-detector   Synced        Healthy
```

Verify the image:

```bash
kubectl get deployment fraud-detector -n default \
  -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Expected:

```text
nginx:1.27-alpine
```

Verify the endpoint:

```bash
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:8085/
```

Expected:

```text
200
```

## Final State

The completed GitOps flow is:

```text
Gitea repository
      │
      │ deployment.yaml
      │ image: nginx:1.27-alpine
      ▼
   ArgoCD
      │
      │ Sync
      ▼
Kubernetes Deployment
      │
      │ nginx:1.27-alpine
      ▼
fraud-detector Pod
      │
      ▼
Service NodePort :30080
      │
      ▼
localhost:8085
      │
      ▼
HTTP 200
```

## Important Notes

- The ArgoCD Application name must be `fraud-detector`.
- The repository URL must point to the in-cluster Gitea repository.
- The manifest path is `manifests`.
- The destination namespace is `default`.
- The image update must be made in Gitea, not in the local `/root/code/manifests/` directory.
- The image change must be committed to `main`.
- After the Git change, refresh and synchronize the Application in ArgoCD.
- Do not bypass GitOps with `kubectl set image`.
- The final Deployment must run `nginx:1.27-alpine`.
- The final ArgoCD Application must be `Synced` and `Healthy`.
- `http://localhost:8085/` must return HTTP `200`.

### Screenshots
