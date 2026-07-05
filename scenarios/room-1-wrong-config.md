# Room 1: The Wrong Config

## Objective

Fix the Production podinfo deployment so Argo CD reports **Healthy** and `/readyz` returns OK.

## Starting state

- `helm/podinfo/values-production.yaml` has `faults.unready: true`
- Argo CD Application `podinfo-production` is **Degraded** or pods are not Ready
- Development and Staging environments are healthy (for comparison)

## Symptoms

- Deployment/sync may succeed in Octopus and Argo
- Pods run but fail readiness checks
- `kubectl -n podinfo-production get pods` shows Not Ready
- `curl` to `/readyz` fails (after port-forward)

## Intended fix path

1. Inspect Octopus project variables — find `Podinfo.Faults.Unready` scoped to Production
2. Set `Podinfo.Faults.Unready` to `false` for Production
3. Create a new release and deploy to Production (or re-run deployment)
4. Octopus commits updated values to Git
5. Argo CD syncs; app becomes Healthy

## Reset

Set `Podinfo.Faults.Unready` back to `true` for Production and redeploy, or commit `unready: true` in values-production.yaml.

## What this teaches

- Environment-scoped Octopus variables
- Git as source of truth (Octopus writes, Argo CD syncs)
- Diagnosing readiness failures vs deploy failures
