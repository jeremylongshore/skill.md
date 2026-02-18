# EigenCloud Workflows

## Deploy a New TEE (Full Detail)

### Step 1 — Check auth

```bash
ecloud auth whoami
```

If not authenticated:
```bash
ecloud auth generate --store    # Generate new key and store in OS keyring
# OR
ecloud auth login               # Store existing private key
```

### Step 2 — Set environment

Default is mainnet. For development:
```bash
ecloud compute env set sepolia
```

### Step 3 — Create the app

```bash
ecloud compute app create --name <app-name> --language <typescript|python|golang|rust>
cd <app-name>
```

Scaffolds a project with Dockerfile, .env.example, and template code.

### Step 4 — Configure environment variables

```bash
cp .env.example .env
```

Rules:
- `MNEMONIC` is auto-injected by KMS at runtime — do NOT set a real mnemonic
- Variables with `_PUBLIC` suffix are publicly visible (for transparency)
- All other variables are encrypted and only accessible inside the TEE
- Default to private (no `_PUBLIC` suffix) unless user explicitly wants transparency

### Step 5 — Handle billing

Check subscription:
```bash
ecloud billing status
```

Subscribe if needed:
```bash
ecloud billing subscribe
```

Opens a Stripe payment portal. Capture the URL and present to the user.

Billing details:
- Metered: $0.00177 per vCPU hour
- $100 credit for new customers
- Up to 10 apps per environment (20 total)
- Cancel anytime: `ecloud billing cancel` (pro-rata refund)

### Step 6 — Deploy

```bash
ecloud compute app deploy
```

CLI prompts for build method, log visibility, instance type, and app profile.

Skip prompts:
```bash
ecloud compute app deploy \
  --name "My App" \
  --log-visibility private \
  --instance-type g1-standard-4t \
  --skip-profile
```

What happens during deploy:
1. Docker image built for `linux/amd64`
2. Image pushed to Docker registry
3. Deployment transaction signed and submitted to Ethereum
4. TEE instance provisioned with Intel TDX
5. KMS generates mnemonic and injects it into the TEE
6. App starts — CLI returns app ID, instance IP, and wallet addresses

### Step 7 — Verify

```bash
ecloud compute app info <app-name>
ecloud compute app logs <app-name> --watch
```

## Manage a Running TEE

### Check status
```bash
ecloud compute app list                   # All apps in current environment
ecloud compute app info <app-name>        # Detailed info (IP, wallet, status)
ecloud compute app logs <app-name>        # View logs (add --watch for real-time)
```

### Stop / Start
```bash
ecloud compute app stop <app-name>        # Pause — wallet persists, costs reduced
ecloud compute app start <app-name>       # Resume — same wallet, same identity
```

### Upgrade
```bash
ecloud compute app upgrade <app-name>
```

Rebuilds and redeploys while preserving wallet identity.

### Terminate (PERMANENT)
```bash
ecloud compute app terminate <app-name>
```

**CRITICAL:** Termination destroys the wallet mnemonic forever. Any funds left are lost permanently. Always confirm the user has withdrawn funds first.

## Billing Management

### Check status
```bash
ecloud billing status
```

### Subscribe
```bash
ecloud billing subscribe
```

Returns a Stripe payment portal URL.

### Cancel
```bash
ecloud billing cancel
```

Pro-rata refund issued. Running apps are terminated.

## Key Gotchas

1. **Termination is permanent** — wallet mnemonic lost forever, funds unrecoverable
2. **MNEMONIC in .env.example is a placeholder** — KMS injects the real one at runtime
3. **Auth key backup** — if overwritten without backup, you lose access to deployed apps
4. **Single KMS in alpha** — EigenLabs currently operates the only KMS node
5. **No SLA in Mainnet Alpha** — not recommended for customer funds yet
6. **Costs continue while stopped** — reduced but not zero
7. **Default to private** — log visibility, env vars, and app config should be private unless explicitly opted into public
