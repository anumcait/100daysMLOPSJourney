# Day 96: Deploy a GitOps Application via the ArgoCD NEW APP Form

The xFusionCorp Industries ML platform team is adopting GitOps for their Kubernetes workloads: every deployable resource lives in a git repo, and ArgoCD keeps the cluster reconciled against that repo. ArgoCD v3.3.6 is already running on the mlops kind cluster and the UI is reachable on port 5000. Your task is to use the UI to create an Application named guestbook pointing at the canonical argoproj/argocd-example-apps guestbook path—a small stand-in workload, since the GitOps reconcile loop you are practising is model-agnostic—sync it, and confirm that the cluster matches the repo.

ArgoCD is running on the mlops kind cluster and its UI is exposed via the ArgoCD UI button at the top of the lab (port 5000); log in as admin / admin. From the UI, create an Application named guestbook (project default) that tracks the guestbook path of https://github.com/argoproj/argocd-example-apps at revision HEAD, deploying to the default namespace of https://kubernetes.default.svc. Enable automatic sync so ArgoCD reconciles the cluster to the repo without manual syncing, then confirm the app converges.

The end state must include:

GET /api/v1/applications/guestbook (after auth) returns an Application.
spec.source.repoURL resolves to https://github.com/argoproj/argocd-example-apps.
spec.source.path == "guestbook".
status.sync.status == "Synced" AND status.health.status == "Healthy" (tests poll up to 240 s).
GitOps is declarative deployment with git as the source of truth: you describe the desired cluster state in a repo, and ArgoCD's controller loop reconciles the real cluster to match. Automatic sync + self-heal means any drift (a teammate kubectl deleteing a pod, say) is corrected within one reconciliation cycle without a human clicking a button.

## Objective

Deploy the `guestbook` application to the `mlops` kind cluster using ArgoCD and GitOps.

ArgoCD v3.3.6 is already running, with the UI exposed on port `5000`.

## ArgoCD Login

- Username: `admin`
- Password: `admin`

## Application Configuration

Create an Application named `guestbook` with the following configuration:

| Setting | Value |
|---|---|
| Application Name | `guestbook` |
| Project | `default` |
| Repository URL | `https://github.com/argoproj/argocd-example-apps` |
| Revision | `HEAD` |
| Path | `guestbook` |
| Cluster URL | `https://kubernetes.default.svc` |
| Namespace | `default` |
| Sync Policy | `Automatic` |
| Self Heal | Enabled |
| Prune Resources | Enabled |

> **Important:** The repository path must be exactly `guestbook`. Do not select `helm-guestbook` or `kustomize-guestbook`.

## Steps

1. Open the ArgoCD UI using the ArgoCD UI button provided by the lab.
2. Log in with `admin / admin`.
3. Go to **Applications → New App**.
4. Enter the application name:
   `guestbook`
5. Select project:
   `default`
6. Configure the source:
   - Repository URL: `https://github.com/argoproj/argocd-example-apps`
   - Revision: `HEAD`
   - Path: `guestbook`
7. Configure the destination:
   - Cluster: `https://kubernetes.default.svc`
   - Namespace: `default`
8. Set the sync policy to **Automatic**.
9. Enable **Self Heal**.
10. Enable **Prune Resources** if available.
11. Click **Create**.
12. Wait for ArgoCD to automatically reconcile the application.

## Verification

The Application should eventually show:

```text
SYNC STATUS: Synced
HEALTH: Healthy
```

The Application summary should contain:

```text
Project:       default
Repository:    https://github.com/argoproj/argocd-example-apps
Target Revision: HEAD
Path:          guestbook
Destination:   in-cluster
Namespace:     default
Sync Policy:   Automated
```

The API endpoint:

```text
GET /api/v1/applications/guestbook
```

must return an Application where:

```text
spec.source.repoURL == "https://github.com/argoproj/argocd-example-apps"
spec.source.path == "guestbook"
status.sync.status == "Synced"
status.health.status == "Healthy"
```

Tests may poll for up to 240 seconds while the application converges.

## GitOps Concept

GitOps uses Git as the source of truth for declarative infrastructure and application configuration.

The desired Kubernetes state is stored in the Git repository, while ArgoCD continuously compares that desired state with the actual cluster state.

With automatic sync and self-heal enabled:

```text
Git Repository
      |
      v
ArgoCD
      |
      v
Kubernetes Cluster
      |
      v
Reconciled State
```

If a resource drifts from the desired state—for example, if a pod is deleted manually—ArgoCD detects the difference and reconciles the cluster back to the state defined in Git.

## Final State

The completed application should be:

```text
Application: guestbook
Repository:  https://github.com/argoproj/argocd-example-apps
Path:        guestbook
Revision:    HEAD
Namespace:   default
Sync:        Synced
Health:      Healthy
Auto Sync:   Enabled
Self Heal:   Enabled
```

The Git repository remains the source of truth, and ArgoCD keeps the Kubernetes cluster synchronized with it.
````

### Screenshots