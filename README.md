# zkScatter

**A privacy-preserving settlement protocol with compliant identity gating.** One
ZK stack powers private **OTC trading** (Pro) and **bulk payouts** (Pay).
Settlement happens on-chain via Groth16 proofs over commitment pools, with
relayers coordinating off-chain (matching orders for Pro, batching payouts for
Pay) and relaying claims gaslessly — so identities and claim structure stay hidden
on-chain, while zk-X509 identity gating keeps the protocol regulatory-compliant.

---

## 🚀 Try it — live on Sepolia

Nothing to build or configure. Bring MetaMask on Sepolia with a little test ETH
([faucet](https://sepoliafaucet.com); TON/USDC/USDT from the
[Tokamak faucet](https://docs.tokamak.network/home/service-guide/faucet-testnet)).

| | | |
| --- | --- | --- |
| **[Hub](https://zkscatter-hub.web.app)** | start here | picks the right app for you |
| **[Pay](https://zkscatter-pay.web.app)** | bulk payouts | pay many recipients, amounts hidden |
| **[Pro](https://zkscatter-pro.web.app)** | OTC trading | private limit orders, no front-running |
| **[Relayer console](https://zkscatter-relayer.web.app)** | operators | run a relayer node |
| **[Developer docs](https://zkscatter-docs.web.app)** | developers | guides + SDK API reference |
| **[Admin](https://zkscatter-admin.web.app)** | operators of *this* deployment | governance — gated by the on-chain owner |

Trading and claiming are gated behind [zk-X509](https://zk-x509.web.app) identity
verification. Testnet only, and one relayer (`bot-1`) serves the whole demo — a
walkthrough, not a liveness guarantee.

<details>
<summary>What's actually running</summary>

Contract addresses come from the committed ledger,
[`contracts/deployments/11155111.json`](contracts/deployments/11155111.json).

| | Where |
| --- | --- |
| Frontends | Firebase Hosting — `zkscatter-<app>.web.app` (`scripts/firebase-deploy.sh`) |
| Shared orderbook, settlement verifier, indexers | one GCP e2-micro, behind Caddy → `https://orderbook.zkscatter.tokamon.io` |
| Relayer `bot-1` | same box → `https://relayer.zkscatter.tokamon.io` (also its on-chain `RelayerRegistry` URL) |
| zk-X509 identity | [separate repo](https://github.com/tokamak-network/zk-X509) → `https://zk-x509.web.app` |

The backends must be https: the apps are served over https, and a browser blocks
http calls from an https page. Details in
[operations/deployment.md](docs/operations/deployment.md).

</details>

---

## 📖 Read the series first (Medium)

New here? Read the Medium series for the *why* and the *how*, then dive into the
code below. Each piece stands on its own.

1. **Product Introduction** — the problem → Pay & Pro → what's hidden and what's shown
   · Read in [English](https://medium.com/@zena_tokamak/8c074a5bfeb6) · [한국어](https://medium.com/@zena_tokamak/fc0bd3d6037a)
2. **Economics & Investors** — why build another, how it sustains itself without a token
   · Read in [English](https://medium.com/@zena_tokamak/8682fb64978b) · [한국어](https://medium.com/@zena_tokamak/5acd65c2df63)
3. **Developers** — under the hood: how ZK notes, half-proofs, and relayers fit together
   · Read in [English](https://medium.com/@zena_tokamak/1959a62ae0ff) · [한국어](https://medium.com/@zena_tokamak/56fba35dd63c)
4. **Protocol Internals** (deep-dive) — how commitments, nullifiers, circuits, and settlement fit together
   · Read in [English](https://medium.com/@zena_tokamak/94466fcdc9be) · [한국어](https://medium.com/@zena_tokamak/7fab0f806aa0)

**Companion — zk-X509 (Identity Gate):** Stop Building New Identity Systems: Bridging 4 Billion Existing IDs to Web3
· Read in [English](https://medium.com/@zena_tokamak/68712efaa09e) · [한국어](https://medium.com/@zena_tokamak/727143a942f1)

---

## The apps — why you'd use each

| App | Why you'd use it | Where |
|-----|------------------|-------|
| **Pay** | Send payroll / grants / bonuses to many people in **one signature**, without publishing who got how much. Recipients claim **gaslessly** and can't see each other's amounts. | hosted |
| **Pro** | Place a **private limit order** — **no MEV**, no desk spread, no balance leak. Matched off-chain, settled on-chain, proceeds claimed gaslessly. | hosted |
| **Operators** | **Run a relayer** and earn deterministic on-chain fees settling private order flow. Permissionless bond, no vendor lock-in, can't see order amounts/sides. | hosted |
| **Admin** | Govern the deployment — CA issuance, sanctions, protocol params, treasury. | hosted |

> 📘 **Full user guide → [docs/user-guide.md](docs/user-guide.md)** — what each app is
> for, the benefits, and step-by-step **how to use it**, all in one place.

---

## How it works

```
Frontend (Next.js)  →  ZK Relayer (Node.js)   →  Contracts (Solidity / Foundry)
     ↕                      ↕                            ↕
  MetaMask            Order matching             PrivateSettlement (ZK)
  EdDSA keys          ZK proof generation        CommitmentPool (incremental Merkle tree)
                      Gasless claims             RelayerRegistry · IdentityGate (multi-CA)
```

- **Private settlement** — Groth16 proofs + commitment pools hide who traded and
  the claim structure on-chain.
- **Compliant by design** — zk-X509 identity gating (Dual-CA: User CA + Relayer
  CA) gates participation without doxxing traders.
- **Off-chain matching, gasless claims** — relayers match orders and relay
  claims so users don't pay gas to collect.

<details>
<summary>ZK circuits (Circom)</summary>

| circuit             | constraints | role                                            |
|---------------------|-------------|-------------------------------------------------|
| `authorize.circom`  | ~15K        | Half-proof per-side settlement authorization    |
| `cancel.circom`     | ~8K         | private order cancel                            |
| `claim.circom`      | ~1.5K       | claim with Merkle inclusion proof               |
| `withdraw.circom`   | ~6K         | withdrawal from commitment pool                 |
| `deposit.circom`    | ~4K         | private deposit into commitment pool            |

</details>

---

## Run locally (development)

### Against the live Sepolia deployment

Runs a frontend on your machine but against the same contracts, relayer, and
orderbook everything else uses. Addresses come from the committed ledger, so
there is nothing to configure — MetaMask on Sepolia is enough.

```bash
./scripts/run-scatter-web.sh <app> sepolia   # hub | pay | pro | operators | admin
```

Ports: pay 4001 · pro 4003 · operators 4004 · admin 4005 · hub 4006. Step-by-step
setup and how to file bugs: [operations/sepolia-team-setup.md](docs/operations/sepolia-team-setup.md).

### Quick start (mock mode — no zk-X509)

Fastest local loop; identity verification is bypassed.

```bash
./scripts/dev.sh --mock
```

Starts its own anvil with `MockIdentityRegistry`, deploys contracts, launches the
zk-relayer + frontend. Open http://localhost:3000.

> First run builds the ZK circuit artifacts (Powers-of-Tau, a few minutes); later
> runs reuse the cached `.ptau`. Needs [Foundry](https://book.getfoundry.sh/getting-started/installation),
> Node.js ≥ 20, and [circom](https://docs.circom.io/getting-started/installation/) 2.x
> (`cd circuits && npm install` once). See
> [docs/operations/local-setup.md](docs/operations/local-setup.md).

### Full local stack (with zk-X509)

zkScatter requires a zk-X509 Identity Registry for user verification. For the full
setup with both systems on a shared anvil, see
[docs/operations/local-setup.md](docs/operations/local-setup.md):

```bash
IDENTITY_REGISTRY=0x... RELAYER_IDENTITY_REGISTRY=0x... ./scripts/dev.sh
```

### Run tests

```bash
(cd contracts && forge test)                         # contract tests
(cd zk-relayer && npm test)                          # relayer unit tests
(cd zk-relayer && npx tsx test/e2e-private-flow.ts)  # full E2E (needs ./scripts/dev.sh --mock)
```

---

## Repository structure

```
contracts/    Solidity contracts + Foundry tests (RelayerRegistry, IdentityGate,
              CommitmentPool, PrivateSettlement, IncrementalMerkleTree)
circuits/     Circom ZK circuits + build scripts
apps/         Next.js frontends — pro, pay, operators, admin, hub
zk-relayer/   ZK order matching + gasless claim relay (orderbook, matcher, DB)
scripts/      Dev, deploy, and E2E scripts
docs/         Guides, design docs, operations runbooks
```

---

## Documentation

Two sets, and they don't overlap:

- **[Developer docs](https://zkscatter-docs.web.app)** — concepts, step-by-step
  guides, and the SDK API reference. Written for people building on zkScatter;
  source lives in [`developers/`](developers/).
- **[`docs/`](docs/)** — architecture and decision records, design specs (including
  work that isn't built yet), operations runbooks, security notes. Written for
  people working on zkScatter itself.

The [user guide](docs/user-guide.md) covers what each app is for and how to use it.

