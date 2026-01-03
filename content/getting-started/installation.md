+++
title = "Installation"
description = "Install Kage on your system."
weight = 1
+++

# Installation

Kage is distributed as a single static binary with zero runtime dependencies. Choose your preferred installation method below.

## Quick Install (Recommended)

The fastest way to install Kage is using the install script:

```bash
curl -fsSL https://kage.raskell.io/install.sh | sh
```

This will:
- Detect your operating system and architecture
- Download the appropriate binary
- Install it to `~/.local/bin` (or `/usr/local/bin` with sudo)
- Add it to your PATH if needed

## Cargo

If you have Rust installed, you can install Kage via Cargo:

```bash
cargo install kage
```

## From Source

Clone the repository and build:

```bash
git clone https://github.com/raskell-io/kage.git
cd kage
cargo build --release
```

The binary will be at `target/release/kage`.

## Verify Installation

After installation, verify Kage is working:

```bash
kage --version
```

## Next Steps

- [Quick Start](/docs/getting-started/quick-start/) - Run your first autonomous agent
