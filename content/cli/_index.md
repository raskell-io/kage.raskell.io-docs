+++
title = "CLI Reference"
description = "Command-line interface reference."
weight = 4
sort_by = "weight"
template = "section.html"
+++

# CLI Reference

Complete reference for Kage CLI commands.

## Daemon Commands

```bash
kage daemon start       # Start the daemon
kage daemon stop        # Stop the daemon
kage daemon status      # Check daemon status
```

## Agent Commands

```bash
kage agent spawn        # Spawn a new agent
kage agent list         # List active agents
kage agent attach <id>  # Attach to an agent
kage agent detach       # Detach from current agent
kage agent kill <id>    # Kill an agent
```

## Task Commands

```bash
kage task add "<goal>"  # Add a new task
kage task list          # List tasks
kage task status <id>   # Check task status
kage task resume <id>   # Resume a paused task
```

## Memory Commands

```bash
kage memory search "<query>"  # Search memories
kage memory list              # List recent memories
kage memory show <id>         # Show memory details
```

## Secret Commands

```bash
kage secret set <name> <value>  # Store a secret
kage secret list                 # List secret names
kage secret delete <name>        # Delete a secret
```
