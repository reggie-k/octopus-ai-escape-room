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
    values-production.yaml    # Production values (Octopus-managed)
octopus/
  templates/                  # Templates for Octopus Update Manifests step
prompts/
  game-master.md              # AI behavior (hints, MCP, no spoilers)
rooms/
  _template.yaml              # Copy to add a new room
  1.yaml … 3.yaml             # player teaser + author setup per room
  registry.yaml               # Room index for README/blog
docs/
  playing.md                  # How to play
  extending.md                # How to add rooms
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
   - Source template from `octopus/templates/values-#{Octopus.Environment.Slug}.yaml`
   - Git credential for `https://github.com/reggie-k/octopus-ai-escape-room.git`
   - Verification: **Argo CD Application is healthy**
   - Trigger Sync: enabled

   > **Note:** Do not use `| ToLower` in the template path — the pipe is treated as a
   > path separator and produces an empty Git package. Use `#{Octopus.Environment.Slug}` instead.

2. **Wait For Argo CD Applications** (optional extra step)

### Lifecycle

`Development → Staging → Production` with a **manual intervention** before Production (Room 2).

## Escape rooms

| Room | Title | Facilitator setup |
|------|-------|-------------------|
| 1 | The Wrong Config | [rooms/1.yaml](rooms/1.yaml) (`author.setup`) |
| 2 | The Missing Approval | [rooms/2.yaml](rooms/2.yaml) |
| 3 | The Runbook Rescue | [rooms/3.yaml](rooms/3.yaml) |

Player-facing text is in each file's `player` section (title + teaser only).
The AI discovers apps, variables, and paths via **Octopus MCP** — no spoilers in repo.

Add Room 4+: copy [rooms/_template.yaml](rooms/_template.yaml) — see [docs/extending.md](docs/extending.md).

## Playing the game

1. Connect Cursor to **Octopus MCP** (project open in this repo — [`.cursor/rules/escape-room.mdc`](.cursor/rules/escape-room.mdc) loads automatically)
2. Apply facilitator setup from `rooms/N.yaml` → `author.setup`
3. Say **"Room N is active"** or **"this release is stuck"**
4. Fix with AI hints; validate via Octopus → Git → Argo CD

Details: [docs/playing.md](docs/playing.md)

## License

MIT
