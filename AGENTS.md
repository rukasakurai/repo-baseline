# AGENTS.md

## Repository Purpose

This is a **baseline template repository** designed to provide a minimal, reusable starting point for new GitHub repositories. It establishes foundational structure and conventions without making premature decisions about specific technologies, licenses, or deployment patterns.

## Collaboration & Decision-Making Style

### For Human Contributors
- **Start lean**: Prefer minimal, focused implementations over comprehensive solutions
- **Defer specifics**: Avoid hardcoding environment-specific values (tenant IDs, subscription IDs, secrets)
- **Template-first thinking**: Any addition should be generally useful across multiple derived repositories

### For Copilot
- **Minimize changes**: Make the smallest possible modifications to achieve the goal
- **Public-safe by default**: Never suggest or add sensitive information (secrets, Azure IDs, API keys)
- **Respect deferred decisions**: Do not add licenses, specific deployment configurations, or technology-specific scaffolding unless explicitly requested
- **Validate assumptions**: When working on derived repositories, check if baseline conventions are still relevant before applying them

## Constraints & Assumptions

### What This Template Includes
- Human and AI collaboration guidance (this file)
- Issue and PR templates for structured communication
- Manual Azure OIDC validation workflow (uses `workflow_dispatch`, requires external configuration)
- Minimal README with post-creation checklist

### What This Template Excludes
- **No license file**: License choice is intentionally deferred to derived repositories
- **No copilot-instructions.md**: Repository-specific instructions belong in derived repos
- **No automatic CI/CD**: Continuous integration and deployment patterns vary by project
- **No secrets or identifiers**: All Azure/cloud credentials must be configured per repository
- **No technology assumptions**: No language-specific tooling, frameworks, or build systems

### Repository Template Usage
This repository is configured as a GitHub template. When creating a new repository from this template:
1. Decide on and add an appropriate LICENSE file
2. Configure Azure OIDC federated credentials (if using Azure)
3. Update README.md with repository-specific details
4. Remove or modify this AGENTS.md file to reflect the new repository's purpose
