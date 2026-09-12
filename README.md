# StellarShip

[![CI](https://img.shields.io/github/actions/workflow/status/Dione-b/caatinga/ci.yml?branch=main&label=CI&logo=github)](https://github.com/Dione-b/caatinga/actions)
[![npm](https://img.shields.io/npm/v/@caatinga/cli?label=%40stellarship%2Fcli&logo=npm)](https://www.npmjs.com/package/@caatinga/cli?activeTab=versions)

**Ship Soroban smart contracts with confidence.**

StellarShip is a deployment orchestration tool for the Stellar network. It handles building, deploying, and wiring multiple Soroban contracts — and keeps a tamper-evident record of every deployment right inside your Git repository.

- 🚀 Deploy one or many Soroban contracts with a single command
- 🗂️ Track every deployment in Git — no hosted registry needed
- 🔗 Auto-generate browser-ready TypeScript bindings on deploy
- 🔒 Stable `v1.0` contract on npm major `3.x` — pin an exact version for reproducible installs

```bash
npm install -g @stellarship/cli
npx ship init my-dapp
```

> See [CHANGELOG](./packages/cli/CHANGELOG.md) and [Public API](./docs/public-api.md) for the full stability contract.

---

## Quick Start

```bash
# 1. Install globally
npm install -g @stellarship/cli

# 2. Verify your toolchain
ship doctor --network testnet --source alice

# 3. Scaffold, build, and deploy
ship init my-dapp && cd my-dapp && npm install
ship build counter
ship deploy counter --network testnet --source alice
```

After `deploy`, your contract ID is written to `stellarship.artifacts.json` and TypeScript bindings are generated automatically. Pass `--no-generate` to skip binding generation. Run `ship doctor` at any time to diagnose missing prerequisites.

**Docs:** [Getting started](./docs/getting-started.md) — or follow the full walkthrough: [From Zero to Testnet](./docs/tutorials/from-zero-to-testnet.md).

---

## What You Get After Deploy

### `stellarship.artifacts.json` — committed to Git

Your single source of truth for all contract deployments, keyed per network:

```json
{
  "project": "my-dapp",
  "version": 2,
  "networks": {
    "testnet": {
      "contracts": {
        "counter": {
          "contractId": "CABCD...",
          "wasmHash": "a1b2c3..."
        }
      }
    }
  }
}
```

No cloud registry. No `.env` copy-paste. Just a file in your repo that every teammate and every CI run can rely on.

### Auto-Generated TypeScript Bindings

Call your contracts from any frontend with full type safety:

```typescript
import { stellarshipClient } from "./stellarship";

const count = await stellarshipClient.contract("counter").read<number>("get");
await stellarshipClient.contract("counter").invoke("increment");
```

`@stellarship/client` resolves contract IDs from your artifacts file and wires your wallet adapter automatically. No manual configuration required. See [Client docs](./docs/client.md) and [Wallets](./docs/wallets.md).

---

## How It Works

StellarShip orchestrates the official Stellar toolchain — `build`, `deploy`, and `invoke` delegate to the Stellar CLI under the hood. `ship generate` runs `npx @stellar/stellar-sdk generate`. You keep full access to the raw Stellar stack while StellarShip manages the coordination layer on top.

```
   stellarship.config.ts                stellarship.artifacts.json
   (contracts, networks)                (contractIds + wasmHash per network)
          │                                       ▲          │
          ▼                                       │          ▼
  ┌───────────────────┐   ┌──────────────────┐  ┌──────────────────────────┐
  │  ship build       │ → │  ship deploy     │→ │  bindings auto-generated │
  │  (Stellar CLI)    │   │  (graph-aware)   │  │  + freshness markers     │
  └───────────────────┘   └──────────────────┘  └──────────────────────────┘
                                                            │
                              browser                       ▼
  ┌─────────────────────────────────────────────────────────────────────────┐
  │  @stellarship/client: bindings + artifacts + wallet adapter             │
  └─────────────────────────────────────────────────────────────────────────┘
```

Deployed IDs live in `stellarship.artifacts.json`, committed to Git, keyed per network. No mandatory hosted registry. See [ADR 0002](./docs/adr/0002-local-artifacts-as-source-of-truth.md) for the reasoning behind this design.

---

## Requirements

| Dependency | Minimum | Recommended |
|---|---|---|
| Node.js | 22+ | latest LTS |
| Stellar CLI | 23.0.0 | 27.0.0 |
| Rust | 1.84.0 | latest stable |
| Target | `wasm32v1-none` | — |

You'll also need a funded local Stellar CLI identity (e.g. `alice`).

Run `ship doctor` to check what's missing. Manual setup instructions: [Getting started → Prerequisites](./docs/getting-started.md#prerequisites). See the full [version contract](./docs/stellar-cli-version-contract.md).

---

## Project Layout

A newly scaffolded StellarShip project looks like this:

```
my-dapp/
├── stellarship.config.ts        # contracts, WASM paths, network targets
├── stellarship.artifacts.json   # deployed contract IDs per network (commit this)
├── contracts/                   # your Rust Soroban contracts live here
└── src/                         # frontend / client code from your chosen template
    └── contracts/generated/     # TypeScript bindings (auto-generated on deploy)
```

---

## CLI Commands

`ship` (or `stellarship`) covers the full deployment lifecycle:

| Command | What it does |
|---|---|
| `ship init` | Scaffold a new project from a template |
| `ship build` | Compile contracts to WASM via Stellar CLI |
| `ship deploy` | Deploy contracts and update artifacts |
| `ship upgrade` | Upgrade a deployed contract's WASM |
| `ship rollback` | Roll back to a previous deployment |
| `ship generate` | Regenerate TypeScript bindings |
| `ship invoke` | Call a contract function |
| `ship read` | Read a contract state value |
| `ship inspect` | Inspect a deployed contract |
| `ship estimate` | Estimate fees before deploying |
| `ship status` | Show current deployment state |
| `ship migrate` | Run artifact schema migrations |
| `ship sync-env` | Sync contract IDs into your `.env` file |
| `ship doctor` | Verify all prerequisites are in place |
| `ship smoke` | Run smoke tests against deployed contracts |
| `ship ci` | Run the full CI pipeline locally |
| `ship identity` | Manage Stellar CLI identities |
| `ship version` | Print StellarShip and toolchain versions |

---

## Packages

StellarShip is a pnpm monorepo with three published packages:

| Package | Role |
|---|---|
| `@stellarship/cli` | The `ship` command — everything you run in the terminal |
| `@stellarship/core` | Config loading, shell orchestration, Stellar CLI adapters, error catalog |
| `@stellarship/client` | Browser/Node contract client, wallet adapters, React hooks |

Full export map: [Packages](./docs/packages.md). All public errors use stable `STELLARSHIP_*` codes — see [Errors](./docs/errors.md).

---

## Documentation

- **Docs site:** [dione-b.github.io/caatinga](https://dione-b.github.io/caatinga/)
- [Getting started](./docs/getting-started.md) — install, scaffold, deploy, wire to browser
- [Cheatsheet](./docs/cheatsheet.md) · [CLI reference](./docs/cli.md) · [Troubleshooting](./docs/troubleshooting.md)
- [Client](./docs/client.md) · [Wallets](./docs/wallets.md) · [Errors](./docs/errors.md)
- [Architecture](./docs/architecture.md) · [ADRs](./docs/adr/index.md) · [Public API](./docs/public-api.md)

---

## Contributing

Read [CONTRIBUTING.md](./CONTRIBUTING.md) before opening a PR.

```bash
git clone https://github.com/Dione-b/caatinga.git && cd caatinga
pnpm install && pnpm build && pnpm test
pnpm dev init my-dapp   # run the CLI from source
```

We use [Conventional Commits](https://www.conventionalcommits.org/) — keep commits scoped and imperative. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full PR checklist.

---

## License

[MIT](./LICENSE)
