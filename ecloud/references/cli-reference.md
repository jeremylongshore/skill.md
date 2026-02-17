# Ecloud CLI Reference

Complete command reference for `@layr-labs/ecloud-cli`.

## Installation

```bash
npm install -g @layr-labs/ecloud-cli
```

## Global Options (available on most commands)

| Flag | Env Var | Description |
|------|---------|-------------|
| `--environment <env>` | `ECLOUD_ENV` | `mainnet-alpha` or `sepolia` |
| `--rpc-url <url>` | `ECLOUD_RPC_URL` | Custom RPC URL |
| `--private-key <key>` | `ECLOUD_PRIVATE_KEY` | Private key (prefer keyring instead) |
| `--env-file <path>` | `ECLOUD_ENVFILE_PATH` | Path to .env file (default: `.env`) |
| `--verbose` | — | Enable verbose logging |

---

## ecloud auth

### generate
Generate a new authentication key.
```bash
ecloud auth generate [--store]
```
- `--store` — Save to OS keyring (macOS Keychain, 1Password, Windows Credential Manager, Linux Secret Service)

### login
Store an existing private key in OS keyring.
```bash
ecloud auth login
```

### logout
Remove stored key from keyring.
```bash
ecloud auth logout [--force]
```

### whoami
Show current wallet address and environment.
```bash
ecloud auth whoami
```

### migrate
Migrate auth from legacy `eigenx` CLI to `ecloud` CLI.
```bash
ecloud auth migrate
```

---

## ecloud compute app

### create
Scaffold a new app from a template.
```bash
ecloud compute app create --name <name> --language <lang>
```
| Flag | Description |
|------|-------------|
| `--name <name>` | App directory name |
| `--language <lang>` | `typescript`, `python`, `golang`, `rust` |
| `--template-repo <url>` | Custom template URL |
| `--template-version <tag>` | Template version |

### deploy
Deploy a new app to a TEE.
```bash
ecloud compute app deploy [flags]
```
| Flag | Env Var | Description |
|------|---------|-------------|
| `--name <name>` | `ECLOUD_NAME` | Display name |
| `--dockerfile <path>` | `ECLOUD_DOCKERFILE_PATH` | Dockerfile path (default: `./Dockerfile`) |
| `--image-ref <ref>` | `ECLOUD_IMAGE_REF` | Pre-built Docker image |
| `--log-visibility <v>` | `ECLOUD_LOG_VISIBILITY` | `public`, `private`, or `off` (**default to private**) |
| `--instance-type <type>` | `ECLOUD_INSTANCE_TYPE` | `g1-standard-4t` (4 vCPU/16GB) or `g1-standard-8t` (8 vCPU/32GB) |
| `--resource-usage-monitoring <v>` | `ECLOUD_RESOURCE_USAGE_MONITORING` | `enable` or `disable` |
| `--skip-profile` | — | Skip app profile setup |
| `--website <url>` | — | Website URL (dashboard) |
| `--description <text>` | — | App description (dashboard) |
| `--x-url <url>` | — | X/Twitter profile (dashboard) |
| `--image <path>` | — | Profile image, JPG/PNG max 4MB (dashboard) |
| `--verifiable` | — | Enable verifiable builds |
| `--repo <url>` | — | Git repo for verifiable build |
| `--commit <sha>` | — | Git commit SHA (40 hex chars) for verifiable build |

### upgrade
Update code/config for an existing app. Same flags as deploy plus app identifier.
```bash
ecloud compute app upgrade [<app-id|name>] [flags]
```

### start
Restart a stopped app (wallet persists).
```bash
ecloud compute app start [<app-id|name>]
```

### stop
Pause a running app (wallet persists, costs reduced).
```bash
ecloud compute app stop [<app-id|name>]
```

### terminate
**PERMANENTLY** delete an app. Wallet mnemonic becomes inaccessible.
```bash
ecloud compute app terminate [<app-id|name>] [--force]
```
- `--force` — Skip confirmation prompt

### list
List all apps in current environment.
```bash
ecloud compute app list [--all] [--address-count <n>]
```
- `--all` — Include terminated apps
- `--address-count <n>` — Number of wallet addresses to show (default: 1)

### info
Show detailed app info (wallet, IP, status).
```bash
ecloud compute app info [<app-id|name>] [--watch] [--address-count <n>]
```

### logs
View app logs.
```bash
ecloud compute app logs [<app-id|name>] [--watch]
```

### releases
View build/release history.
```bash
ecloud compute app releases [<app-id|name>] [--json] [--full]
```

### profile set
Update app profile metadata (shown on verification dashboard).
```bash
ecloud compute app profile set [<app-id|name>] [--name <n>] [--website <url>] [--description <text>] [--x-url <url>] [--image <path>]
```

### configure tls
Add HTTPS/TLS configuration (Caddy + Let's Encrypt).
```bash
ecloud compute app configure tls
```

---

## ecloud compute env

### set
Switch deployment environment.
```bash
ecloud compute env set <sepolia|mainnet-alpha>
```

### list / show
```bash
ecloud compute environment list
ecloud compute environment show
```

---

## ecloud billing

### subscribe
Open Stripe payment portal to subscribe.
```bash
ecloud billing subscribe
```
Returns a payment portal URL. Capture and present to the user.

### status
View subscription status and manage payment methods.
```bash
ecloud billing status
```

### cancel
Cancel subscription (pro-rata refund, apps terminated).
```bash
ecloud billing cancel [--force]
```

---

## Instance types

| Type | vCPUs | Memory | Architecture |
|------|:-----:|:------:|-------------|
| `g1-standard-4t` | 4 | 16 GB | Intel TDX |
| `g1-standard-8t` | 8 | 32 GB | Intel TDX |
