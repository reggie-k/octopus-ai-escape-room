# Room 3: The Runbook Rescue

## Objective

Use an Octopus runbook to fix Production and verify recovery.

## Starting state

- Production is broken again (`Podinfo.Faults.Unready: true` or bad image tag)
- A runbook exists: **Fix Production Podinfo** (you create this in Octopus)

## Suggested runbook steps

1. Update project variable `Podinfo.Faults.Unready` to `false` (Production scope)
2. Deploy the latest release to Production  
   — or trigger **Update Argo CD Application Manifests** via runbook if modeled as a runbook-only process

## Symptoms

- Same as Room 1 (unhealthy Production app)
- Player must use the runbook instead of fixing variables manually in the UI

## Intended fix path

1. AI hints point toward the runbook
2. Run **Fix Production Podinfo** runbook with correct parameters
3. Octopus updates Git and/or redeploys
4. Argo CD syncs; Production becomes Healthy

## Reset

Run a separate runbook or manual step to set `Podinfo.Faults.Unready: true` again.

## What this teaches

- Runbooks as operational automation
- Recovery workflows in Octopus
- Combining runbooks with GitOps (Argo CD)
