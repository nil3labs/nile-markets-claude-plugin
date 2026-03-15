# Nile Markets — Claude Code Plugin

Claude Code plugin for the [Nile Markets Protocol](https://docs.nilemarkets.com), a EUR/USD non-deliverable forward (NDF) protocol on Ethereum Sepolia.

> **Sepolia Testnet Only** — This plugin connects to Sepolia testnet infrastructure. All data is for development and testing purposes only.

> **Unstable API (v0.3.0)** — Tool schemas, response formats, and skill instructions may change without notice during the M2 milestone.

## Installation

This plugin is distributed as part of the Nile Markets monorepo. See [docs.nilemarkets.com/ai-agents/claude-code-plugin](https://docs.nilemarkets.com/ai-agents/claude-code-plugin) for installation.

If you already have the monorepo checked out:

```bash
claude --plugin-dir /path/to/nile-markets-monorepo/integrations/claude-plugin
```

### Verify installation

After launching Claude Code with the plugin, the `nile-markets` MCP server should appear in your available tools. You can verify by asking:

```
What MCP tools are available from nile-markets?
```

## Usage

Launch Claude Code with the plugin:

```bash
claude --plugin-dir ./integrations/claude-plugin
```

On launch, the `nile-markets` MCP server connects automatically. Invoke skills with:

- `/nile-markets:query` — query live protocol data
- `/nile-markets:integrate` — generate integration code
- `/nile-markets:explain` — explain protocol concepts

To switch between local and production MCP endpoints, edit `.mcp.json`:

```json
{
  "mcpServers": {
    "nile-markets": {
      "url": "http://localhost:3100/api/mcp",
      "transport": "streamable-http"
    }
  }
}
```

## Skills

The plugin provides three skills:

### `/nile-markets:query` — Query Protocol Data

Query live protocol data via MCP tools. Covers pool state, positions, oracle prices, account balances, daily statistics, and trade simulation.

**Example prompts:**

```
What is the current pool state on Nile Markets?
```

```
Show me all open positions for account 0x1234...
```

```
What is the current EUR/USD 1W forward price?
```

```
Simulate opening a LONG 1W position with 1000 USDC notional.
```

```
What is the protocol mode? Can I open new positions?
```

```
Show me the daily stats for the last 7 days.
```

### `/nile-markets:integrate` — Generate Integration Code

Generate TypeScript (viem/wagmi) or Rust (alloy-rs) code to interact with Nile Markets contracts using the official SDK.

**Example prompts:**

```
Write TypeScript code to read the current forward prices from Nile Markets.
```

```
Generate a Rust function to open a LONG 1W EUR/USD position.
```

```
How do I deposit USDC into my margin account using viem?
```

```
Show me how to set up an ERC-20 approval for the PositionManager contract.
```

### `/nile-markets:explain` — Explain Protocol Concepts

Get clear explanations of protocol mechanics: margin, liquidation, modes, settlement, forward pricing, pool mechanics, and trading flows.

**Example prompts:**

```
How does liquidation work on Nile Markets?
```

```
Explain the difference between initial margin and maintenance margin.
```

```
What happens when the protocol enters DEGRADED mode?
```

```
How is PnL calculated for a SHORT position?
```

```
What fees are charged when opening a position?
```

```
How does the ERC-4626 vault share price change over time?
```

## MCP Server

The plugin connects to the Nile Markets MCP server for live protocol data:

- **Production**: `https://mcp.nilemarkets.com/api/mcp`
- **Local development**: `http://localhost:3000/api/mcp`

### Transport

Streamable HTTP (MCP spec 2025-03-26). No authentication required for M2 (Sepolia testnet).

### Rate Limits

100 requests per minute per IP. The server returns `429` with `Retry-After` and `X-RateLimit-*` headers when limits are exceeded.

### Local MCP Server (development)

To run the MCP server locally:

```bash
cd packages/mcp
pnpm dev
```

The server starts on `http://localhost:3000`. To use the local server instead of production, update `.mcp.json`:

```json
{
  "mcpServers": {
    "nile-markets": {
      "url": "http://localhost:3000/api/mcp",
      "transport": "streamable-http"
    }
  }
}
```

### Available Tools

| Tool | Description | Source |
|------|-------------|--------|
| `get_pool_state` | Pool TVL, shares, exposure, utilization, fees | RPC + Subgraph |
| `get_positions` | Positions by account with pagination | Subgraph |
| `get_position` | Single position with real-time PnL | RPC |
| `search_positions` | Filter positions by side, tenor, status | Subgraph |
| `get_forward_price` | Current forward prices by tenor | RPC |
| `get_oracle_state` | Full oracle state (spot + forwards + validity) | RPC |
| `get_account` | Account margin balances and position summary | RPC + Subgraph |
| `get_protocol_mode` | Current protocol mode (NORMAL/DEGRADED/etc.) | RPC |
| `get_daily_stats` | Daily volume, fees, position stats | Subgraph |
| `get_pool_transactions` | Historical deposits and withdrawals | Subgraph |
| `get_fee_events` | Fee event breakdown for analytics | Subgraph |
| `simulate_open_position` | Simulate opening a position (margin, fees, strike) | RPC |

## Documentation

- **Protocol docs**: [https://docs.nilemarkets.com](https://docs.nilemarkets.com)
- **Quick start**: [https://docs.nilemarkets.com/build/quick-start](https://docs.nilemarkets.com/build/quick-start)
- **MCP server docs**: [https://docs.nilemarkets.com/ai-agents/mcp-server](https://docs.nilemarkets.com/ai-agents/mcp-server)
- **Claude Code plugin docs**: [https://docs.nilemarkets.com/ai-agents/claude-code-plugin](https://docs.nilemarkets.com/ai-agents/claude-code-plugin)

## Deployment

See [`packages/mcp/README.md`](../../packages/mcp/README.md) for MCP server deployment instructions.

## License

MIT License.
