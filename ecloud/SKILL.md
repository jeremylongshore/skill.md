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

Deploy containerized applications to Trusted Execution Environments (TEEs) on EigenCompute. Each app gets its own persistent wallet, runs in hardware-isolated Intel TDX enclaves, and has cryptographic attestation proving exactly what code is running.

## When to use

- User wants to deploy an app to EigenCompute / a TEE
- User wants to create a new TEE
- User asks about billing, subscriptions, or Stripe payment for EigenCompute
- User wants to manage (start/stop/upgrade/terminate) a deployed TEE
- User wants to check app status, logs, or wallet addresses
- User wants to set up authentication keys for EigenCompute
- User wants to configure TLS/HTTPS for a deployed app
- User is building an autonomous onchain app (trading bot, escrow, verifiable service)

## Core concepts

**TEE (Trusted Execution Environment)**: A Docker container running inside an Intel TDX enclave. Each TEE:
- Gets a unique, persistent wallet (BIP-39 mnemonic from KMS, available as `process.env.MNEMONIC`)
- Has hardware-isolated memory (no one, not even the cloud provider, can read it)
- Is recorded onchain by Docker digest for verifiability
- Can hold funds, sign transactions, and operate autonomously

**KMS**: Key Management System that generates and delivers mnemonics to verified TEEs. Currently single-operator (EigenLabs) in Mainnet Alpha. The mnemonic is deterministic per app ID — same app always gets the same wallet.

**Environments**: `sepolia` (testnet, uses Sepolia ETH) and `mainnet-alpha` (production, uses real ETH).

## Prerequisites

The agent MUST verify these before attempting any ecloud operations:

1. **ecloud CLI installed**: `npm install -g @layr-labs/ecloud-cli`
2. **Docker running and logged in**: `docker login`
3. **Auth key configured**: `ecloud auth whoami` — if not set up, run `ecloud auth generate --store`
4. **ETH for gas**: The auth wallet needs Sepolia ETH (testnet) or mainnet ETH for deployment transactions

Check prerequisites by running `ecloud auth whoami`. If it fails, walk the user through setup.

## Environment

- `ECLOUD_PRIVATE_KEY` — Can be set instead of using keyring auth (optional, prefer keyring)
- `ECLOUD_ENV` — Override environment (`sepolia` or `mainnet-alpha`)

No environment variables are required to be pre-set if the user has already run `ecloud auth generate --store` or `ecloud auth login`.

## Workflow: Deploy a new TEE from scratch

This is the most common flow. Follow these steps in order:

### Step 1 — Check auth

```bash
ecloud auth whoami
```

If not authenticated, guide the user:
```bash
ecloud auth generate --store    # Generate new key and store in OS keyring
# OR
ecloud auth login               # Store existing private key
```

### Step 2 — Set environment (if needed)

Default is mainnet. For development:
```bash
ecloud compute env set sepolia
```

### Step 3 — Create the app from a template

```bash
ecloud compute app create --name <app-name> --language <typescript|python|golang|rust>
cd <app-name>
```

This scaffolds a project with a Dockerfile, .env.example, and template code.

### Step 4 — Configure environment variables

```bash
cp .env.example .env
```

Edit `.env` with the user's secrets. Rules:
- `MNEMONIC` is auto-injected by KMS at runtime — do NOT set a real mnemonic in `.env`
- Variables with `_PUBLIC` suffix are publicly visible (for transparency)
- All other variables are encrypted and only accessible inside the TEE
- Default to keeping variables private (no `_PUBLIC` suffix) unless the user explicitly wants transparency

### Step 5 — Handle billing/subscription

Before deploying, the user needs an active subscription. Check with:
```bash
ecloud billing status
```

If no subscription exists:
```bash
ecloud billing subscribe
```

This opens a **Stripe payment portal** in the browser. The user enters their card details and subscribes. The CLI waits for confirmation.

**Important billing details to tell the user:**
- Metered billing: $0.00177 per vCPU hour
- $100 credit for all new customers
- Up to 10 apps per environment (10 sepolia + 10 mainnet = 20 total)
- Cancel anytime with pro-rata refund: `ecloud billing cancel`

**Returning the Stripe link**: When `ecloud billing subscribe` runs, it outputs a payment portal URL. If the agent is orchestrating this programmatically, capture that URL and present it to the end user so they can complete payment in their browser.

### Step 6 — Deploy

```bash
ecloud compute app deploy
```

The CLI will prompt for:
- **Build method**: Select "Build and deploy from Dockerfile"
- **Log visibility**: Default to `private` unless the user explicitly wants public logs
- **Instance type**: `g1-standard-4t` (4 vCPU, 16GB) or `g1-standard-8t` (8 vCPU, 32GB)
- **App profile**: Name, description, website (optional, shown on dashboard)

To skip prompts and keep things private:
```bash
ecloud compute app deploy \
  --name "My App" \
  --log-visibility private \
  --instance-type g1-standard-4t \
  --skip-profile
```

**What happens during deploy:**
1. Docker image built for `linux/amd64`
2. Image pushed to Docker registry
3. Deployment transaction signed and submitted to Ethereum
4. TEE instance provisioned with Intel TDX
5. KMS generates mnemonic and injects it into the TEE
6. App starts — CLI returns app ID, instance IP, and wallet addresses

### Step 7 — Verify deployment

```bash
ecloud compute app info <app-name>
ecloud compute app logs <app-name> --watch
```

## Workflow: Manage a running TEE

### Check status
```bash
ecloud compute app list                   # All apps in current environment
ecloud compute app info <app-name>        # Detailed info (IP, wallet, status)
ecloud compute app logs <app-name>        # View logs (add --watch for real-time)
```

### Stop / Start (pause without destroying)
```bash
ecloud compute app stop <app-name>        # Pause — wallet persists, costs reduced
ecloud compute app start <app-name>       # Resume — same wallet, same identity
```

### Upgrade (update code or config)
```bash
# Edit code or .env, then:
ecloud compute app upgrade <app-name>
```

This rebuilds and redeploys while preserving the wallet identity.

### Terminate (PERMANENT — irreversible)
```bash
# BEFORE terminating: withdraw ALL funds from the TEE wallet!
ecloud compute app terminate <app-name>
```

**CRITICAL WARNING**: Termination destroys the wallet mnemonic forever. Any funds left in the wallet are lost permanently. Always warn the user and confirm they've withdrawn funds before proceeding.

## Workflow: Billing management

### Check subscription status
```bash
ecloud billing status
```

Returns subscription details and a link to manage payment methods.

### Subscribe (get Stripe payment link)
```bash
ecloud billing subscribe
```

Opens payment portal. The output includes the portal URL — capture and return this to the user if they need to complete payment in a browser.

### Cancel subscription
```bash
ecloud billing cancel
```

Pro-rata refund issued. Running apps are terminated.

## Workflow: Configure HTTPS/TLS

After deploying, to add HTTPS:

```bash
ecloud compute app configure tls
```

This generates a Caddyfile and `.env.example.tls`. Then:

1. Add TLS vars to `.env`:
   ```
   DOMAIN=myapp.example.com
   APP_PORT=3000
   ACME_STAGING=true          # Use Let's Encrypt staging first!
   ```
2. Set DNS A record pointing your domain to the instance IP (from `ecloud compute app info`)
3. Upgrade: `ecloud compute app upgrade <app-name>`
4. Once staging cert works, switch to production:
   ```
   ACME_STAGING=false
   ACME_FORCE_ISSUE=true
   ```
5. Upgrade again to get production cert

**Gotcha**: Let's Encrypt has a rate limit of 5 certs per week per domain. Always test with staging first.

## Dockerfile requirements

Every TEE Dockerfile MUST:
- Target `linux/amd64`: `FROM --platform=linux/amd64 node:18`
- Run as root user (required for TEE)
- Include `EXPOSE <port>` directive
- Bind to `0.0.0.0` (not `localhost` or `127.0.0.1`)

Example:
```dockerfile
FROM --platform=linux/amd64 node:18
USER root
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

## Wallet usage in app code

The TEE mnemonic is available as `process.env.MNEMONIC`. Use viem to derive wallets:

```typescript
import { mnemonicToAccount } from "viem/accounts";

const wallet = mnemonicToAccount(process.env.MNEMONIC!);
console.log("Address:", wallet.address);

// Sign messages
const signature = await wallet.signMessage({ message: "hello" });

// Derive for other chains (Solana, etc.) using standard BIP-32/BIP-44 paths
```

**Security rules for app code:**
- NEVER log `process.env.MNEMONIC`
- NEVER expose the mnemonic in API responses
- NEVER include a real mnemonic in `.env` — it's auto-injected by KMS

## Key gotchas and warnings

1. **Termination is permanent** — wallet mnemonic lost forever, funds unrecoverable
2. **MNEMONIC in .env.example is a placeholder** — KMS injects the real one at runtime
3. **Auth key backup** — if overwritten without backup, you lose access to your deployed apps
4. **Single KMS in alpha** — EigenLabs currently operates the only KMS node
5. **No SLA in Mainnet Alpha** — not recommended for customer funds yet
6. **Costs continue while stopped** — reduced but not zero
7. **Let's Encrypt rate limits** — 5 certs/week per domain, always test with staging
8. **Must bind to 0.0.0.0** — localhost won't work in TEE networking
9. **Default to private** — log visibility, env vars, and app config should be private unless the user explicitly opts into public

## Error handling and troubleshooting

| Problem | Solution |
|---------|----------|
| Docker build fails | Ensure `FROM --platform=linux/amd64` is set |
| Deploy transaction fails | Check ETH balance with `ecloud auth whoami` |
| Image push fails | Run `docker login` |
| App not starting | Check `ecloud compute app logs <name>` — common: wrong port, missing env vars |
| No subscription | Run `ecloud billing subscribe` to get payment link |
| Auth fails | Run `ecloud auth generate --store` or `ecloud auth login` |

## References

- `references/cli-reference.md` — Complete CLI command reference with all flags and options
- `references/architecture.md` — TEE architecture, KMS, security model, and trust guarantees
