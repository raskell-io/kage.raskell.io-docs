# kage.raskell.io-docs

Documentation site for [Kage](https://github.com/raskell-io/kage).

**Live:** https://kage.raskell.io/docs

## Quick Start

```bash
# Install tools
mise install

# Start dev server
mise run serve
```

Visit http://127.0.0.1:1111

## Tasks

| Task | Description |
|------|-------------|
| `mise run serve` | Dev server with live reload |
| `mise run build` | Build for production |
| `mise run check` | Check for broken links |
| `mise run clean` | Clean build artifacts |

## Structure

```
kage.raskell.io-docs/
├── config.toml           # Zola configuration
├── content/
│   ├── _index.md         # Introduction
│   ├── getting-started/  # Installation, quick start
│   ├── concepts/         # Architecture, design
│   ├── configuration/    # Config reference
│   ├── cli/              # CLI reference
│   └── appendix/         # FAQ
├── syntaxes/             # Custom syntax highlighting
└── themes/tanuki/        # Documentation theme
```

## Writing Docs

Create `content/section/page.md`:

```markdown
+++
title = "Page Title"
weight = 1
+++

Content in Markdown...
```

## Tech Stack

- [Zola](https://www.getzola.org/) — Static site generator
- [mise](https://mise.jdx.dev/) — Task runner
- [Tanuki](https://github.com/raskell-io/tanuki) — Documentation theme

## Related

- [kage](https://github.com/raskell-io/kage) — Main repository
- [kage.raskell.io](https://github.com/raskell-io/kage.raskell.io) — Landing page
- [Discussions](https://github.com/raskell-io/kage/discussions) — Questions and ideas

## License

Apache 2.0
