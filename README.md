# UnitAIOS

> An abstraction of an operating system, exposed as a governable, network-reachable
> service that other operating systems and AI agents can interact with.

## Overview

A running deployment of UnitAIOS is called an **instance**. Each instance exposes a
controlled set of **tools** (capabilities) through a standard **MCP interface**.

External parties — **peer UnitAIOS instances** and **AI agents** — authenticate to
an instance and are granted differentiated levels of access to its tools. A
separate **public, unauthenticated endpoint** lets anyone discover non-sensitive
information about an instance without gaining tool access.

Core properties of every instance:

- **Authentication & authorization** — mutual TLS (mTLS) for OS-to-OS trust, tokens
  for AI agents, with per-party, per-tool access levels.
- **MCP interface** — the standard surface for discovering and invoking tools.
- **Public endpoint** — anonymous discovery of basic instance information.
- **Mandatory journaling** — every request is recorded for audit.
- **Extensibility** — new tools can be added under published authoring rules and
  interaction protocols.
- **Self-knowledge** — an instance introspects itself and its tools to generate
  local **knowledge-atoms**.
- **AI concierge** — a built-in agent, exposed as a tool, that answers questions
  about the instance from those knowledge-atoms.
- **Configurable connection modes** — local (stdio) and/or network (HTTP), chosen
  at initialization.

## Status

🚧 Early development — requirements defined, implementation not yet started.

## Documentation

- [Business Requirements Document](./Documents/BRD.md) — what UnitAIOS must do and
  why (functional and non-functional requirements).

## Getting Started

```bash
git clone git@github.com:UnitAIOS/UnitAIOS.git
cd UnitAIOS
```

_Build and run instructions will be added once implementation begins._

## Contributing

Contributions are welcome. Please open an issue to discuss substantial changes
before submitting a pull request.

## License

[MIT](./LICENSE) © 2026 Gevorg Aznauryan
