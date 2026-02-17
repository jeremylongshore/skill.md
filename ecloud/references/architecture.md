# EigenCompute Architecture & Security

## TEE Architecture

EigenCompute uses Intel TDX (Trust Domain Extensions) via Google Confidential Space to provide hardware-isolated execution environments.

### Deployment Flow

```
Developer                    Ethereum              Coordinator           TEE (Intel TDX)          KMS
   |                            |                      |                      |                    |
   |-- Build Docker image ----->|                      |                      |                    |
   |-- Sign & submit tx ------->|                      |                      |                    |
   |                            |-- Event emitted ---->|                      |                    |
   |                            |                      |-- Provision TEE ---->|                    |
   |                            |                      |                      |-- Request keys --->|
   |                            |                      |                      |<-- Mnemonic -------|
   |                            |                      |                      |   (after attestation)
   |                            |                      |<-- App running ------|                    |
   |<-- App ID, IP, wallet -----|                      |                      |                    |
```

### Components

- **ecloud CLI**: Builds Docker images, signs deployment transactions, pushes to registry
- **Ethereum**: Records app deployment by Docker digest (immutable audit trail)
- **EigenLabs Coordinator**: Listens for onchain events, provisions TEE instances
- **Intel TDX TEE**: Hardware-isolated enclave with encrypted memory
- **KMS**: Verifies TEE attestation, delivers encrypted mnemonic

## Key Management System (KMS)

### How it works

1. Each app ID maps deterministically to a unique BIP-39 mnemonic
2. KMS encrypts the mnemonic so only a verified TEE can decrypt it
3. TEE proves its identity via hardware attestation (proves exact Docker image running)
4. KMS verifies attestation matches onchain-whitelisted Docker digest
5. Only then does KMS release the mnemonic

### Mnemonic properties

- **Deterministic**: Same app ID always produces the same mnemonic
- **Persistent**: Survives app restarts, upgrades, stop/start cycles
- **Multi-chain**: BIP-39 mnemonic derives keys for Ethereum, Solana, any HD-compatible chain
- **Isolated**: Only decryptable inside the specific TEE that passes attestation

### Current state (Mainnet Alpha)

- Single KMS operator: EigenLabs (Google Cloud KMS)
- EigenLabs theoretically has access to KMS keys
- Future: Distributed KMS with BLS12-381 threshold cryptography (no single operator can access full key)

## Security Model

### What the TEE protects against

| Threat | Protection |
|--------|-----------|
| Malicious cloud provider | TEE hardware isolation — encrypted memory |
| Infrastructure compromise | Host OS cannot read TEE memory |
| Man-in-the-middle | Attestation-verified encryption between TEE and KMS |
| Secret exfiltration | Mnemonic bound to verified TEE, encrypted at rest |
| Credential theft | Never in plaintext outside TEE |
| Supply chain (hardware) | Intel TDX attestation verifies hardware |

### What the TEE does NOT protect against

| Threat | Responsibility |
|--------|---------------|
| Vulnerable application code | Developer |
| Secrets logged by app code | Developer (never log MNEMONIC) |
| Compromised dependencies | Developer (audit packages) |
| Side-channel attacks | TEE mitigation is good but not perfect |
| Physical access attacks | Remote protection only |

### Trust assumptions

You are trusting:
- Intel TDX hardware security (open source, audited)
- Google Confidential Space attestation service
- KMS attestation verification process
- EigenLabs as sole KMS operator (in alpha)

You are responsible for:
- Application logic and dependency security
- Secret handling in your code
- Not logging or exposing the mnemonic

## Privacy Model

### Encrypted (private by default)

- `MNEMONIC` — auto-injected, never in .env
- All `.env` variables without `_PUBLIC` suffix
- Application source code
- Runtime data and memory
- Private keys derived from mnemonic

### Public (only if explicitly opted in)

- Variables with `_PUBLIC` suffix
- App metadata (ID, name, deployment status)
- Container image reference and tags
- Network endpoints (IPs, ports)
- Logs (only if `--log-visibility public`)

### Best practices

- Default everything to private
- Use `_PUBLIC` suffix only for config meant to be transparent
- Never log `MNEMONIC` or derived private keys
- Don't expose secrets in API responses
- Set `--log-visibility private` unless user explicitly wants public

## Trust Guarantees

### Available now (Mainnet Alpha)

1. **Hardware-isolated execution** — Intel TDX with encrypted memory
2. **Cryptographic attestation** — Proof of exact Docker image running
3. **Onchain deployment record** — Immutable audit trail by Docker digest

### In development

1. **Verifiable execution** — Proof of correct computation
2. **Forced inclusion** — Proof app will process your request
3. **Liveness guarantees** — Proof app is responsive
4. **Upgrade delays** — Time-locked code changes (like smart contract timelocks)
5. **Distributed KMS** — Threshold cryptography, no single point of trust

## Verification Dashboards

- Mainnet: https://verify.eigencloud.xyz/
- Sepolia: https://verify-sepolia.eigencloud.xyz/

These show deployed apps with their Docker digests, attestation status, and public logs (if enabled).
