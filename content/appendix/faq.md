+++
title = "FAQ"
description = "Frequently asked questions about Kage."
weight = 1
+++

# Frequently Asked Questions

## General

### What is Kage?

Kage is a local-first agentic work orchestrator that enables Claude Code agents to work autonomously, share context, and scale across multiple repositories.

### Why is it called Kage?

"Kage" (影) means "shadow" in Japanese. The name reflects the philosophy of shadow agents working invisibly, executing with precision while you focus on what matters.

### Is Kage open source?

Yes, Kage is open source under the Apache 2.0 license.

## Technical

### What agents does Kage support?

Currently, Kage has first-class support for Claude Code. The trait-based architecture allows extending to other providers (OpenAI, Gemini, etc.) in the future.

### Does Kage require a database?

No. Kage uses embedded storage (redb) that's compiled into the binary. There are no external dependencies.

### Does Kage work offline?

Yes. Kage is local-first and doesn't require any cloud services to operate. However, your AI agents may need internet access to function.

### Where is my data stored?

All data is stored locally on your machine:
- State: `~/.local/share/kage/`
- Config: `~/.config/kage/`
- Secrets: Your OS keychain (macOS Keychain, Linux Secret Service, Windows Credential Manager)

## Security

### How are secrets stored?

Secrets are stored in your operating system's native keychain:
- macOS: Keychain
- Linux: Secret Service (GNOME Keyring, KWallet)
- Windows: Credential Manager

Secrets are never written to disk in plaintext.

### Is it safe to run agents autonomously?

Kage provides guardrails like iteration limits and approval workflows. You can configure:
- Maximum iterations before pause
- Approval requirements for writes or commits
- Checkpoint intervals for review
