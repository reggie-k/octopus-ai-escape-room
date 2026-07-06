# Adding a new room

## 1. Copy the template

```bash
cp rooms/_template.yaml rooms/4.yaml
```

## 2. Fill in the file

**`player`** (AI reads this during play):

- `title` — escape room name for the narrative
- `teaser` — atmosphere only; no variable names, paths, or pod names

**`constraint`** (optional):

- `null` — default; MCP-driven investigation
- `approval-required` — player must clear a lifecycle gate
- `runbook-required` — recovery must use a runbook

**`author`** (you + blog readers; AI must not read during play):

- `setup` — what to break in Octopus/Git before the room starts
- `reset` — how to restore for the next run
- `teaches` — optional blurb for README/blog

## 3. Register (optional)

Add an entry to `rooms/registry.yaml` for the README table.

## 4. Apply the break

Follow `author.setup` in Octopus (variables, lifecycle, runbook, etc.).

## 5. Play

Tell the AI: **"Room 4 is active"**.

## Example breaks

| Idea | Author setup sketch |
|------|---------------------|
| Bad image tag | Scope a bad `Podinfo.ImageTag` to Production |
| Wrong env variable | Scope a fault flag differently per environment |
| Blocked promotion | Lifecycle manual step before Production |
| Runbook only | Create runbook; break prod; `constraint: runbook-required` |
