+++
title = "Namespaces"
description = "Organizing repositories with namespaces."
weight = 2
+++

# Namespaces

Namespaces are how Kage organizes repositories and enables context sharing between agents.

## What are Namespaces?

A namespace is a logical grouping of repositories. For example:

- `backend` - All backend services
- `frontend` - Frontend applications
- `infra` - Infrastructure and DevOps

## Why Namespaces?

Namespaces enable:

- **Context sharing** - Agents in the same namespace can share discoveries
- **Scoped configuration** - Different settings per namespace
- **Organized CLI** - Filter agents by namespace

## Creating Namespaces

Namespaces are defined in your config:

```toml
# ~/.config/kage/config.toml

[namespaces.backend]
repos = [
    "~/code/api-server",
    "~/code/auth-service",
]

[namespaces.frontend]
repos = [
    "~/code/web-app",
    "~/code/mobile-app",
]
```

## Memory Sharing

Control how memory is shared within a namespace:

```toml
[namespaces.backend]
memory_sharing = "full"       # All memories shared automatically

[namespaces.experiments]
memory_sharing = "explicit"   # Only share what's marked
```
