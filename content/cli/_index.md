+++
title = "CLI Reference"
description = "Complete command-line interface reference for Kage."
weight = 4
sort_by = "weight"
template = "section.html"
+++

# CLI Reference

Complete reference for all Kage CLI commands.

## Global Usage

```bash
kage [COMMAND]
kage --help
kage --version
```

Running `kage` without arguments launches the interactive dashboard (if the `tui` feature is enabled).

---

## Dashboard

```bash
kage dashboard
```

Launch the interactive TUI dashboard for real-time monitoring of agents, tasks, and approvals.

---

## Daemon Commands

Manage the Kage daemon process.

### Start

```bash
kage daemon start [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-f, --foreground` | Run in foreground (don't daemonize) |

**Examples:**
```bash
# Start daemon in background
kage daemon start

# Start in foreground for debugging
kage daemon start --foreground
```

### Stop

```bash
kage daemon stop
```

Gracefully stop the running daemon.

### Status

```bash
kage daemon status
```

Show daemon status including uptime, active agents, and pending tasks.

---

## Agent Commands

Manage Claude Code agents.

### Spawn

```bash
kage agent spawn [OPTIONS]
```

Spawn a new agent in the current or specified repository.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-r, --repo <PATH>` | Repository path | `.` (current directory) |
| `-n, --namespace <NAME>` | Namespace to use | — |
| `-p, --prompt <GOAL>` | Initial prompt/goal | — |

**Examples:**
```bash
# Spawn agent in current directory
kage agent spawn

# Spawn with a specific goal
kage agent spawn --prompt "Fix the failing tests in src/auth"

# Spawn in a different repo within a namespace
kage agent spawn --repo ~/code/api --namespace backend
```

### List

```bash
kage agent list [OPTIONS]
```

List active agents.

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --namespace <NAME>` | Filter by namespace |
| `--json` | Output as JSON |

### Attach

```bash
kage agent attach <ID>
```

Attach to an agent interactively. You'll see the agent's output in real-time and can provide input.

**Arguments:**
- `<ID>` — Agent ID or name

### Detach

```bash
kage agent detach
```

Detach from the currently attached agent. The agent continues running in the background.

### Kill

```bash
kage agent kill <ID> [OPTIONS]
```

Terminate an agent.

**Arguments:**
- `<ID>` — Agent ID or name

**Options:**
| Flag | Description |
|------|-------------|
| `-f, --force` | Force kill without confirmation |

### Status

```bash
kage agent status <ID>
```

Show agent status and recent output.

**Arguments:**
- `<ID>` — Agent ID or name

---

## Task Commands

Manage autonomous tasks with checkpoints and approval workflows.

### Add

```bash
kage task add "<GOAL>" [OPTIONS]
```

Create a new task for an agent to work on.

**Arguments:**
- `<GOAL>` — Task goal in natural language

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-n, --namespace <NAME>` | Namespace | — |
| `-r, --repo <PATH>` | Repository | — |
| `--max-iterations <N>` | Maximum iterations before pause | `10` |
| `--checkpoint-every <N>` | Checkpoint interval | `2` |
| `--approval <LEVEL>` | Approval level | `on-commit` |
| `-p, --priority <0-255>` | Priority (higher = more urgent) | `100` |
| `--depends-on <ID>` | Depends on task (repeatable) | — |
| `--success-on <CRITERIA>` | Success criteria (repeatable) | — |
| `--abort-on <CRITERIA>` | Abort criteria (repeatable) | — |

**Approval Levels:**
- `none` — No approval needed
- `on-write` — Approve file modifications
- `on-commit` — Approve git commits
- `always` — Approve all actions

**Success Criteria Formats:**
- `pattern:REGEX` — Match regex in agent output
- `file-exists:PATH` — Check if file was created
- `file-contains:PATH:PATTERN` — Check file contains pattern
- `command:CMD` — Run command and check exit code
- `tests-pass` — Auto-detect and run test suite

**Abort Criteria Formats:**
- `error-count:MAX:PATTERN` — Abort if errors exceed threshold
- `repeated-output:MIN:WINDOW` — Detect stuck loops
- `output-contains:PHRASE1,PHRASE2` — Match abort phrases
- `no-progress:ITERATIONS` — Abort if no file changes

**Examples:**
```bash
# Simple task
kage task add "Refactor the auth module to use JWT"

# Task with success criteria
kage task add "Fix the bug in user registration" \
    --success-on "tests-pass" \
    --success-on "pattern:All tests passed"

# Task with abort criteria and dependencies
kage task add "Deploy to staging" \
    --depends-on abc123 \
    --abort-on "error-count:3:deployment failed" \
    --approval always
```

### List

```bash
kage task list [OPTIONS]
```

List tasks.

**Options:**
| Flag | Description |
|------|-------------|
| `-s, --status <STATUS>` | Filter by status |
| `-n, --namespace <NAME>` | Filter by namespace |
| `--json` | Output as JSON |

**Status Values:** `pending`, `running`, `paused`, `completed`, `failed`, `cancelled`

### Show

```bash
kage task show <ID>
```

Display task details including goal, status, iterations, and checkpoints.

### Pause

```bash
kage task pause <ID>
```

Pause a running task. The agent will stop after the current iteration.

### Resume

```bash
kage task resume <ID> [OPTIONS]
```

Resume a paused task.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `--extend-iterations <N>` | Extend iteration limit | `5` |
| `--guidance <TEXT>` | Provide guidance for next steps | — |

**Example:**
```bash
kage task resume abc123 \
    --extend-iterations 10 \
    --guidance "Focus on the edge cases in the payment flow"
```

### Cancel

```bash
kage task cancel <ID> [OPTIONS]
```

Cancel a task.

**Options:**
| Flag | Description |
|------|-------------|
| `-f, --force` | Force cancel without confirmation |

### Checkpoint Subcommands

Manage task checkpoints.

```bash
# List checkpoints for a task
kage task checkpoint list <TASK_ID>

# Show checkpoint details
kage task checkpoint show <TASK_ID> [OPTIONS]
    --checkpoint-id <ID>    # Specific checkpoint (defaults to latest)

# Prune old checkpoints
kage task checkpoint prune <TASK_ID> [OPTIONS]
    --keep <N>              # Number to keep (default: 3)
```

---

## Memory Commands

Manage the two-tier memory system.

### List

```bash
kage memory list [OPTIONS]
```

List memory entries.

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --namespace <NAME>` | Filter by namespace |
| `-t, --type <TYPE>` | Filter by type |
| `--json` | Output as JSON |

**Memory Types:** `patterns`, `errors`, `decisions`, `context`, `insights`, `questions`

### Search

```bash
kage memory search "<QUERY>" [OPTIONS]
```

Full-text search across memory.

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --namespace <NAME>` | Filter by namespace |

### Show

```bash
kage memory show <ID>
```

Display a specific memory entry.

### Share

```bash
kage memory share <ID> --scope <SCOPE>
```

Share a memory entry to a wider scope.

**Scope Values:** `namespace`, `global`

### Watch

```bash
kage memory watch [OPTIONS]
```

Stream memory updates in real-time.

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --namespace <NAME>` | Filter by namespace |

### Export

```bash
kage memory export [OPTIONS]
```

Export memory to a file.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-f, --format <FORMAT>` | Output format (`json`, `markdown`) | `json` |
| `-o, --output <PATH>` | Output file path | — |

### Prune

```bash
kage memory prune [OPTIONS]
```

Delete old memory entries.

**Options:**
| Flag | Description |
|------|-------------|
| `--older-than <DURATION>` | Delete entries older than (e.g., `30d`, `1w`) |
| `--dry-run` | Show what would be deleted |

---

## Namespace Commands

Manage namespace groupings for repositories.

### Create

```bash
kage namespace create <NAME>
```

### List

```bash
kage namespace list
```

### Show

```bash
kage namespace show <NAME>
```

### Add Repository

```bash
kage namespace add-repo <NAMESPACE> <PATH> [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --name <NAME>` | Repository name (defaults to directory name) |

### Remove Repository

```bash
kage namespace remove-repo <NAMESPACE> <REPO>
```

### Delete

```bash
kage namespace delete <NAME> [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-f, --force` | Force delete without confirmation |

---

## Secret Commands

Manage secrets stored in the OS keychain.

### Set

```bash
kage secret set <NAME> [OPTIONS]
```

Store a secret securely. You'll be prompted to enter the value.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-s, --scope <SCOPE>` | Scope level | `global` |

**Scope Formats:**
- `global` — Available everywhere
- `namespace:<name>` — Available in namespace
- `repo:<path>` — Available in repository

### List

```bash
kage secret list [OPTIONS]
```

List secret names (not values).

**Options:**
| Flag | Description |
|------|-------------|
| `-s, --scope <SCOPE>` | Filter by scope |

### Delete

```bash
kage secret delete <NAME> [OPTIONS]
```

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-s, --scope <SCOPE>` | Scope | `global` |

---

## Subscription Commands

Manage multiple Claude Code subscriptions for parallel scaling.

### Add

```bash
kage subscription add [OPTIONS]
```

Register a new subscription. You'll be prompted to enter the API key.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-n, --name <NAME>` | Subscription name (required) | — |
| `-p, --provider <TYPE>` | Provider type | `claude-code` |
| `--priority <0-255>` | Priority (higher = used first) | `100` |
| `-t, --tags <TAGS>` | Tags for routing (comma-separated) | — |
| `-d, --description <TEXT>` | Description | — |

**Example:**
```bash
kage subscription add \
    --name "work-account" \
    --priority 150 \
    --tags "high-priority,backend" \
    --description "Work Claude Code Max subscription"
```

### List

```bash
kage subscription list [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-s, --status <STATUS>` | Filter by status |
| `--json` | Output as JSON |

**Status Values:** `active`, `disabled`, `rate_limited`, `unhealthy`

### Show

```bash
kage subscription show <NAME>
```

Display subscription details including health and usage.

### Remove

```bash
kage subscription remove <NAME> [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-f, --force` | Force remove without confirmation |

### Enable / Disable

```bash
kage subscription enable <NAME>
kage subscription disable <NAME>
```

### Status

```bash
kage subscription status
```

Show pool overview including health of all subscriptions.

### Usage

```bash
kage subscription usage [OPTIONS]
```

View usage statistics.

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-p, --period <PERIOD>` | Time period | `week` |
| `-s, --subscription <NAME>` | Filter by subscription | — |
| `--json` | Output as JSON |

**Period Values:** `today`, `week`, `month`

### Route

```bash
kage subscription route [OPTIONS]
```

Configure routing rules.

**Options:**
| Flag | Description |
|------|-------------|
| `-n, --namespace <NAME>` | Namespace to route |
| `-p, --priority <LEVEL>` | Priority level (`high`, `normal`, `batch`) |
| `-s, --subscriptions <NAMES>` | Subscriptions to route to (comma-separated) |

**Example:**
```bash
# Route high-priority tasks to premium subscriptions
kage subscription route \
    --priority high \
    --subscriptions "premium-1,premium-2"

# Route backend namespace to specific subscriptions
kage subscription route \
    --namespace backend \
    --subscriptions "backend-sub-1,backend-sub-2"
```

---

## Approval Commands

Manage approval requests from agents.

### List

```bash
kage approval list [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `--json` | Output as JSON |

### Approve

```bash
kage approval approve <ID>
```

Approve an action. Use `"all"` to approve all pending.

### Reject

```bash
kage approval reject <ID> [OPTIONS]
```

**Options:**
| Flag | Description |
|------|-------------|
| `-r, --reason <TEXT>` | Reason for rejection |

### Show

```bash
kage approval show <ID>
```

View approval details including the proposed action.

---

## Server Commands

*Requires the `server` feature.*

### Start

```bash
kage server start [OPTIONS]
```

**Options:**
| Flag | Description | Default |
|------|-------------|---------|
| `-l, --listen <ADDR>` | Listen address | `0.0.0.0:50051` |
| `--tls` | Enable TLS | — |
| `--multiuser` | Enable multi-user mode | — |

### Stop

```bash
kage server stop
```

### Enable Multi-user

```bash
kage server enable-multiuser
```

### Status

```bash
kage server status
```

---

## User Commands

*Requires the `server` feature.*

### Invite

```bash
kage user invite <EMAIL>
```

### List

```bash
kage user list
```

### Remove

```bash
kage user remove <ID>
```

---

## Connect

*Requires the `server` feature.*

```bash
kage connect <ADDRESS>
```

Connect to a remote Kage server.

**Example:**
```bash
kage connect vps.example.com:50051
```
