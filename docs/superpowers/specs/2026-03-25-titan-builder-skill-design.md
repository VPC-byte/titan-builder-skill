# titan-builder-skill + README Polish + MCP Enhancement Design Spec

## Overview

Three deliverables in one spec:

1. **titan-builder-skill** — A Claude Code skill file (`titan-builder.md`) that turns Claude into a Titan Builder MEV development assistant and bundle debugger
2. **README polish** — Marketing-ready READMEs for both `titan-builder-mcp` and `titan-builder-skill` repos, designed for the user to tag @titanbuilder_ on X/Twitter
3. **MCP get_bundle_stats enhancement** — Add AI-ready diagnostic analysis to the `get_bundle_stats` tool output

## 1. titan-builder-skill

### Positioning

AI-powered MEV development assistant and interactive bundle debugger for Titan Builder. Not an API wrapper — a knowledge layer that helps developers write MEV bot code, debug bundle failures, and understand Titan Builder's infrastructure.

### Deliverable

A single Markdown file: `titan-builder.md`

### Installation

```bash
# Project-level
cp titan-builder.md .claude/skills/

# Global
cp titan-builder.md ~/.claude/skills/
```

Invoked as `/titan-builder` in Claude Code.

### File Structure

```markdown
---
name: titan-builder
version: 0.1.0
description: |
  AI-powered MEV development assistant and bundle debugger for Titan Builder.
  Query bundle status, diagnose failures, generate MEV bot code, and get
  best practices for bundle construction and submission.
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
---

# Titan Builder — MEV Development Assistant

## Role Definition
[System prompt defining Claude as a Titan Builder expert]

## Titan Builder API Reference
### Endpoints (regional + stats)
### eth_sendBundle (full params, curl example, response format)
### eth_cancelBundle
### titan_getBundleStats
### eth_sendRawTransaction
### eth_sendBlobs

## Bundle Debugging
### Interactive Diagnostic Flow
[Step-by-step: curl stats endpoint → parse status → analyze → recommend]
### Status Reference (7 statuses with root cause + fix)
### Common Failure Patterns Checklist

## MEV Development Best Practices
### Bundle Construction Patterns (backrun, sandwich, liquidation)
### Gas & Bribe Estimation Strategy
### Refund Configuration & Calculation
### Bundle Replacement Strategy (replacementUuid + seqNumber)
### Sponsored Bundles

## Code Generation Guidelines
[When user asks for code, generate Titan Builder API calls in project language]

## Builder Public Keys
[Full list for block verification]
```

### Interaction Patterns

**Bundle Debugging (core feature):**
```
User: /titan-builder debug bundle 0x164d7d...

Skill flow:
1. curl -X POST stats.titanbuilder.xyz with titan_getBundleStats
2. Parse response: status, builderPayment, error
3. Match status to diagnostic template
4. Return: status explanation + root cause + actionable suggestions
```

**MEV Development:**
```
User: /titan-builder how to build a backrun bundle?

Skill flow:
1. Explain backrun pattern (detect target tx → construct backrun → bundle both)
2. Generate code in project language
3. Include refund config, replacementUuid best practices
```

**API Reference:**
```
User: /titan-builder what params does eth_sendBundle accept?

Skill flow:
1. Return structured parameter list with descriptions
2. Include curl example
```

### allowed-tools Rationale

| Tool | Purpose |
|---|---|
| Bash | Execute curl commands to query titan_getBundleStats |
| Read | Read user's code for context-aware assistance |
| Grep | Search user's codebase for Titan Builder usage patterns |
| Glob | Find relevant files in user's project |

## 2. MCP get_bundle_stats Enhancement

### Current Behavior

Returns raw JSON-RPC response from `stats.titanbuilder.xyz`:
```json
{"status":"SimulationFail","builderPayment":"0","error":"BundleRevert. Reverting Hash: 0x…456"}
```

### Enhanced Behavior

Returns raw JSON + structured analysis in the MCP tool result text:

```
## Raw Response
{
  "status": "SimulationFail",
  "builderPayment": "0",
  "builderPaymentWhenIncluded": "0",
  "error": "BundleRevert. Reverting Hash: 0x…456"
}

## Analysis
Status: SIMULATION_FAIL
Root cause: Transaction 0x…456 reverted during top-of-block simulation.

Suggestions:
- Add 0x…456 to revertingTxHashes if this revert is acceptable
- Verify the transaction succeeds via estimateGas against current state
- Ensure builder payment (post-bundle balance - pre-bundle balance) is > 0
- If builderPayment is 0, increase the priority fee on your transactions
```

### Implementation

Add `fn analyze_bundle_status(response: &serde_json::Value) -> String` in `src/tools/get_bundle_stats.rs`.

Pure Rust string formatting — match on the `status` field and generate the corresponding analysis. No external API calls.

#### Status Analysis Templates

| Status | Root Cause | Suggestions |
|---|---|---|
| Received | Bundle arrived too late for the mempool | Submit earlier in the slot window; check network latency to the RPC endpoint |
| Invalid | Malformed bundle (bad RLP, wrong chainId, mined nonces, etc.) | Verify transactions with estimateGas; check block number > current or set to 0; verify chain ID |
| SimulationFail | Transaction revert or builder payment <= 0 | Check revert hash; add to revertingTxHashes if acceptable; ensure builder payment > 0 |
| SimulationPass | Passed simulation but arrived too late for inclusion | Submit earlier; consider using a regional endpoint closer to you |
| ExcludedFromBlock | Valid but not selected (usually insufficient bribe) | Increase priority fee; 99% of cases are solved by increasing bribe |
| IncludedInBlock | Included in a candidate block but a more valuable block won | Increase bribe; check builderPaymentWhenIncluded for your bundle's value in the losing block |
| Submitted | Bundle was submitted to a relay | Success — monitor the relay for block inclusion |

Additional analysis:
- If `builderPayment` is "0" and status is not Submitted: flag as likely payment issue
- If `error` field contains "BundleRevert": extract the reverting hash and suggest adding it to revertingTxHashes

### Modified Tool Output

The `get_bundle_stats` tool method changes from:
```rust
Ok(CallToolResult::success(vec![Content::text(pretty_json)]))
```
To:
```rust
let analysis = analyze_bundle_status(&value);
let output = format!("## Raw Response\n{}\n\n{}", pretty_json, analysis);
Ok(CallToolResult::success(vec![Content::text(output)]))
```

## 3. README Polish

### Common Style

- **Language:** English
- **Badges:** CI status, version (crates.io for MCP), license (MIT)
- **Architecture:** ASCII diagram showing data flow
- **Tables:** For parameters, status codes, endpoints
- **Tone:** Professional, technical, concise — like reth/alloy ecosystem projects
- **Visual unity:** Both READMEs share the same structural patterns to feel like a suite

### titan-builder-mcp README Structure

```
# titan-builder-mcp

[badges: CI | crates.io | license]

One-line tagline: Rust MCP server for Titan Builder

## Highlights
- 5 MCP tools mapping to Titan Builder API
- Enhanced Bundle Tracing with AI-ready diagnostics
- Single binary, zero runtime dependencies
- Regional endpoint support

## Architecture
[ASCII: AI Agent ←stdio→ MCP Server ←HTTPS→ Titan Builder RPC]

## Quick Start (3 steps: install, configure, use)

## Tools (table: tool name | description | Titan API method)

## Enhanced Bundle Tracing
[Explain the diagnostic analysis feature — this is the selling point]

## Bundle Status Reference (table)

## Configuration (env vars + regional endpoints table)

## Building from Source

## License
```

### titan-builder-skill README Structure

```
# titan-builder-skill

[badges: license]

One-line tagline: Claude Code skill — AI MEV development assistant & bundle debugger for Titan Builder

## Highlights
- Interactive bundle debugging via titan_getBundleStats
- MEV development best practices built-in
- Code generation for Titan Builder API integration
- Complete API reference embedded

## Quick Start (2 steps: copy file, use)

## Usage Examples
- Debug a failed bundle (show interaction)
- Generate backrun bundle code
- Query API reference

## What's Inside
[Brief overview of skill contents]

## Bundle Status Reference (table — same as MCP for consistency)

## Titan Builder Endpoints (table)

## Related
- titan-builder-mcp (link)

## License
```

## Non-Goals

- Skill does not handle private keys or sign transactions
- Skill does not replace the MCP server — they are complementary (skill = knowledge, MCP = execution)
- READMEs do not include GIFs/screenshots in v1 (can add later)
- MCP enhancement does not add new API calls — only analyzes existing response data
