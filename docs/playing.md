# Playing the escape room

## Prerequisites

- This repo open in Cursor (or another MCP client)
- [Octopus MCP](https://octopus.com/docs/octopus-ai/mcp) connected to your instance
- The `octopus-ai-escape-room` project deployed at least once

## Start a room

Say any of:

- **"Room 1 is active"**
- **"Release <Release number> is stuck in Production — help me"**
- **"My deployment succeeded but the app is unhealthy"**

The AI loads `prompts/game-master.md` and (for numbered rooms) the `player`
section of `rooms/N.yaml`. It investigates with Octopus MCP and gives hints
one at a time.

You do not paste scenario files or pod names — MCP discovers them.

## Win

Depends on what is broken. Usually: Octopus deploy complete, Argo CD **Healthy**
for the target environment, pods ready.

## Facilitator

Before playing, configure the break using the `author.setup` section in
`rooms/N.yaml`. See [extending.md](extending.md).
