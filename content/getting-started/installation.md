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
cargo install raskell-kage
```

## Docker

Kage is available as a multi-architecture Docker image:

```bash
docker pull ghcr.io/raskell-io/kage:latest
```

Run as a container:

```bash
docker run -d \
  --name kage \
  -v ~/.config/kage:/etc/kage \
  -v ~/.local/share/kage:/var/lib/kage \
  ghcr.io/raskell-io/kage:latest
```

Available tags:
- `latest` - Latest stable release
- `v0.1.2` - Specific version
- `v0.1.2-amd64` - Specific architecture

Supported architectures: `linux/amd64`, `linux/arm64`

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
