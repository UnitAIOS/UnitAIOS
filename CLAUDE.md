# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language

Conversation with the user may happen in any language. All documentation, code
comments, commit messages, and other written artifacts in this repository must
always be in English.

## What UnitAIOS is

UnitAIOS is an abstraction of an operating system, exposed as a governable,
network-reachable service. A running deployment is an **instance**; each instance
exposes a controlled set of **tools** over a standard **MCP interface**. External
parties — peer UnitAIOS instances and AI agents — authenticate (mTLS for OS-to-OS,
tokens for agents) and get differentiated, per-tool access levels. An instance also
has a public unauthenticated endpoint for discovery, mandatory request journaling,
a tool-extension mechanism, self-introspection that produces local
**knowledge-atoms**, and a built-in **AI concierge** (exposed as an MCP tool) that
answers questions from those atoms. Connection modes (local stdio / network HTTP)
are configurable per instance.

The authoritative requirements live in [`Documents/BRD.md`](./Documents/BRD.md).
Read it before proposing design or implementation work.

## Repository status

Requirements are defined; **no source code, build system, dependency manifest, or
tests exist yet**. The repository currently holds only `LICENSE`, `README.md`, this
file, and `Documents/BRD.md`. There are therefore no build/lint/test commands to
run — do not invent any. Replace this section with real commands and architecture
once implementation begins.

## When adding the first real code

When the project's purpose and stack are decided, update this file to capture:

- **Build / run / test commands**, including how to run a single test.
- **High-level architecture** — the cross-file "big picture" that isn't obvious
  from reading any one file.

Keep the project metadata consistent across `README.md`, `LICENSE`, and the git
remote (`git@github.com:UnitAIOS/UnitAIOS.git`). The project was renamed from
`UnitOS` to `UnitAIOS` (commit `80b8c63`); watch for stale references to the old
name.
