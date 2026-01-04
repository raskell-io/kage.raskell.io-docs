+++
title = "Secrets Management"
description = "Secure credential storage using your OS keychain."
weight = 7
+++

# Secrets Management

Kage stores sensitive credentials (API keys, tokens, passwords) in your operating system's secure keychain. Secrets are never stored in plain text files.

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                      Kage CLI                           │
│  kage secret set API_KEY --scope global                 │
└─────────────────────┬───────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────┐
│              OS Keychain Integration                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  macOS     │  Keychain Access (keychain-services)       │
│  Linux     │  Secret Service (libsecret / GNOME Keyring)│
│  Windows   │  Credential Manager                         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

Your secrets are:
- **Encrypted at rest** by the OS
- **Protected by your login** — only accessible when you're logged in
- **Never written to disk** in plain text
- **Scoped** to specific contexts (global, namespace, or repo)

## Setting Secrets

```bash
# Set a global secret (prompted for value)
kage secret set API_KEY

# Set a namespace-scoped secret
kage secret set DB_PASSWORD --scope namespace:backend

# Set a repo-scoped secret
kage secret set DEPLOY_TOKEN --scope repo:~/code/api
```

When you run these commands, you'll be prompted to enter the secret value securely (input is hidden).

## Scope Levels

Secrets can be scoped to control where they're available:

| Scope | Format | Availability |
|-------|--------|--------------|
| Global | `global` | All agents, all repositories |
| Namespace | `namespace:<name>` | Agents in the specified namespace |
| Repository | `repo:<path>` | Agents in the specified repository |

### Scope Resolution

When an agent needs a secret, Kage resolves it from most specific to least:

1. **Repository scope** — Check for repo-specific secret
2. **Namespace scope** — Check namespace if repo is in one
3. **Global scope** — Fall back to global secret

This allows you to:
- Set a global default
- Override for specific namespaces
- Override for specific repos

### Example

```bash
# Global API key
kage secret set ANTHROPIC_API_KEY --scope global

# Different key for production namespace
kage secret set ANTHROPIC_API_KEY --scope namespace:production

# Specific key for a critical repo
kage secret set ANTHROPIC_API_KEY --scope repo:~/code/payments
```

When an agent runs in `~/code/payments`:
1. Uses the repo-scoped key (if set)
2. Falls back to namespace key (if in a namespace)
3. Falls back to global key

## Listing Secrets

```bash
# List all secret names (values are not shown)
kage secret list

# Filter by scope
kage secret list --scope global
kage secret list --scope namespace:backend
```

Output shows secret names, scopes, and when they were last updated:

```
Secrets
=======

┌──────────────────────┬─────────────────────┬──────────────────┐
│ Name                 │ Scope               │ Updated          │
├──────────────────────┼─────────────────────┼──────────────────┤
│ ANTHROPIC_API_KEY    │ global              │ 2 days ago       │
│ ANTHROPIC_API_KEY    │ namespace:production│ 1 day ago        │
│ DB_PASSWORD          │ namespace:backend   │ 1 week ago       │
│ DEPLOY_TOKEN         │ repo:~/code/api     │ 3 days ago       │
└──────────────────────┴─────────────────────┴──────────────────┘
```

## Deleting Secrets

```bash
# Delete a global secret
kage secret delete API_KEY

# Delete a scoped secret
kage secret delete DB_PASSWORD --scope namespace:backend
```

## Using Secrets in Agents

Secrets are automatically available to agents as environment variables. When an agent spawns, Kage:

1. Resolves all applicable secrets for the scope
2. Injects them as environment variables
3. The agent can use them naturally

### Example: Subscription API Keys

When you add a subscription, the API key is stored as a secret:

```bash
kage subscription add --name "work-account"
# Prompts for API key, stores in keychain
```

When agents use this subscription, Kage retrieves the key from the keychain and passes it to Claude Code.

## Best Practices

### Use Descriptive Names

```bash
# Good
kage secret set GITHUB_DEPLOY_TOKEN
kage secret set PROD_DATABASE_PASSWORD

# Avoid
kage secret set TOKEN
kage secret set PW
```

### Scope Appropriately

- Use **global** for secrets needed everywhere
- Use **namespace** for environment-specific secrets (dev, staging, prod)
- Use **repo** for project-specific credentials

### Rotate Regularly

```bash
# Update a secret (same command, new value)
kage secret set API_KEY --scope global
```

### Audit Your Secrets

```bash
# See what secrets exist
kage secret list

# Remove unused secrets
kage secret delete OLD_API_KEY
```

## Backend Configuration

By default, Kage uses the OS keychain. For enterprise deployments, you can configure alternative backends:

```toml
# ~/.config/kage/config.toml

[secrets]
# Default: OS keychain
backend = "keyring"

# AWS Secrets Manager (requires aws feature)
# backend = "aws"
# region = "us-east-1"

# Azure Key Vault (requires azure feature)
# backend = "azure"
# vault_url = "https://myvault.vault.azure.net/"
```

## Troubleshooting

### "Keychain access denied"

On macOS, you may need to grant Kage access to the keychain:

1. Open **Keychain Access**
2. Find entries starting with `kage-`
3. Right-click → Get Info → Access Control
4. Add the Kage binary

### "Secret not found"

Check the scope:

```bash
# List all secrets to see what exists
kage secret list

# Verify the exact scope
kage secret list --scope namespace:backend
```

### Linux: Secret Service not available

Install and enable the secret service:

```bash
# Debian/Ubuntu
sudo apt install gnome-keyring libsecret-1-0

# Start the keyring daemon
eval $(gnome-keyring-daemon --start)
export SSH_AUTH_SOCK
```
