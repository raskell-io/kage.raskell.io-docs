+++
title = "Introduction"
description = "Kage is a local-first agentic work orchestrator that enables Claude Code agents to work autonomously."
sort_by = "weight"
template = "section.html"
+++

# Welcome to Kage

Kage (影) is a local-first agentic work orchestrator written in Rust. It enables **Claude Code** agents to work autonomously while you're away, share context with each other, and scale across multiple repositories.

## What is Kage?

Kage acts as a **supervisor** for AI agents. Instead of just managing terminal sessions, Kage:

- **Spawns agents with specific goals** - Give your agents clear objectives
- **Monitors health and progress** - Track what your agents are doing
- **Enforces iteration limits** - Set guardrails to prevent runaway execution
- **Enables checkpoint/resume workflows** - Pick up where you left off

## Core Philosophy

> Shadow agents working invisibly, executing with precision.

The name "Kage" (影) means "shadow" in Japanese. Like shadows working behind the scenes, Kage enables AI agents to execute with precision while you focus on what matters.

## Key Features

### Local-First
Your data stays on your machine. Single binary with embedded storage. No cloud dependencies, no external services.

### Claude Code Native
First-class support for Claude Code as the primary agent. Trait-based architecture supports other providers when needed.

### Context Sharing
Two-tier memory system enables agents to share discoveries, learned patterns, and decisions with each other.

### Single Binary
Distributed as a single static binary with zero runtime dependencies. Just download and run.

## Quick Start

```bash
# Install Kage
curl -fsSL https://kage.raskell.io/install.sh | sh

# Start the daemon
kage daemon start

# Spawn an agent
kage agent spawn --repo ~/code/myproject

# Attach to the agent
kage agent attach <id>
```

## Next Steps

- [Installation](/docs/getting-started/installation/) - Get Kage installed on your system
- [Quick Start](/docs/getting-started/quick-start/) - Run your first autonomous agent
- [Concepts](/docs/concepts/) - Understand how Kage works
