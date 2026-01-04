+++
title = "Configuration Options"
description = "Complete reference for all configuration options."
weight = 2
+++

# Configuration Options

Kage uses TOML for configuration. This page documents all available options.

## Global Configuration

Located at `~/.config/kage/config.toml`:

```toml
# Daemon settings
[daemon]
# Unix socket path for IPC
socket_path = "/tmp/kage.sock"

# Directory for persistent state (agents, tasks, events)
state_dir = "~/.local/share/kage/state/"

# Directory for daemon logs
log_dir = "~/.local/share/kage/logs/"

# Maximum concurrent agents
max_agents = 10

# gRPC listen address (enables remote mode)
# grpc_listen = "0.0.0.0:50051"

# Claude Code agent settings
[claude]
# Default model (opus, sonnet, haiku)
model = "sonnet"

# Maximum iterations before pause
max_iterations = 50

# Default approval level (none, on-write, on-commit, always)
approval = "on-commit"

# Path to custom CLAUDE.md instruction file
# claude_md = "/path/to/custom/CLAUDE.md"

# Memory system settings
[memory]
# Storage backend (filesystem, s3, azure)
backend = "filesystem"

# Working memory TTL
working_memory_ttl = "24h"

# Long-term memory retention
long_term_retention = "90d"

# Maximum entries per namespace
max_entries_per_namespace = 10000

# Secrets backend
[secrets]
# Backend type (keyring, aws, azure)
backend = "keyring"

# Task defaults
[tasks]
# Default approval level for new tasks
default_approval = "on-commit"

# Default max iterations
default_max_iterations = 10

# Default checkpoint interval
default_checkpoint_every = 2

# Approval settings
[approvals]
# Timeout before auto-pausing task
timeout = "30m"

# Enable system notifications
notify = true

# Subscription pool settings
[subscriptions]
# Default routing strategy (round-robin, priority, least-used)
default_priority = "priority"

# Rate limit window duration
rate_limit_window = "1m"

# Cooldown after hitting rate limit
rate_limit_cooldown = "5m"
```

## Namespace Configuration

Define namespaces in your global config:

```toml
[namespaces.backend]
# Repositories in this namespace
repos = [
    "~/code/api",
    "~/code/auth",
    "~/code/worker"
]

# Memory sharing mode (full, isolated, cascade)
memory_sharing = "full"

# Namespace-specific approval level
approval = "on-commit"

# Maximum concurrent agents in this namespace
max_agents = 5

[namespaces.frontend]
repos = [
    "~/code/web",
    "~/code/mobile"
]
memory_sharing = "cascade"
approval = "on-write"
```

### Memory Sharing Modes

| Mode | Description |
|------|-------------|
| `full` | All agents share all memory within the namespace |
| `isolated` | Agents only see their own memory |
| `cascade` | Agents see shared + their own memory |

## Subscription Routing

Configure subscription routing rules:

```toml
[subscriptions.routing]
# Route namespace to specific subscriptions
backend = ["backend-dedicated", "work-premium"]
frontend = ["frontend-sub", "personal"]

# Route priority levels
high = ["premium-1", "premium-2"]
normal = ["work-1", "work-2"]
batch = ["batch-1", "batch-2", "batch-3"]
```

## Per-Project Configuration

Located at `<repo>/.kage/config.toml`:

```toml
# Override namespace assignment
namespace = "backend"

# Project-specific Claude settings
[claude]
model = "opus"
max_iterations = 100

# Custom CLAUDE.md for this project
claude_md = ".kage/CLAUDE.md"

# Project-specific approval level
[tasks]
default_approval = "on-commit"

# Custom memory settings
[memory]
# Tags automatically applied to memory from this repo
auto_tags = ["api", "backend"]
```

## Environment Variables

All configuration options can be overridden via environment variables:

| Variable | Description |
|----------|-------------|
| `KAGE_SOCKET_PATH` | Unix socket path |
| `KAGE_STATE_DIR` | State directory |
| `KAGE_LOG_DIR` | Log directory |
| `KAGE_MAX_AGENTS` | Maximum concurrent agents |
| `KAGE_CLAUDE_MODEL` | Default Claude model |
| `KAGE_CLAUDE_MAX_ITERATIONS` | Default max iterations |
| `KAGE_APPROVAL` | Default approval level |

Environment variables follow the pattern `KAGE_<SECTION>_<KEY>` in uppercase with underscores.

## Claude-Specific Settings

The `~/.config/kage/claude.toml` file contains Claude-specific settings:

```toml
# Model selection
model = "sonnet"  # opus, sonnet, haiku

# Allowed tools (empty = all allowed)
allowed_tools = []

# Denied tools
denied_tools = ["Bash(rm -rf:*)"]

# MCP servers to enable
[mcp.servers.memory]
command = "npx"
args = ["-y", "@anthropic/mcp-server-memory"]
enabled = true

[mcp.servers.filesystem]
command = "npx"
args = ["-y", "@anthropic/mcp-server-filesystem", "--root", "~"]
enabled = true
```

## Example: Complete Configuration

Here's a comprehensive example configuration:

```toml
# ~/.config/kage/config.toml

[daemon]
socket_path = "/tmp/kage.sock"
state_dir = "~/.local/share/kage/state/"
log_dir = "~/.local/share/kage/logs/"
max_agents = 8

[claude]
model = "sonnet"
max_iterations = 30
approval = "on-commit"

[memory]
backend = "filesystem"
working_memory_ttl = "24h"
long_term_retention = "90d"

[secrets]
backend = "keyring"

[tasks]
default_approval = "on-commit"
default_max_iterations = 15
default_checkpoint_every = 3

[approvals]
timeout = "1h"
notify = true

[subscriptions]
default_priority = "priority"

[subscriptions.routing]
backend = ["work-main", "work-backup"]
high = ["premium"]

[namespaces.backend]
repos = ["~/code/api", "~/code/auth"]
memory_sharing = "full"
max_agents = 4

[namespaces.frontend]
repos = ["~/code/web"]
memory_sharing = "cascade"
approval = "on-write"
max_agents = 2

[namespaces.infrastructure]
repos = ["~/code/infra", "~/code/deploy"]
approval = "always"
max_agents = 1
```
