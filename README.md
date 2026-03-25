# titan-builder-skill

A [Claude Code](https://claude.ai/claude-code) skill for interacting with [Titan Builder](https://titanbuilder.xyz) — the high-performance Ethereum block builder.

This skill provides Claude Code with deep knowledge of Titan Builder's API and MEV workflows, enabling natural language interaction for bundle management, transaction submission, and bundle debugging.

## Features

- **Bundle Management** — Send, cancel, and trace MEV bundles through natural language
- **Bundle Debugging** — Understand why bundles failed with detailed status explanations
- **Transaction Submission** — Submit raw transactions and blob transactions via Titan Builder's private RPC
- **Refund Calculation** — Calculate and configure bundle refund parameters
- **Regional Endpoint Selection** — Automatically select the best RPC endpoint

## Quick Start

### Installation

Copy the skill file to your Claude Code skills directory:

```bash
# Project-level
mkdir -p .claude/skills
cp titan-builder.md .claude/skills/

# Or global
cp titan-builder.md ~/.claude/skills/
```

### Usage

Once installed, you can interact with Titan Builder using natural language in Claude Code:

```
> /titan-builder send a bundle with these two transactions to block 0x102286B

> /titan-builder check the status of bundle 0x164d7d...

> /titan-builder why did my bundle fail?

> /titan-builder cancel bundle with UUID abcd1234
```

## Skill Capabilities

### Bundle Operations

- **Send Bundle** — Construct and send `eth_sendBundle` requests with full parameter support
- **Cancel Bundle** — Cancel bundles by `replacementUuid`
- **Bundle Stats** — Query `titan_getBundleStats` and interpret results

### Transaction Operations

- **Send Raw Transaction** — Submit signed transactions via `eth_sendRawTransaction`
- **Send Blobs** — Submit blob transaction permutations via `eth_sendBlobs`

### Diagnostics

- **Status Interpretation** — Explain bundle statuses (Received, Invalid, SimulationFail, SimulationPass, ExcludedFromBlock, IncludedInBlock, Submitted)
- **Troubleshooting** — Suggest fixes based on bundle status and error messages
- **Refund Calculation** — Compute expected refund amounts given bundle parameters

## Titan Builder Endpoints

| Region | URL |
|---|---|
| Global (geo-routed) | `https://rpc.titanbuilder.xyz` |
| Europe | `https://eu.rpc.titanbuilder.xyz` |
| United States | `https://us.rpc.titanbuilder.xyz` |
| Asia | `https://ap.rpc.titanbuilder.xyz` |
| Testnet (Hoodi) | `https://rpc-hoodi.titanbuilder.xyz` |
| Bundle Stats | `https://stats.titanbuilder.xyz` |

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

MIT
