# Octopus AI Escape Room (podinfo + Argo CD)

A DevOps escape room powered by **Octopus Deploy**, **Argo CD**, and **podinfo** on Kubernetes.

Octopus orchestrates environment promotion and writes Helm values to Git. Argo CD syncs the cluster. An AI assistant connected via **Octopus MCP** guides you through diagnosing and fixing broken deployments.

## Architecture

```text
Octopus Cloud
  → Update Argo CD Application Manifests (commits values to Git)
  → Argo CD syncs
  → k3s cluster (podinfo)

Cursor / Claude + Octopus MCP = game master
```

## Prerequisites

- k3s cluster with `kubectl` access
- [Argo CD](https://argo-cd.readthedocs.io/) installed in the cluster
- [Octopus Cloud](https://octopus.com/) instance
- [Octopus Argo CD Gateway](https://octopus.com/docs/argo-cd/instances) installed on k3s
- Git credentials configured in Octopus for this repository

## Quick start (k3s)

### 1. Install Argo CD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd wait deploy/argocd-server --for=condition=Available --timeout=300s
```

### 2. Register the podinfo applications

After pushing this repo to GitHub, apply the app-of-apps:

```bash
kubectl apply -f argocd/bootstrap-app.yaml
```

Or apply individual apps:

```bash
kubectl apply -f argocd/applications/
```

### 3. Verify in Argo CD

```bash
kubectl -n argocd port-forward svc/argocd-server 8080:443
```

Open https://localhost:8080 and confirm `podinfo-dev`, `podinfo-staging`, and `podinfo-production` are **Synced** and **Healthy**.

### 4. Access podinfo (dev)

```bash
kubectl -n podinfo-dev port-forward svc/podinfo 9898:9898
curl http://localhost:9898/readyz
curl http://localhost:9898/version
```

## Repository layout

```text
argocd/
  bootstrap-app.yaml          # App-of-apps (optional)
  applications/               # Argo CD Application per environment
helm/
  podinfo/
    values-development.yaml   # Working config (Development)
    values-staging.yaml       # Working config (Staging)
    values-production.yaml    # Room 1 starting state (broken)
octopus/
  templates/                  # Templates for Octopus Update Manifests step
prompts/
  game-master.md              # AI game master prompt (for Cursor)
scenarios/
  room-1-wrong-config.md
  room-2-missing-approval.md
  room-3-runbook-rescue.md
```

## Octopus setup

### Project

Create a project with slug: **`octopus-ai-escape-room`**

### Environments

| Environment  | Slug           |
|--------------|----------------|
| Development  | `development`  |
| Staging      | `staging`      |
| Production   | `production`   |

Environment slugs must match the `argo.octopus.com/environment.<source-name>` annotations on the Argo CD Applications.

Multi-source applications (Helm chart + Git values ref) require **named sources** and **source-scoped** annotations on the Git source Octopus updates:

```yaml
metadata:
  annotations:
    argo.octopus.com/project.values: octopus-ai-escape-room
    argo.octopus.com/environment.values: development
    argo.octopus.com/path.values: helm/podinfo
spec:
  sources:
    - name: chart
      repoURL: https://stefanprodan.github.io/podinfo
      chart: podinfo
      # ...
    - name: values
      repoURL: https://github.com/reggie-k/octopus-ai-escape-room.git
      ref: values
      # ...
```

### Project variables

| Variable | Development | Staging | Production |
|----------|-------------|---------|------------|
| `Podinfo.Faults.Unready` | `false` | `false` | `true` |
| `Podinfo.ImageTag` | `6.14.0` | `6.14.0` | `6.14.0` |

### Deployment process

1. **Update Argo CD Application Manifests**
   - Source template from `octopus/templates/values-#{Octopus.Environment.Name | ToLower}.yaml`
   - Target Git path: `helm/podinfo/values-#{Octopus.Environment.Name | ToLower}.yaml`
   - Verification: **Argo CD Application is healthy**
   - Trigger Sync: enabled

2. **Wait For Argo CD Applications** (optional extra step)

### Lifecycle

`Development → Staging → Production` with a **manual intervention** before Production (Room 2).

## Escape rooms

| Room | Scenario | Fix |
|------|----------|-----|
| 1 | Wrong config in Production (`faults.unready: true`) | Set `Podinfo.Faults.Unready` to `false`, redeploy |
| 2 | Production promotion blocked | Approve manual intervention, redeploy |
| 3 | Broken config needs runbook | Runbook updates variable and triggers redeploy |

See [scenarios/](scenarios/) for details.

## Playing the game

1. Connect Cursor to **Octopus MCP**
2. Paste the prompt from [prompts/game-master.md](prompts/game-master.md)
3. Add the active room scenario from [scenarios/](scenarios/)
4. Deploy / promote in Octopus and fix issues with AI hints

## License

MIT
