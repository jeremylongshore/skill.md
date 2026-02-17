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

Install this skill: `https://github.com/Layr-Labs/skill.md`

Works with Claude Code, Codex, OpenClaw, and any agent that supports the [Agent Skills](https://github.com/openai/skills) open standard.

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
