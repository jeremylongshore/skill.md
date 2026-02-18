# EigenCloud Development Guide

## Dockerfile Requirements

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

## Wallet Usage in App Code

The TEE mnemonic is available as `process.env.MNEMONIC`. Use viem to derive wallets:

```typescript
import { mnemonicToAccount } from "viem/accounts";

const wallet = mnemonicToAccount(process.env.MNEMONIC!);
console.log("Address:", wallet.address);

// Sign messages
const signature = await wallet.signMessage({ message: "hello" });

// Derive for other chains (Solana, etc.) using standard BIP-32/BIP-44 paths
```

Security rules:
- NEVER log `process.env.MNEMONIC`
- NEVER expose the mnemonic in API responses
- NEVER include a real mnemonic in `.env` — it is auto-injected by KMS

## Configure HTTPS/TLS

After deploying, add HTTPS:

```bash
ecloud compute app configure tls
```

This generates a Caddyfile and `.env.example.tls`. Then:

1. Add TLS vars to `.env`:
   ```
   DOMAIN=myapp.example.com
   APP_PORT=3000
   ACME_STAGING=true
   ```
2. Set DNS A record pointing your domain to the instance IP (from `ecloud compute app info`)
3. Upgrade: `ecloud compute app upgrade <app-name>`
4. Once staging cert works, switch to production:
   ```
   ACME_STAGING=false
   ACME_FORCE_ISSUE=true
   ```
5. Upgrade again to get production cert

**Note:** Let's Encrypt rate-limits to 5 certs per week per domain. Always test with staging first.

## Environment Variables

- `ECLOUD_PRIVATE_KEY` — Alternative to keyring auth (optional, prefer keyring)
- `ECLOUD_ENV` — Override environment (`sepolia` or `mainnet-alpha`)

No environment variables required if the user has run `ecloud auth generate --store` or `ecloud auth login`.
