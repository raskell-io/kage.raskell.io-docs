+++
title = "File Locations"
description = "Where Kage stores its files."
weight = 1
+++

# File Locations

Kage follows XDG conventions for file storage.

## Configuration Files

| File | Path |
|------|------|
| Global config | `~/.config/kage/config.toml` |
| Claude settings (global) | `~/.config/kage/claude.toml` |
| Namespace Claude settings | `~/.config/kage/namespaces/<name>/claude.toml` |
| Per-project config | `<repo>/.kage/config.toml` |

## State Files

| Purpose | Path |
|---------|------|
| State directory | `~/.local/share/kage/state/` |
| Agent registry | `~/.local/share/kage/state/agents.redb` |
| Task state | `~/.local/share/kage/state/tasks.redb` |
| Event logs | `~/.local/share/kage/state/events/` |
| Checkpoints | `~/.local/share/kage/state/checkpoints/` |

## Runtime Files

| Purpose | Path |
|---------|------|
| Daemon logs | `~/.local/share/kage/logs/` |
| Unix socket | `/tmp/kage.sock` or `$XDG_RUNTIME_DIR/kage.sock` |
| PID file | `/tmp/kage.pid` |

## Configuration Precedence

Configuration is loaded in order, with later values overriding earlier ones:

1. Built-in defaults
2. Global config (`~/.config/kage/config.toml`)
3. Namespace config
4. Repository config (`.kage/config.toml`)
5. Environment variables
6. CLI flags
