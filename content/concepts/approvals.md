+++
title = "Approval Workflows"
description = "Control what agents can do with configurable approval gates."
weight = 5
+++

# Approval Workflows

Kage provides configurable approval workflows that let you control what actions agents can perform autonomously. This enables a balance between productivity and oversight.

## Approval Levels

When creating a task, specify an approval level:

```bash
kage task add "Refactor the auth module" --approval on-commit
```

| Level | Description | Use Case |
|-------|-------------|----------|
| `none` | No approval needed | Trusted, low-risk tasks |
| `on-write` | Approve file modifications | Code generation, refactoring |
| `on-commit` | Approve git commits | Production code changes |
| `always` | Approve all tool usage | Sensitive operations, learning |

### Level Details

#### `none`

The agent runs fully autonomously. Best for:
- Tasks in isolated/throwaway branches
- Research and exploration
- Documentation updates
- Test generation

```bash
kage task add "Generate unit tests for utils.ts" --approval none
```

#### `on-write`

The agent pauses before writing to any file. You see:
- File path and name
- Number of lines being added/removed
- Preview of the changes

Best for:
- Code generation
- Automated refactoring
- Configuration changes

```bash
kage task add "Add error handling to API endpoints" --approval on-write
```

#### `on-commit`

The agent can modify files freely but pauses before committing. You see:
- Commit message
- List of changed files
- Diff summary

Best for:
- Feature development
- Bug fixes
- Most day-to-day work

```bash
kage task add "Fix the login bug reported in #123" --approval on-commit
```

#### `always`

Every tool invocation requires approval. You see:
- Tool being called
- Full arguments
- Context of why it's being used

Best for:
- First-time tasks in critical repos
- Training yourself on agent patterns
- Compliance-sensitive work

```bash
kage task add "Update production config" --approval always
```

## Managing Approvals

### Viewing Pending Approvals

```bash
kage approval list
```

Output:
```
Pending Approvals (3)
=====================

┌────────┬─────────────────────────────────────┬───────────┬─────────┐
│ ID     │ Action                              │ Agent     │ Waiting │
├────────┼─────────────────────────────────────┼───────────┼─────────┤
│ apr_01 │ Write: src/auth/login.ts (+45, -12)│ agent_a1  │ 2m      │
│ apr_02 │ Commit: "Fix null check in auth"   │ agent_a1  │ 1m      │
│ apr_03 │ Write: config/prod.yaml (+3, -1)   │ agent_b2  │ 30s     │
└────────┴─────────────────────────────────────┴───────────┴─────────┘
```

### Viewing Details

```bash
kage approval show apr_01
```

Shows the full details of the proposed action, including:
- Complete diff for file writes
- Commit message and changed files for commits
- Tool arguments for other actions

### Approving

```bash
# Approve a single action
kage approval approve apr_01

# Approve all pending actions
kage approval approve all
```

### Rejecting

```bash
# Reject with a reason
kage approval reject apr_01 --reason "Don't modify this file, use the new auth module"
```

When you reject with a reason, the agent receives your feedback and adjusts its approach.

## Dashboard Integration

The TUI dashboard shows pending approvals in real-time:

```
┌─ Approvals (3 pending) ────────────────────────────────┐
│                                                         │
│ [a] apr_01  Write src/auth/login.ts           2m ago   │
│ [r] apr_02  Commit "Fix null check"           1m ago   │
│     apr_03  Write config/prod.yaml           30s ago   │
│                                                         │
│ [a]pprove  [r]eject  [Enter] details  [A]pprove all   │
└─────────────────────────────────────────────────────────┘
```

Keyboard shortcuts:
- `a` — Approve selected
- `r` — Reject selected (prompts for reason)
- `Enter` — View details
- `A` — Approve all pending

## Approval Flow

```
Agent wants to perform action
           │
           ▼
┌─────────────────────────┐
│  Check approval level   │
└───────────┬─────────────┘
            │
    ┌───────┴───────┐
    │               │
    ▼               ▼
 Approval       No Approval
 Required       Required
    │               │
    ▼               │
┌───────────┐       │
│  Pause    │       │
│  Agent    │       │
└─────┬─────┘       │
      │             │
      ▼             │
┌───────────────┐   │
│ Wait for user │   │
│   response    │   │
└───────┬───────┘   │
        │           │
   ┌────┴────┐      │
   │         │      │
   ▼         ▼      │
Approved  Rejected  │
   │         │      │
   │         ▼      │
   │    ┌─────────┐ │
   │    │ Agent   │ │
   │    │ adjusts │ │
   │    │ approach│ │
   │    └────┬────┘ │
   │         │      │
   └────┬────┘      │
        │           │
        ▼           ▼
   ┌─────────────────┐
   │ Continue task   │
   └─────────────────┘
```

## Approval Types

### File Write

Shows the proposed file changes:

```
Approval Request: Write File
============================

File: src/auth/login.ts
Lines: +45, -12

Preview:
───────────────────────────────────
@@ -23,12 +23,45 @@
 export async function login(credentials: Credentials) {
-  const user = await db.findUser(credentials.email);
-  if (!user) {
-    throw new Error('User not found');
-  }
+  try {
+    const user = await db.findUser(credentials.email);
+    if (!user) {
+      return { success: false, error: 'INVALID_CREDENTIALS' };
+    }
+
+    const validPassword = await verifyPassword(
+      credentials.password,
+      user.passwordHash
+    );
...
───────────────────────────────────

[a]pprove  [r]eject  [v]iew full diff
```

### Git Commit

Shows the commit details:

```
Approval Request: Git Commit
============================

Message: Fix null pointer in auth flow

Files changed (3):
  M src/auth/login.ts
  M src/auth/session.ts
  A src/auth/types.ts

Stats: +67, -23

[a]pprove  [r]eject  [d]iff
```

### Tool Usage (always mode)

Shows the tool invocation:

```
Approval Request: Tool Usage
============================

Tool: Bash
Command: npm install jsonwebtoken

Context: Installing JWT library for token generation

[a]pprove  [r]eject
```

## Best Practices

### Start Strict, Relax Over Time

When working with a new repository or task type, start with `--approval always` to understand how the agent works. As you build trust, relax to `on-commit` or `on-write`.

### Use Rejection Reasons

Always provide a reason when rejecting:

```bash
kage approval reject apr_01 --reason "Use the existing auth utils instead of creating new ones"
```

This helps the agent learn your preferences and improve subsequent attempts.

### Batch Approvals Carefully

`kage approval approve all` is convenient but use it only when you've reviewed the pending items in the dashboard first.

### Namespace Defaults

Set default approval levels per namespace in your config:

```toml
[namespaces.production]
default_approval = "always"

[namespaces.experiments]
default_approval = "none"
```

## Configuration

### Global Default

```toml
# ~/.config/kage/config.toml
[tasks]
default_approval = "on-commit"
```

### Per-Namespace

```toml
[namespaces.backend]
approval = "on-commit"

[namespaces.infrastructure]
approval = "always"
```

### Timeout Settings

```toml
[approvals]
# Auto-pause task if no response within timeout
timeout = "30m"

# Notify via system notification
notify = true
```
