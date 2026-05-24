---
name: nile-markets:execute
description: Orchestrate on-chain operations on the Nile Markets protocol — positions, margin, vault, and token operations with signing guidance
version: 0.3.2
metadata:
  filePattern: []
  bashPattern: ["nile\\s+(position|account|pool|token|vault)"]
  priority: 80
---

# Execute On-Chain Operations

Orchestrate on-chain operations on the Nile Markets protocol.

For the canonical, agent-agnostic workflow reference (read operations, write
operations, signing paths, response formats), see:
**https://mcp.nilemarkets.com/workflow.md**

This skill adds Claude Code-specific guidance on top of those workflows.

## Claude Code Setup

### Add Nile Markets MCP Server

```bash
claude mcp add nile-markets --transport http https://mcp.nilemarkets.com/api/mcp
```

### Install Claude Plugin (optional)

```bash
# From monorepo root (provides query, integrate, explain, execute skills)
claude plugin add ./integrations/claude-plugin
```

## Quick Reference

### Path A — Nile CLI + OWS (works today)

If the user has Nile CLI + OWS wallet installed:
```bash
nile config set-wallet <wallet-name>
nile position open --side LONG --tenor 1M --notional 10000
```

### Path B — MCP + OWS MCP (planned, not yet available)

`ows serve --mcp` is not yet available in OWS v1.0.0. When shipped, this will enable
a fully MCP-native flow with no CLI. See [workflow.md](https://mcp.nilemarkets.com/workflow.md#path-b-mcp--ows-mcp-server-planned-not-yet-available).

### Path C — MCP + Any Signing Tool (works today, recommended)

Use Nile MCP write tools to generate unsigned calldata, then sign externally.
This is the recommended path for AI agents today.

For CLI-capable agents, use Foundry `cast`:
```bash
cast send <to> --data <data> --rpc-url <rpc-url> --private-key <key>
```

For programmatic agents, use viem/ethers.js to sign and broadcast.

Complete step-by-step flow in [workflow.md](https://mcp.nilemarkets.com/workflow.md#path-c-mcp--any-signing-tool-works-today-recommended-for-agents).

## Safety Rules

- **NEVER** accept private keys or passphrases in conversation
- **NEVER** store wallet credentials — all key management is the user's responsibility
- **ALWAYS** simulate before generating write calldata for position operations
- **ALWAYS** check balance and allowance before deposit/open operations
- **ALWAYS** inform users that testnet operations have no real-world value
