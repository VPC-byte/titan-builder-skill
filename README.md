# titan-builder-skill

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

**Claude Code skill for [Titan Builder](https://titanbuilder.xyz) — AI-powered MEV development assistant & bundle debugger.**

Turn Claude into a Titan Builder expert. Debug bundle failures interactively, get MEV development best practices, generate integration code, and access the complete Titan Builder API reference — all through natural language.

## Highlights

- **Interactive bundle debugging** — query `titan_getBundleStats` and get diagnostic analysis with actionable suggestions
- **MEV best practices** — bundle construction patterns, gas/bribe strategy, refund configuration, submission timing
- **Code generation** — generate Titan Builder API integration code in your project's language
- **Complete API reference** — all 5 endpoints documented with parameters, examples, and edge cases

## Quick Start

```bash
# Install globally
cp titan-builder.md ~/.claude/skills/

# Or project-level
mkdir -p .claude/skills
cp titan-builder.md .claude/skills/
```

Then in Claude Code:

```
> /titan-builder debug bundle 0x164d7d41f24b...

> /titan-builder why does my bundle keep getting ExcludedFromBlock?

> /titan-builder generate a backrun bundle submission in Rust
```

## Usage Examples

### Debug a Failed Bundle

```
> /titan-builder debug bundle 0x164d7d41f24b7f333af3b4a70b690cf93f636227165ea2b699fbb7eed09c46c7
```

The skill queries `stats.titanbuilder.xyz`, analyzes the response, and provides:
- Status explanation (e.g., SimulationFail)
- Root cause (e.g., transaction 0x…456 reverted)
- Actionable fix (e.g., add hash to `revertingTxHashes`)

### MEV Development Guidance

```
> /titan-builder how should I structure a backrun bundle with refund?
```

Get step-by-step guidance on bundle construction, including `refundPercent` configuration, `replacementUuid` strategy, and submission timing.

### API Reference

```
> /titan-builder what parameters does eth_sendBundle accept?
```

Instant access to complete parameter documentation with types, defaults, and usage notes.

## What's Inside

The skill embeds comprehensive knowledge of:

- **Titan Builder API** — all 5 endpoints with full parameter reference
- **Bundle Tracing** — 7 bundle statuses with diagnostic flowcharts
- **MEV Patterns** — backrun, sandwich, and liquidation bundle construction
- **Operational Knowledge** — gas estimation, bribe strategy, timing windows, regional endpoints
- **Builder Identity** — public keys and coinbase address for block verification

## Bundle Status Reference

| Status | Meaning | Common Fix |
|---|---|---|
| `Received` | Arrived too late for the pool | Submit earlier, use regional endpoint |
| `Invalid` | Malformed bundle | Check RLP, chain ID, nonces, block number |
| `SimulationFail` | Revert or zero builder payment | Fix reverting tx or increase priority fee |
| `SimulationPass` | Passed but too late | Submit earlier in the slot |
| `ExcludedFromBlock` | Insufficient bribe (99% of cases) | Increase priority fee |
| `IncludedInBlock` | Lost to a more valuable block | Increase bribe |
| `Submitted` | Success — submitted to relay | Monitor for on-chain inclusion |

## Titan Builder Endpoints

| Region | URL |
|---|---|
| United States | `https://us.rpc.titanbuilder.xyz` |
| Europe | `https://eu.rpc.titanbuilder.xyz` |
| Asia | `https://ap.rpc.titanbuilder.xyz` |
| Testnet (Hoodi) | `https://rpc-hoodi.titanbuilder.xyz` |
| Bundle Stats | `https://stats.titanbuilder.xyz` |

## Related

- [titan-builder-mcp](https://github.com/VPC-byte/titan-builder-mcp) — Rust MCP server for Titan Builder with Enhanced Bundle Tracing

## License

[MIT](LICENSE)
