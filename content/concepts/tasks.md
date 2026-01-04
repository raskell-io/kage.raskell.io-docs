+++
title = "Tasks & Checkpoints"
description = "Autonomous task execution with checkpoints, criteria, and guardrails."
weight = 6
+++

# Tasks & Checkpoints

Tasks are the core unit of work in Kage. A task represents a goal you want an agent to achieve, with built-in checkpointing, iteration limits, and success/abort criteria.

## Task Lifecycle

```
                    ┌─────────┐
                    │ Pending │
                    └────┬────┘
                         │ Agent picks up
                         ▼
                    ┌─────────┐
              ┌─────│ Running │─────┐
              │     └────┬────┘     │
              │          │          │
         User pauses     │     Iteration limit
              │          │          │
              ▼          │          ▼
         ┌────────┐      │     ┌────────┐
         │ Paused │◄─────┼─────│ Paused │
         └────┬───┘      │     └────┬───┘
              │          │          │
         User resumes    │     User resumes
              │          │     with guidance
              └──────────┼──────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    Success          Abort           User
    criteria         criteria        cancels
    met              met
         │               │               │
         ▼               ▼               ▼
    ┌───────────┐  ┌────────┐    ┌───────────┐
    │ Completed │  │ Failed │    │ Cancelled │
    └───────────┘  └────────┘    └───────────┘
```

## Creating Tasks

```bash
kage task add "<goal>" [OPTIONS]
```

### Basic Example

```bash
kage task add "Fix the null pointer exception in the checkout flow"
```

### With Options

```bash
kage task add "Refactor the auth module to use JWT" \
    --namespace backend \
    --max-iterations 20 \
    --checkpoint-every 5 \
    --approval on-commit \
    --priority 150
```

## Iterations

An **iteration** is one cycle of the agent working on the task. Kage tracks iterations to:

1. **Prevent runaway agents** — Agents pause at the iteration limit
2. **Enable checkpointing** — Save state at regular intervals
3. **Allow human review** — Opportunity to provide guidance

### Iteration Limits

```bash
# Set max iterations (default: 10)
kage task add "Complex refactoring" --max-iterations 50
```

When the limit is reached:
- Agent pauses automatically
- Checkpoint is saved
- You can review progress and resume with additional iterations

### Resuming with More Iterations

```bash
kage task resume abc123 \
    --extend-iterations 20 \
    --guidance "Focus on the edge cases, the main flow looks good"
```

## Checkpoints

Checkpoints capture the agent's state at a point in time, allowing:

- **Resume from interruption** — Recover from crashes or pauses
- **Review progress** — See what the agent has done
- **Provide guidance** — Give feedback for the next phase

### Automatic Checkpoints

```bash
# Checkpoint every N iterations (default: 2)
kage task add "Build new feature" --checkpoint-every 3
```

### Managing Checkpoints

```bash
# List checkpoints for a task
kage task checkpoint list abc123

# View checkpoint details
kage task checkpoint show abc123

# View a specific checkpoint
kage task checkpoint show abc123 --checkpoint-id chk_01

# Clean up old checkpoints (keep last 3)
kage task checkpoint prune abc123 --keep 3
```

### What's in a Checkpoint?

- Agent context and memory
- Files modified so far
- Iteration count
- Task progress state
- Timestamp

## Success Criteria

Define conditions that mark a task as complete:

```bash
kage task add "Fix all tests" \
    --success-on "tests-pass" \
    --success-on "pattern:All tests passed"
```

### Available Criteria

| Criteria | Format | Description |
|----------|--------|-------------|
| Pattern match | `pattern:REGEX` | Match regex in agent output |
| File exists | `file-exists:PATH` | Check if file was created |
| File contains | `file-contains:PATH:PATTERN` | Check file for pattern |
| Command success | `command:CMD` | Run command, check exit code |
| Tests pass | `tests-pass` | Auto-detect and run test suite |

### Examples

```bash
# Complete when tests pass
--success-on "tests-pass"

# Complete when specific file is created
--success-on "file-exists:src/auth/jwt.ts"

# Complete when agent says it's done
--success-on "pattern:Implementation complete"

# Complete when build succeeds
--success-on "command:npm run build"

# Multiple criteria (all must be met)
--success-on "tests-pass" \
--success-on "file-exists:docs/api.md"
```

## Abort Criteria

Define conditions that should stop the task:

```bash
kage task add "Fix the bug" \
    --abort-on "error-count:5:compilation failed" \
    --abort-on "repeated-output:3:10"
```

### Available Criteria

| Criteria | Format | Description |
|----------|--------|-------------|
| Error count | `error-count:MAX:PATTERN` | Abort if errors exceed threshold |
| Repeated output | `repeated-output:MIN:WINDOW` | Detect stuck loops |
| Output contains | `output-contains:PHRASE1,PHRASE2` | Match abort phrases |
| No progress | `no-progress:ITERATIONS` | Abort if no file changes |

### Examples

```bash
# Abort after 5 compilation errors
--abort-on "error-count:5:compilation failed"

# Abort if same output repeats 3 times in 10 iterations
--abort-on "repeated-output:3:10"

# Abort on specific error messages
--abort-on "output-contains:fatal error,out of memory"

# Abort if no file changes for 5 iterations
--abort-on "no-progress:5"
```

## Task Dependencies

Create tasks that depend on other tasks:

```bash
# Create a parent task
kage task add "Set up database schema" --namespace backend
# Returns: task_abc123

# Create dependent task
kage task add "Implement API endpoints" \
    --depends-on task_abc123 \
    --namespace backend

# Create another dependent
kage task add "Write integration tests" \
    --depends-on task_abc123 \
    --namespace backend
```

Dependent tasks remain `pending` until their dependencies are `completed`.

## Priority

Tasks are scheduled by priority (0-255, higher = more urgent):

```bash
# High priority
kage task add "Critical bugfix" --priority 200

# Normal priority (default: 100)
kage task add "Feature work" --priority 100

# Low priority / batch work
kage task add "Code cleanup" --priority 50
```

## Task Status

| Status | Description |
|--------|-------------|
| `pending` | Waiting to be picked up or for dependencies |
| `running` | Agent is actively working |
| `paused` | Paused by user or iteration limit |
| `completed` | Success criteria met |
| `failed` | Abort criteria met or error |
| `cancelled` | Cancelled by user |

### Viewing Tasks

```bash
# List all tasks
kage task list

# Filter by status
kage task list --status running

# Filter by namespace
kage task list --namespace backend

# Show task details
kage task show abc123
```

## Workflow Examples

### Simple Fix

```bash
kage task add "Fix the login bug reported in issue #42"
```

### Guarded Feature Development

```bash
kage task add "Implement user profile page" \
    --max-iterations 30 \
    --checkpoint-every 5 \
    --approval on-commit \
    --success-on "tests-pass" \
    --abort-on "error-count:10:TypeError"
```

### Multi-Stage Pipeline

```bash
# Stage 1: Research
kage task add "Analyze the auth codebase and document patterns" \
    --approval none \
    --success-on "file-exists:docs/auth-analysis.md"
# Returns: task_1

# Stage 2: Implementation (depends on Stage 1)
kage task add "Implement OAuth2 based on the analysis" \
    --depends-on task_1 \
    --approval on-commit \
    --success-on "tests-pass"
# Returns: task_2

# Stage 3: Documentation (depends on Stage 2)
kage task add "Update API documentation for OAuth2" \
    --depends-on task_2 \
    --approval on-write
```

### Overnight Batch Work

```bash
kage task add "Migrate all API endpoints to new error format" \
    --max-iterations 100 \
    --checkpoint-every 10 \
    --priority 50 \
    --success-on "pattern:Migration complete" \
    --abort-on "error-count:20:migration failed"
```
