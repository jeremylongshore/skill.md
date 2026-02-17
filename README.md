# skill.md

Agent skills for EigenCloud. Drop these into any AI coding agent and give it the ability to deploy, manage, and operate verifiable applications on EigenCompute TEEs.

## What's in here

| Skill | What it does |
|-------|-------------|
| [`ecloud`](./ecloud/) | Full lifecycle management for EigenCompute TEEs — auth, billing/Stripe, deploy, operate, TLS, terminate. Any agent that loads this skill can scaffold a new app, handle subscription payments, deploy to a TEE, and manage it from there. |

## What an agent can do with this

Once the ecloud skill is loaded, an agent can:

- **Create and deploy TEEs** from templates (TypeScript, Python, Go, Rust)
- **Handle Stripe billing** — trigger `ecloud billing subscribe`, capture the payment portal URL, and return it to the user
- **Manage running TEEs** — start, stop, upgrade, view logs, check wallet addresses
- **Configure HTTPS/TLS** with automatic Let's Encrypt certificates
- **Terminate TEEs safely** — with fund withdrawal warnings and confirmation flows
- **Write app code** that uses the TEE wallet (`process.env.MNEMONIC` via viem)
- **Troubleshoot** common deployment issues (Docker platform targeting, port binding, auth failures)

The skill defaults to keeping everything private — log visibility, environment variables, and app configuration are all private unless explicitly opted into public.

## Install

Skills follow the [Agent Skills](https://github.com/openai/skills) open standard. A skill is a folder with a `SKILL.md` file, optional `references/` for supporting docs, and optional `scripts/` for tooling. All three major agent platforms support this format.

### Claude Code

Copy the skill folder into your Claude Code skills directory:

```bash
cp -r ecloud/ ~/.claude/skills/ecloud/
```

The skill appears automatically. Invoke with `/ecloud` or let Claude pick it up implicitly when you mention deploying to EigenCompute, TEEs, or ecloud.

**Project-level** (scoped to a repo):
```bash
cp -r ecloud/ .claude/skills/ecloud/
```

### Codex

Copy into any of the Codex skill discovery paths:

```bash
# User-level (available across all projects)
cp -r ecloud/ ~/.agents/skills/ecloud/

# Repo-level (scoped to this project)
cp -r ecloud/ .agents/skills/ecloud/
```

Restart Codex to pick up the skill. Invoke with `$ecloud` or let Codex match it implicitly from your prompt.

### OpenClaw

Copy into your OpenClaw skills directory:

```bash
# User-level (available to all agents)
cp -r ecloud/ ~/.openclaw/skills/ecloud/

# Workspace-level (scoped to current workspace)
cp -r ecloud/ skills/ecloud/
```

OpenClaw loads it on the next session. Workspace skills take precedence over user-level skills.

### Any other agent

The skill is just markdown files. Point your agent at `ecloud/SKILL.md` and it will know what to do. The `references/` folder has the CLI command reference and architecture docs for deeper context.

## Skill structure

```
ecloud/
├── SKILL.md                        # Main skill — workflows, rules, examples
└── references/
    ├── cli-reference.md            # Complete ecloud CLI command reference
    └── architecture.md             # TEE architecture, KMS, security model
```

## Prerequisites

The skill expects the user's machine to have:

- **Docker** installed and logged in (`docker login`)
- **ecloud CLI** installed (`npm install -g @layr-labs/ecloud-cli`)
- **ETH** for deployment gas (Sepolia testnet or mainnet)

The skill walks the agent through checking and setting these up if they're missing.

## Contributing

Add new skills as top-level folders with a `SKILL.md`. Keep the same structure:

```
my-skill/
├── SKILL.md              # Required — name, description, workflows
└── references/           # Optional — supporting docs
```

Frontmatter format:

```yaml
---
name: "skill-name"
description: "One-liner for when the agent should activate this skill."
---
```

The `description` field matters — agents use it to decide whether to load the skill for a given task. Be specific about triggers and scope.
