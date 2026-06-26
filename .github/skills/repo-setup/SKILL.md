---
name: repo-setup
description: Use when setting up, initializing, or revisiting a repository created from the repo-baseline template — e.g. "set up this repo", "finish the post-creation checklist", "what should I configure now?", or when the user wants to add a license, wire up Azure OIDC, find and install Agent Skills relevant to the repo, or replace the template's starter documentation.
---

# Repo Setup

Establish the repository's identity first; the rest depends on it. Most remaining steps are independent — do the ones that apply, in any order. A few real constraints: settle whether the repo uses Azure before configuring Azure, and wind down the scaffolding only once the rest is done. Treat any authentication, GUI, or portal step as **blocking**: stop and hand back to the user, then continue once they confirm. Keep every change lean and public-safe — never add secrets, tenant/subscription IDs, a license, or technology-specific scaffolding the user has not asked for. See [CONTRIBUTING.md](../../../CONTRIBUTING.md) for the canonical constraints.

## First, determine the context

Before doing anything, work out whether you are in **repo-baseline itself** or in a **repository derived from it**. Signals that you are still in the template: the repository or remote is named `repo-baseline`, the `README.md` is still titled `# repo-baseline`, or `AGENTS.md` still describes "a baseline template repository."

- **If this is the template itself:** do not perform setup — there is nothing here to configure. You are most likely maintaining or testing this skill; act on that intent instead.
- **If this is a derived repository:** proceed below.

## Establish the repository's identity

Do this first — declaring what the repo is drives the license, Azure, and skill decisions that follow, and makes AGENTS.md authoritative for the rest of the work.

- **Rewrite the README's descriptive content.** Replace the template's title and "what this is" framing with this repository's real purpose, stack, and usage. (Leave the Post-Creation Checklist for now — it is removed during wind-down.)
- **Update AGENTS.md.** Trim or rewrite [AGENTS.md](../../../AGENTS.md) to reflect this repository's actual purpose and conventions.
- **Consider the license.** The template omits one intentionally. Discuss the appropriate license and add a `LICENSE` file once the user has chosen one; do not pick one for them.

## Configure for the declared stack

Independent steps — apply those that fit, in any order.

- **Equip agents with relevant skills.** Not a one-time step — most valuable now that the stack is declared, and worth repeating over time as new skills appear. See [Discover and install Agent Skills](#discover-and-install-agent-skills) below.
- **Decide whether the repo uses Azure.** If it does not, raise removing the Azure-oriented scaffolding — the OIDC, E2E, and azd workflows under `.github/workflows/` and the `docs/azure-*.md` guides — so the repository stays lean and technology-neutral; confirm with the user before deleting anything. If it does, continue with the next two items.
- **Configure Azure OIDC** (Azure only). Set up federated credentials and the `AZURE_CLIENT_ID` variable plus `AZURE_TENANT_ID` / `AZURE_SUBSCRIPTION_ID` secrets, following [docs/azure-oidc-setup.md](../../../docs/azure-oidc-setup.md). Then have the user run the **Azure OIDC Connectivity Check** workflow to verify.
- **Enable AI agent Azure access** (Azure only). Run `azd coding-agent config` to give agents read-time visibility into Azure state. See [docs/azure-coding-agent-guide.md](../../../docs/azure-coding-agent-guide.md).

## Wind down the template scaffolding

Do this last: it removes the skill's own entry point and the skill itself.

- **Remove the Post-Creation Checklist** from the README — the leftover template breadcrumb that points at this skill.
- **Retire this skill.** This `repo-setup` skill is itself bootstrap scaffolding. Once setup is complete, offer to remove it (the `.github/skills/repo-setup/` directory); it exists to start a repository, not to live in it. Keep it only if the user wants to revisit skill discovery periodically.

## Discover and install Agent Skills

Install skills that match **this** repository's technology stack and goals — skip anything irrelevant, since unused skills are noise.

### Where to look (in priority order)

1. **The vendor that owns the technology.** First-party skills are the highest-trust option. Look for official skills published by the maintainers of the language, framework, or platform the repo uses — search `github.com/<vendor>/skills` or the vendor's documentation. Also consider your own or your organization's curated skills repository (e.g. [`rukasakurai/agent-skills`](https://github.com/rukasakurai/agent-skills)).
2. **GitHub Copilot's default marketplaces** (pre-registered, no setup): [`github/copilot-plugins`](https://github.com/github/copilot-plugins) (official, GitHub-maintained) and [`github/awesome-copilot`](https://github.com/github/awesome-copilot) (large, community-contributed). See the official guidance on [agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) and [plugins](https://docs.github.com/en/copilot/concepts/agents/about-plugins).
3. **Search across GitHub:** `gh skill search <keyword>` finds `SKILL.md` skills in public repositories (GitHub CLI v2.90.0+).

### Install (GitHub Copilot)

```bash
gh skill preview <owner/repo> <skill>   # always inspect first
gh skill install <owner/repo> <skill>   # installs into .github/skills/
```

- Commit installed skills under `.github/skills/` when they should apply to everyone working in the repository.
- Pin to a reviewed version or commit SHA so later updates can't silently swap in changed content: `gh skill install <owner/repo> <skill>@<version>` (or `--pin`). `gh skill install` records the source repo, ref, and tree SHA in the skill's frontmatter as provenance.

### Security notes

Treat a third-party skill as untrusted supply-chain content: its instructions enter the agent's context and its bundled scripts may be executed.

- **Skills are not verified or signed.** Inspect before installing — not just any `scripts/`, but the instruction body too, which can carry hidden or obfuscated prompt-injection directives (e.g. to exfiltrate secrets or run commands).
- **Do not pre-approve `shell`/`bash`** in a skill's `allowed-tools` unless you have reviewed the skill and its scripts and trust the source — doing so lets a malicious or injected skill run arbitrary commands without confirmation.
- **Use the least privilege the task needs.**
- **Installers are interchangeable.** The same skill can be installed with `gh skill install`, with `npx skills add <owner/repo>`, or by placing the skill folder directly in `.github/skills/` (or `.agents/skills/` for cross-tool use). Choose whichever matches the agent in use.
