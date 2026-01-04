+++
title = "Subscription Pooling"
description = "Scale your development capacity with multiple Claude Code subscriptions."
weight = 4
+++

# Subscription Pooling

Kage supports registering multiple Claude Code subscriptions to enable parallel productivity scaling. This allows you to run multiple agents simultaneously, each using a different subscription to avoid rate limits.

## Why Multiple Subscriptions?

A single Claude Code subscription has usage limits. When you hit those limits, you need to wait before continuing work. With subscription pooling, Kage automatically routes requests across multiple subscriptions, giving you:

- **Parallel scaling** — Run more agents simultaneously
- **Rate limit avoidance** — Rotate to available subscriptions when one hits limits
- **Failover** — Automatic fallback when a subscription becomes unavailable
- **Usage tracking** — Per-subscription metrics for cost management

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                   Subscription Pool                      │
├─────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │   Sub 1  │  │   Sub 2  │  │   Sub 3  │   ...        │
│  │ Active   │  │ Active   │  │ Rate     │              │
│  │ Pri: 150 │  │ Pri: 100 │  │ Limited  │              │
│  └────┬─────┘  └────┬─────┘  └──────────┘              │
│       │             │                                   │
│       └──────┬──────┘                                   │
│              ▼                                          │
│     ┌─────────────────┐                                │
│     │  Load Balancer  │                                │
│     │  (Priority +    │                                │
│     │   Health-aware) │                                │
│     └────────┬────────┘                                │
└──────────────┼──────────────────────────────────────────┘
               ▼
         Agent Requests
```

### Subscription Selection

When an agent needs to make a request, Kage selects a subscription based on:

1. **Health status** — Only active, healthy subscriptions are considered
2. **Priority** — Higher priority (0-255) subscriptions are preferred
3. **Tags** — Routing rules can direct specific namespaces to specific subscriptions
4. **Rate limits** — Subscriptions approaching limits are deprioritized

### Subscription States

| State | Description |
|-------|-------------|
| `active` | Healthy and available for requests |
| `rate_limited` | Hit rate limit, will auto-recover after cooldown |
| `exhausted` | Quota exhausted, waiting for reset |
| `disabled` | Manually disabled by user |
| `unhealthy` | Experiencing errors, temporarily removed from pool |

## Adding Subscriptions

```bash
# Add a subscription with default settings
kage subscription add --name "personal"

# Add a high-priority subscription
kage subscription add \
    --name "work-premium" \
    --priority 150 \
    --description "Work account with higher limits"

# Add a tagged subscription for specific routing
kage subscription add \
    --name "backend-dedicated" \
    --tags "backend,api" \
    --priority 120
```

When adding a subscription, you'll be prompted to enter the API credentials securely. Credentials are stored in your OS keychain, never in plain text.

## Routing Configuration

You can configure routing rules to direct specific workloads to specific subscriptions.

### By Namespace

Route all agents in a namespace to specific subscriptions:

```bash
kage subscription route \
    --namespace backend \
    --subscriptions "backend-dedicated,work-premium"
```

### By Priority Level

Route high-priority tasks to premium subscriptions:

```bash
# High-priority tasks get premium subscriptions
kage subscription route \
    --priority high \
    --subscriptions "premium-1,premium-2"

# Batch/background work uses lower-tier subscriptions
kage subscription route \
    --priority batch \
    --subscriptions "batch-1,batch-2,batch-3"
```

## Monitoring

### Pool Status

```bash
kage subscription status
```

Shows an overview of all subscriptions:

```
Subscription Pool Status
========================

Active:       3/4
Rate Limited: 1
Total Usage:  45,230 tokens today

┌──────────────────┬──────────┬──────────┬─────────────┐
│ Name             │ Status   │ Priority │ Today Usage │
├──────────────────┼──────────┼──────────┼─────────────┤
│ work-premium     │ active   │ 150      │ 12,450      │
│ personal         │ active   │ 100      │ 18,200      │
│ backend-dedicated│ active   │ 120      │ 14,580      │
│ overflow         │ rate_ltd │ 50       │ —           │
└──────────────────┴──────────┴──────────┴─────────────┘
```

### Usage Statistics

```bash
# Weekly usage
kage subscription usage --period week

# Usage for specific subscription
kage subscription usage --subscription work-premium --period month
```

## Best Practices

### Subscription Naming

Use descriptive names that indicate the subscription's purpose:

```bash
# Good
--name "work-main"
--name "personal-backup"
--name "ci-pipeline"

# Avoid
--name "sub1"
--name "account"
```

### Priority Strategy

- **150-255**: Premium/emergency subscriptions
- **100-149**: Primary work subscriptions
- **50-99**: Secondary/overflow subscriptions
- **0-49**: Batch/background work

### Tag-Based Routing

Use tags to create logical groups:

```bash
# Security-sensitive work uses audited subscriptions
kage subscription add --name "sec-audited" --tags "security,compliance"
kage subscription route --namespace security --subscriptions "sec-audited"
```

### Handling Rate Limits

When a subscription hits rate limits:

1. Kage automatically routes to the next available subscription
2. The rate-limited subscription enters cooldown
3. After cooldown, it automatically rejoins the pool

You don't need to manually intervene unless all subscriptions are rate-limited.

## Configuration File

Subscriptions can also be configured in `~/.config/kage/config.toml`:

```toml
[subscriptions]
# Default routing priority
default_priority = "round-robin"  # or "priority", "least-used"

# Rate limit settings
rate_limit_window = "1m"
rate_limit_cooldown = "5m"

[subscriptions.routing]
# Namespace-specific routing
backend = ["backend-dedicated", "work-premium"]
frontend = ["frontend-1", "personal"]

# Priority-level routing
high = ["premium-1", "premium-2"]
batch = ["batch-1", "batch-2"]
```
