# AGENTS.md

> **Canonical contribution guidelines live in [CONTRIBUTING.md](CONTRIBUTING.md).**
> This file adds only agent/tool-operational context. All general collaboration rules, constraints, and assumptions defined there apply here as well.

## Repository Purpose

This is a **baseline template repository** designed to provide a minimal, reusable starting point for new GitHub repositories. It establishes foundational structure and conventions without making premature decisions about specific technologies, licenses, or deployment patterns.

## Working Style

**Simplicity.** Verbosity and complexity are harmful by default; deleting and simplifying almost always adds value. When asked to fix, prefer fixing in place over adding. Make a habit of asking: "what can I cut?"

## Azure Access

The [Azure Developer CLI Copilot Coding Agent Extension](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/extensions/copilot-coding-agent-extension) can give agents read-time visibility into Azure state while authoring changes. See [docs/azure-coding-agent-guide.md](docs/azure-coding-agent-guide.md) for guidance.