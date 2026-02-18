---
name: ecloud
description: >
  Deploy and manage verifiable applications on EigenCompute (EigenCloud).
  Use when the user wants to create, deploy, operate, or manage TEE apps
  running in Trusted Execution Environments, handle billing/Stripe subscriptions,
  or build autonomous onchain applications with hardware-isolated wallets.
  Trigger with "deploy TEE", "ecloud", "EigenCompute", "eigencloud",
  "trusted execution", "verifiable app", or "TEE wallet".
allowed-tools: Read, Bash
version: 1.0.0
author: Layr-Labs
license: MIT
---

# EigenCloud (Ecloud) Skill

Deploy containerized applications to Trusted Execution Environments (TEEs) on EigenCompute with persistent wallets, Intel TDX isolation, and cryptographic attestation.

## Overview

EigenCompute provides 26 CLI commands across auth, compute, and billing for managing TEE-deployed apps. Each TEE gets a unique persistent wallet (BIP-39 via KMS), hardware-isolated memory, and onchain verifiability by Docker digest.

**Environments:** `sepolia` (testnet) and `mainnet-alpha` (production, real ETH).

## Prerequisites

1. **ecloud CLI installed**: `npm install -g @layr-labs/ecloud-cli`
2. **Docker running**: `docker login`
3. **Auth configured**: `ecloud auth whoami` — if not set, run `ecloud auth generate --store`
4. **ETH for gas**: Sepolia ETH (testnet) or mainnet ETH

## Instructions

### Deploy a new TEE

1. Verify auth: `ecloud auth whoami`
2. Set environment: `ecloud compute env set sepolia`
3. Create app: `ecloud compute app create --name my-app --language typescript`
4. Configure env: `cp .env.example .env` (MNEMONIC is auto-injected by KMS — never set it manually)
5. Subscribe if needed: `ecloud billing subscribe` (returns Stripe payment URL)
6. Deploy: `ecloud compute app deploy --log-visibility private --instance-type g1-standard-4t --skip-profile`
7. Verify: `ecloud compute app info my-app`

For detailed deploy/manage/billing/TLS workflows, read `{baseDir}/references/workflows.md`.

### Manage a running TEE

| Action | Command |
|--------|---------|
| List apps | `ecloud compute app list` |
| View logs | `ecloud compute app logs <name> --watch` |
| Stop (pause) | `ecloud compute app stop <name>` |
| Start (resume) | `ecloud compute app start <name>` |
| Upgrade code | `ecloud compute app upgrade <name>` |
| Terminate | `ecloud compute app terminate <name>` |

**CRITICAL:** Termination destroys the wallet mnemonic forever. Withdraw ALL funds first.

### Dockerfile requirements

Target `linux/amd64`, run as root, expose port, bind `0.0.0.0`.

For Dockerfile templates, wallet usage, and TLS setup, read `{baseDir}/references/development-guide.md`.

## Output

- Return structured CLI output with app ID, instance IP, and wallet addresses
- Surface billing status and Stripe payment URLs when relevant
- Flag warnings for destructive operations (terminate, fund loss)

## Error Handling

| Problem | Solution |
|---------|----------|
| Docker build fails | Ensure `FROM --platform=linux/amd64` |
| Deploy tx fails | Check ETH balance: `ecloud auth whoami` |
| Image push fails | Run `docker login` |
| App not starting | Check `ecloud compute app logs <name>` |
| No subscription | Run `ecloud billing subscribe` |
| Auth fails | Run `ecloud auth generate --store` |

## Examples

**Quick carrier deploy:**
```bash
ecloud compute app create --name my-bot --language typescript
ecloud compute app deploy --log-visibility private --instance-type g1-standard-4t --skip-profile
ecloud compute app info my-bot
```

**Check and subscribe:**
```bash
ecloud billing status
ecloud billing subscribe    # Returns Stripe URL
```

**Manage lifecycle:**
```bash
ecloud compute app stop my-bot     # Pause (wallet persists)
ecloud compute app start my-bot    # Resume
ecloud compute app upgrade my-bot  # Redeploy with new code
```

## Resources

- CLI reference (all flags): `{baseDir}/references/cli-reference.md`
- TEE architecture and security: `{baseDir}/references/architecture.md`
- Detailed workflows: `{baseDir}/references/workflows.md`
- Development guide (Dockerfile, wallet, TLS): `{baseDir}/references/development-guide.md`
- Verification dashboard: https://verify.eigencloud.xyz/
