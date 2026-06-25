---
name: repo-setup
description: Use when setting up or initializing a repository created from the repo-baseline template (prompts like "set up this repo", "finish the post-creation checklist", or "what should I configure now?"). Guides choosing a license, configuring Azure OIDC, enabling AI agent Azure access, discovering and installing Agent Skills relevant to the repository's technology stack, and replacing the template documentation. Do not use for general feature work in an already-configured repository.
---

# Repo Setup

Guide a contributor (or yourself) through configuring a repository that was created from the **repo-baseline** template.

Work through the steps in order. Treat any authentication, GUI, or portal step as **blocking**: stop and hand back to the user, then continue once they confirm. Keep every change lean and public-safe — never add secrets, tenant/subscription IDs, a license, or technology-specific scaffolding the user has not asked for. See [CONTRIBUTING.md](../../../CONTRIBUTING.md) for the canonical constraints.

## First, determine the context

Before doing anything, work out whether you are in **repo-baseline itself** or in a **repository derived from it**. Signals that you are still in the template: the repository or remote is named `repo-baseline`, the `README.md` is still titled `# repo-baseline`, or `AGENTS.md` still describes "a baseline template repository."

- **If this is the template itself:** do not perform setup — there is nothing here to configure. You are most likely maintaining or testing this skill; act on that intent instead.
- **If this is a derived repository:** proceed with the steps below.

## Setup steps

1. **Choose a license.** The template omits one intentionally. Add a `LICENSE` file only when the user has chosen a license; do not pick one for them.
2. **Configure Azure OIDC** (only if the repo uses Azure). Set up federated credentials and the `AZURE_CLIENT_ID` variable plus `AZURE_TENANT_ID` / `AZURE_SUBSCRIPTION_ID` secrets, following [docs/azure-oidc-setup.md](../../../docs/azure-oidc-setup.md). Then have the user run the **Azure OIDC Connectivity Check** workflow to verify.
3. **Enable AI agent Azure access** (only if using Azure with the Copilot coding agent). Run `azd coding-agent config` to give agents read-time visibility into Azure state. See [docs/azure-coding-agent-guide.md](../../../docs/azure-coding-agent-guide.md).
4. **Decide on the optional Azure scaffolding.** The template ships Azure-oriented assets — the OIDC, E2E, and azd workflows under `.github/workflows/` and the `docs/azure-*.md` guides. These are optional defaults, not requirements. If the project is not using Azure, raise removing them so the repository stays lean and technology-neutral; confirm with the user before deleting anything.
5. **Discover and install relevant Agent Skills.** See [Discover and install Agent Skills](#discover-and-install-agent-skills) below.
6. **Replace the template README.** Swap the generic `README.md` for documentation specific to this repository.
7. **Review AGENTS.md.** Update or trim [AGENTS.md](../../../AGENTS.md) to reflect this repository's actual purpose and conventions.
8. **Retire this skill.** In the same spirit as replacing the README and updating AGENTS.md, this `repo-setup` skill is itself bootstrap scaffolding. Once setup is complete, offer to remove it (the `.github/skills/repo-setup/` directory) and the Post-Creation Checklist from the README; it exists to start a repository, not to live in it.

## Discover and install Agent Skills

Agent Skills are reusable, tool-portable capabilities defined by the [SKILL.md open standard](https://agentskills.io). Install skills that match **this** repository's technology stack and goals — skip anything irrelevant, since unused skills are noise.

### Where to look (in priority order)

1. **The vendor that owns the technology.** First-party skills are the highest-trust option. Look for official skills published by the maintainers of the language, framework, or platform the repo uses — search `github.com/<vendor>/skills` or the vendor's documentation.
2. **GitHub Copilot's default marketplaces** (pre-registered, no setup): [`github/copilot-plugins`](https://github.com/github/copilot-plugins) (official, GitHub-maintained) and [`github/awesome-copilot`](https://github.com/github/awesome-copilot) (large, community-contributed). See the official guidance on [agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) and [plugins](https://docs.github.com/en/copilot/concepts/agents/about-plugins).
3. **Search across GitHub:** `gh skill search <keyword>` finds `SKILL.md` skills in public repositories (GitHub CLI v2.90.0+).

### Install (GitHub Copilot)

```bash
gh skill preview <owner/repo> <skill>   # always inspect first
gh skill install <owner/repo> <skill>   # installs into .github/skills/
```

- Commit installed skills under `.github/skills/` when they should apply to everyone working in the repository.
- Pin a version for stability: `gh skill install <owner/repo> <skill>@<version>`.

### Notes

- **Skills are not verified or signed.** Always inspect a skill's contents (especially any `scripts/`) before installing — they can contain prompt injection or executable code.
- **Installers are interchangeable.** The same skill can be installed with `gh skill install`, with `npx skills add <owner/repo>`, or by placing the skill folder directly in `.github/skills/` (or `.agents/skills/` for cross-tool use). Choose whichever matches the agent in use.
