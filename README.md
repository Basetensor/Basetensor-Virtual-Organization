# Basetensor

Decentralized AI/ML inference on **Base (L2, chain ID 8453)** with validation via **Bittensor** and a self-hosted data layer (**TensorDB**, a Ceramic fork). Miners run workloads, publish results to IPFS/TensorDB, and earn dual rewards. Developers get low-cost, programmable inference; consumers get free or affordable endpoints via our partner **BlackRain.ai**.

---

## Highlights

- **On-chain settlement:** EVM smart contracts on **Base** for registration, staking, task lifecycle, and rewards.  
- **Verifiable results:** Oracles & Validators run on **Bittensor TestNet** (see subnet mapping below).  
- **Data layer built for AI:** **TensorDB** stores miner metadata, task descriptors, and output references; large artifacts in **IPFS**.  
- **CLI-first UX:** `tscli` handles wallets, miner registration, task submission, staking, and monitoring.  
- **Consumer access:** BlackRain.ai provides **free/affordable inference** and lets users exchange **AI Inference Credits ↔ AIT\***.  
- **Token (draft):** Ticker **AIT\*** (*subject to change*) for fees, staking/slashing, and governance.

> **Bittensor Integration (TestNet Subnets)**  
> • **All** Oracles & Validators operate on **Bittensor TestNet**.  
> • **Subnet 415 → MainNet (production environment) on TestNet**  
> • **Subnet 416 → DevNet (development/staging) on TestNet**  
> *(We are using TestNet subnets because we cannot register a mainnet subnet right now—and we won’t wait. TestNet still gives us Bittensor’s security/validation mechanics.)*

---

## Table of Contents
- [Architecture](#architecture)
- [Repository Layout](#repository-layout)
- [Quickstart](#quickstart)
- [Configuration](#configuration)
- [CLI (`tscli`)](#cli-tscli)
- [Contracts](#contracts)
- [Networks & Subnets](#networks--subnets)
- [Tokenomics (Draft)](#tokenomics-draft)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Security](#security)
- [License](#license)

---

## Architecture

- **Smart Contracts (Base, L2 8453):**  
  - `NodeRegistry`: miner registration (address, nodeId, public endpoint).  
  - `TaskRegistry`: task assignment & submission; events for observers.  
  - Staking/rewards modules.

- **Bittensor Validation Plane (TestNet):**  
  - **Oracles** provide task parameters/feeds.  
  - **Validators** verify miner outputs and inform rewards.

- **TensorDB (Ceramic fork):**  
  - Mutable, queryable streams for miner metadata, task descriptors, result CIDs.  
  - Optional SQL indexing for analytics (MySQL-compatible).  
  - Self-hosted, with **IPFS** for large artifacts.

- **BlackRain.ai (consumer plane):**  
  - Free/affordable inference endpoints.  
  - **AI Inference Credits ↔ AIT\*** exchange path (guardrails & spread).

---

## Repository Layout

```

/contracts         # Base L2 smart contracts (NodeRegistry, TaskRegistry, staking/rewards)
/tscli             # Command-line client (wallets, register, submit, stake, monitor)
/tensordb          # Ceramic fork + schemas + indexers
/validators        # Bittensor TestNet oracle/validator reference nodes
/docs              # Whitepaper, specs, diagrams, threat model
/examples          # Minimal apps, SDK snippets, integration samples

````

*(Directories may be introduced progressively as components are open-sourced.)*

---

## Quickstart

**Prerequisites**
- Node.js ≥ 18, pnpm/npm
- Python ≥ 3.10 (for validator/oracle utilities)
- Docker (optional, for IPFS & local services)
- Base RPC endpoint (e.g., from your provider)
- Bittensor **TestNet** access (for running oracles/validators)

**1) Clone & install**
```bash
git clone https://github.com/<your-org>/basetensor.git
cd basetensor
pnpm install   # or npm i
````

**2) Bring up local services (optional)**

```bash
# Example: start self-hosted IPFS & TensorDB dev containers
docker compose up -d
```

**3) Configure environment**
Create `.env` at the repo root (see **Configuration** below).

**4) Build & run**

```bash
pnpm -w build

# tscli help
cd tscli
pnpm start -- --help
```

---

## Configuration

Create a `.env` file with the following (example values):

```
# Base / EVM
BASE_RPC_URL=https://base-mainnet.infura.io/v3/<key>
CHAIN_ID=8453
WALLET_PRIVATE_KEY=0x...

# TensorDB / IPFS
TENSORDB_URL=http://localhost:7007
IPFS_API_URL=http://localhost:5001
IPFS_GATEWAY_URL=http://localhost:8080/ipfs/

# Bittensor (we operate entirely on TestNet for now)
BITTENSOR_NETWORK=testnet
# OUR environments on TestNet:
BITTENSOR_SUBNET_MAINNET=415    # OUR Mainnet (production) on TestNet
BITTENSOR_SUBNET_DEVNET=416     # OUR DevNet (staging) on TestNet

# Telemetry / Misc
LOG_LEVEL=info
```

> Keep private keys and secrets secure. For production, use KMS or a secrets vault.

---

## CLI (`tscli`)

Representative commands (final names/flags may change):

```bash
# Create or import an EVM wallet
tscli create-wallet

# Register a miner (public IP/port; one-miner-per-user enforced via DID + contracts)
tscli register-miner --tensornode 10 --ip 203.0.113.10 --port 8080

# Start processing tasks for a tensornode
tscli --start miner --tensornode 10

# Submit a task result (uploads artifact to IPFS and records CID in TensorDB)
tscli submit-task --tensornode 10 --task-id task123 --data ./result.json

# Rotate endpoint
tscli update-miner --ip 203.0.113.11 --port 8081

# Stake / claim / query
tscli stake --amount 1000
tscli claim-rewards
tscli query-stake
```

---

## Contracts

* Network: **Base** (L2, chain ID 8453)
* Addresses: *TBD — will be published in `/contracts/README.md` and release notes*
* Events: `MinerRegistered`, `TaskSubmitted`, `StakeUpdated`, `RewardsClaimed`, …

**Audits:** Prior to production enablement, external audits will be performed. Do not use unaudited code in production.

---

## Networks & Subnets

| Plane | Network     | Subnet/ID | Purpose                                             |
| ----: | ----------- | --------- | --------------------------------------------------- |
|   EVM | Base L2     | 8453      | Settlement: registry, staking, rewards              |
|   TAO | **TestNet** | **415**   | **OUR Mainnet (production environment) on TestNet** |
|   TAO | **TestNet** | **416**   | **OUR DevNet (development/staging) on TestNet**     |

> **Why TestNet now?** Bittensor mainnet subnet registration isn’t available to us yet, and we will not wait. Running on **TestNet** allows us to ship and still leverage Bittensor’s security/validation mechanics. We will migrate/mirror as mainnet options open up.

---

## Tokenomics (Draft)

* **Ticker:** **AIT\*** (*subject to change*).
* **Utility:** Fees (task submission, priority routing, enhanced indexing), staking/slashing for QoS/security, future governance.
* **Emissions:** Decaying rewards to bootstrap supply + validation, with small tail emission.
* **Credits bridge:** **BlackRain.ai** users can exchange **AI Inference Credits ↔ AIT\*** within guardrails.

*Complete draft (allocations, emissions, sinks/treasury) is in `/docs/whitepaper`.*

---

## Roadmap

1. **Public Testnet (Base + Bittensor TestNet):** contracts + TensorDB + `tscli`, miner registration, basic rewards.
2. **Validation Plane:** TestNet oracles/validators; dashboards & metrics; audits.
3. **Consumer Endpoints:** BlackRain.ai free/affordable tiers; **AI Inference Credits ↔ AIT\*** conversion.
4. **Evolution Path:** Maintain **415/416** on TestNet; adopt/mirror mainnet subnet(s) when registration is feasible.
5. **AI-Optimized TensorDB:** indexing for models/datasets/embeddings; confidentiality (TEEs/ZK) pilots.

---

## Contributing

We welcome issues, PRs, and discussions. Please:

* Open an issue describing the change or bug.
* Keep PRs focused and covered with tests where applicable.
* Follow code style and sign your commits if required.

---

## Security

If you discover a vulnerability, **please email** [security@basetensor.com](mailto:security@basetensor.com) with details and reproduction steps.
We’ll acknowledge receipt within 72 hours and coordinate a fix and disclosure window.

---

## License

Copyright © Basetensor contributors.
See [`LICENSE`](./LICENSE) for details. (License will be finalized prior to production.)

---

### Links

* Whitepaper: see `/docs/whitepaper`
* Getting started examples: `/examples`
* Questions: open a GitHub Discussion or email [hello@basetensor.com](mailto:hello@basetensor.com)

> *AIT\**: ticker and parameters are drafts and **subject to change** pending simulations, audits, and community feedback.


