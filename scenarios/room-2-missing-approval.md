# Room 2: The Missing Approval

## Objective

Promote a release to Production when a manual intervention blocks the deployment.

## Starting state

- Lifecycle requires **manual approval** before Production
- Room 1 is fixed (`Podinfo.Faults.Unready: false` in Production)
- A release has been deployed to Development and Staging successfully

## Symptoms

- Production deployment is **blocked** or **waiting**
- Octopus shows a manual intervention / approval required
- No Git change needed yet — the gate is in Octopus

## Intended fix path

1. Use Octopus MCP to find pending interruptions or blocked tasks
2. Approve the manual intervention in Octopus UI (or via API if configured)
3. Resume / redeploy to Production
4. Verify Argo CD `podinfo-production` is Healthy

## Reset

Create a new release and attempt Production promotion again (approval will be required).

## What this teaches

- Octopus lifecycles and promotion controls
- Manual interventions as operational guardrails
- Difference between "app broken" vs "deploy blocked"
