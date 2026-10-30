# Home AI docs

Planning documents for a local-first home automation and AI agent hub: one Claude-based agent makes decisions, Home Assistant executes device commands, and sandboxed subagents do background work with phone approval for anything sensitive.

| Document | Markdown | HTML |
|---|---|---|
| Design doc | [docs/design-doc.md](docs/design-doc.md) | [docs/design-doc.html](docs/design-doc.html) |
| Roadmap: epics, stories, tasks | [docs/roadmap.md](docs/roadmap.md) | [docs/roadmap.html](docs/roadmap.html) |

The HTML files are generated from the Markdown with `pandoc -f gfm -t html5 -s`. Edit the Markdown.

## Never commit

This repo is public. Do not commit API keys, passwords, tokens, IP addresses, hostnames, or account details. Keep secrets in the vault and reference them by name only.
