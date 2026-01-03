+++
title = "Architecture"
description = "How Kage is designed and why."
weight = 1
+++

# Architecture

Kage is designed around several core principles that make it uniquely suited for orchestrating AI agents.

## Supervisor Pattern

Kage acts as a **supervisor** for AI agents, not just a session manager. The supervisor:

- Spawns agents with specific goals
- Monitors health and progress
- Enforces iteration limits and guardrails
- Enables checkpoint/resume workflows

This is inspired by Erlang/OTP supervision trees, where supervisors manage the lifecycle of their children.

## Event Sourcing

All agent context is stored as **immutable events**. This enables:

- **Full audit trails** - See everything your agents have done
- **Cross-agent context sharing** - Agents can learn from each other
- **Replay for debugging** - Understand what went wrong
- **Time-travel queries** - Query context at any point in time

## Single Binary

Kage is distributed as a **single static binary** with zero runtime dependencies:

- No database server required
- No external services needed
- Pure Rust with embedded storage (redb)
- Works on Linux, macOS, and Windows

## Daemon Architecture

Kage runs as a background daemon that:

- Listens on a Unix socket for commands
- Manages multiple agent processes
- Persists state across restarts
- Handles graceful shutdown

```
┌─────────────────────────────────────────────────┐
│                  Kage Daemon                    │
├─────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐              │
│  │   Agent 1   │  │   Agent 2   │   ...        │
│  │  (Claude)   │  │  (Claude)   │              │
│  └─────────────┘  └─────────────┘              │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │           Memory Store                   │   │
│  │  ┌──────────────┐  ┌──────────────┐     │   │
│  │  │   Working    │  │  Long-term   │     │   │
│  │  │   Memory     │  │   Memory     │     │   │
│  │  └──────────────┘  └──────────────┘     │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```
