# Nile Markets — Claude Code Plugin

Claude Code plugin for [Nile Markets](https://docs.nilemarkets.com) — on-chain FX markets powered by the Open Nile Protocol, starting with non-deliverable forwards (NDFs).

> **Sepolia Testnet Only** — All data is for development and testing purposes only.

> **Unstable API (v0.3.0)** — Tool schemas and skill instructions may change without notice during the M2 milestone.

## Installation

```shell
# Add the marketplace (one-time)
/plugin marketplace add nil3labs/nile-markets-claude-plugin

# Install the plugin
/plugin install nile-markets@nile-markets-claude-plugin
```

Or test from a local checkout:

```bash
claude --plugin-dir ./integrations/claude-plugin
```

## Skills

| Skill | Purpose |
|-------|---------|
| `/nile-markets:query` | Query pool state, positions, oracle prices, account balances |
| `/nile-markets:integrate` | Generate TypeScript (viem/wagmi) or Rust (alloy-rs) integration code |
| `/nile-markets:explain` | Explain protocol mechanics (margin, liquidation, modes, etc.) |
| `/nile-markets:execute` | Step-by-step write flow orchestration with signing guidance |

## MCP Tools

The plugin connects to the Nile Markets MCP server (`mcp.nilemarkets.com`) via Streamable HTTP. No authentication required.

| Tool | Description |
|------|-------------|
| `get_pool_state` | Pool TVL, shares, exposure, utilization, fees |
| `get_positions` | Positions by account with pagination |
| `get_position` | Single position with real-time PnL |
| `search_positions` | Filter positions by side, tenor, status |
| `get_forward_price` | Current forward prices by tenor |
| `get_oracle_state` | Full oracle state (spot + forwards + validity) |
| `get_account` | Account margin balances and position summary |
| `get_protocol_mode` | Current protocol mode (NORMAL/DEGRADED/etc.) |
| `get_daily_stats` | Daily volume, fees, position stats |
| `get_pool_transactions` | Historical deposits and withdrawals |
| `get_fee_events` | Fee event breakdown for analytics |
| `simulate_open_position` | Simulate opening a position (margin, fees, strike) |

## Documentation

- [Protocol docs](https://docs.nilemarkets.com)
- [MCP server](https://docs.nilemarkets.com/ai-agents/mcp-server)
- [Claude Code plugin](https://docs.nilemarkets.com/ai-agents/claude-code-plugin)
- [Quick start](https://docs.nilemarkets.com/build/quick-start)

## License

[MIT](LICENSE)
