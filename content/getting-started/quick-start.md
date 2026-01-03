+++
title = "Quick Start"
description = "Run your first autonomous agent with Kage."
weight = 2
+++

# Quick Start

This guide will walk you through running your first autonomous agent with Kage.

## Prerequisites

- Kage installed ([Installation Guide](/docs/getting-started/installation/))
- Claude Code CLI installed and configured
- A project directory to work in

## Start the Daemon

Kage runs as a background daemon that manages your agents:

```bash
kage daemon start
```

You can check the daemon status with:

```bash
kage daemon status
```

## Spawn an Agent

Spawn a new agent in your project directory:

```bash
kage agent spawn --repo ~/code/myproject
```

This will:
- Register the repository with Kage
- Start a new Claude Code session
- Begin monitoring the agent

## Attach to the Agent

To interact with the agent:

```bash
kage agent attach <agent-id>
```

You can get the agent ID from `kage agent list`.

## Detach from the Agent

Press `Ctrl+A` then `D` to detach from the agent while keeping it running.

## Give the Agent a Task

You can assign tasks to agents:

```bash
kage task add "Fix all TypeScript errors in the project"
```

The agent will work on the task autonomously until:
- The task is completed
- An iteration limit is reached
- You intervene

## Next Steps

- [Concepts](/docs/concepts/) - Understand how Kage works
- [Configuration](/docs/configuration/) - Customize Kage for your workflow
