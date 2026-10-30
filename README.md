# Home AI docs

Planning documents for a local-first home automation and AI agent hub: one Claude-based agent makes decisions, Home Assistant executes device commands, and sandboxed subagents do background work with phone approval for anything sensitive.

## Two setups

The docs describe two hardware setups and say, section by section, what runs where in each:

- **Mac mini only** (Phases 0 and 1): the Mac mini runs the house, voice and the agent service with cloud models; agent workspaces run on a rented cloud VM. It is a complete system on its own.
- **Mac mini + Mac Studio** (Phase 2 on): the mini keeps the house, vault and egress routes; the Studio runs voice, agents, workspaces and local models, and the cloud VM is retired.

The full side-by-side table is under "What runs where in each setup" in the design doc.

## Documents

| Document | Markdown | HTML |
|---|---|---|
| Design doc | [docs/design-doc.md](docs/design-doc.md) | [docs/design-doc.html](docs/design-doc.html) |
| Roadmap: epics, stories, tasks | [docs/roadmap.md](docs/roadmap.md) | [docs/roadmap.html](docs/roadmap.html) |

The HTML files are generated from the Markdown with `pandoc -f gfm -t html5 -s`. Edit the Markdown.

## Never commit

This repo is public. Do not commit API keys, passwords, tokens, IP addresses, hostnames, or account details. Keep secrets in the vault and reference them by name only.
