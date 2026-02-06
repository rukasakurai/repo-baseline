# AGENTS.md

> **Canonical contribution guidelines live in [CONTRIBUTING.md](CONTRIBUTING.md).**
> This file adds only agent/tool-operational context. All general collaboration rules, constraints, and assumptions defined there apply here as well.

## Repository Purpose

This is a **baseline template repository** designed to provide a minimal, reusable starting point for new GitHub repositories. It establishes foundational structure and conventions without making premature decisions about specific technologies, licenses, or deployment patterns.

## Azure Provisioning and Testing

AI agents can be enabled to provision, deploy, and test against Azure autonomously using the [Azure Developer CLI Copilot Coding Agent Extension](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/extensions/copilot-coding-agent-extension). This removes the bottleneck of requiring a human to manually test `azd provision` and deployment. See [docs/azure-coding-agent-setup.md](docs/azure-coding-agent-setup.md) for setup instructions.

## Agent-Specific Constraints

- **No copilot-instructions.md**: Repository-specific instructions belong in derived repos
