---
name: nile-markets:query
description: Query Nile Markets protocol data — pool state, positions, oracle prices, account balances via MCP tools
version: 0.3.0
---
<!-- Sync with integrations/openclaw/SKILL.md -->
<!-- Universal skill file: packages/mcp/public/skill.md (served at mcp.nilemarkets.com/skill.md) -->

# Query Nile Markets Protocol Data

> **WARNING: Sepolia Testnet Only** — All data returned by these tools is from the Ethereum Sepolia testnet. Token balances and positions have no real-world value. Do not use this data for financial decisions.

> **Unstable API** — Tool schemas, response formats, and available fields may change without notice during the M2 milestone. Pin to version 0.3.x for stability.

## Available MCP Tools

Use the `nile-markets` MCP server to query live protocol data. All tools return JSON with `protocol` (name + version), `network`, and `data` fields. Tools sourcing data from the subgraph include `_meta.lastIndexedBlock` to detect sync lag.

### Pool State

**Tool**: `get_pool_state`
**Input**: (none)

Returns current liquidity pool state including total assets (USDC, 6 decimals), total shares, computed share price, net exposure, gross notional, utilization (basis points), cumulative fees, and open position count. Combines RPC (real-time vault data) and subgraph (fee totals) sources.

Use this to answer questions about pool health, TVL, utilization, or fee revenue.

### Positions by Account

**Tool**: `get_positions`
**Input**:
- `account` (required): Ethereum address, e.g., `"0x1234..."`
- `status` (optional): `"OPEN"` or `"CLOSED"`
- `first` (optional): Page size, default 25, max 1000
- `skip` (optional): Offset for pagination, default 0

Returns positions for a specific account with side, tenor, notional, entry strike, margin, PnL, and status. Data sourced from subgraph.

### Single Position (Real-Time PnL)

**Tool**: `get_position`
**Input**:
- `id` (required): Position ID as a string, e.g., `"42"`

Returns a single position with real-time unrealized PnL computed via RPC. Use this when you need current mark-to-market data rather than the last-indexed subgraph snapshot.

### Search Positions

**Tool**: `search_positions`
**Input** (all optional):
- `side`: `"LONG"` or `"SHORT"`
- `tenor`: `"1D"`, `"1W"`, or `"1M"`
- `status`: `"OPEN"` or `"CLOSED"`
- `minNotional`: Minimum notional filter (string, e.g., `"1000"`)
- `first`: Page size, default 25, max 1000
- `skip`: Offset for pagination, default 0

Search positions across all accounts with filters. Calling with no filters returns recent positions ordered by open timestamp descending.

### Forward Prices

**Tool**: `get_forward_price`
**Input**:
- `tenor` (optional): `"1D"`, `"1W"`, or `"1M"`. Omit to get all tenors.

Returns current forward prices from the on-chain oracle (RPC). Each price includes the forward rate (18 decimals), round ID, publish timestamp, and validity flag.

### Oracle State

**Tool**: `get_oracle_state`
**Input**: (none)

Returns complete oracle state including spot price and all forward prices, plus staleness and validity information. The `mode` field is a convenience copy of the protocol mode (canonical source: `get_protocol_mode`). Data from RPC.

### Account State

**Tool**: `get_account`
**Input**:
- `address` (required): Ethereum address, e.g., `"0x1234..."`

Returns account margin state: collateral deposited, locked margin, available margin, position count, and total notional. Combines RPC (margin balances) and subgraph (position aggregates).

### Protocol Mode

**Tool**: `get_protocol_mode`
**Input**: (none)

Returns current protocol operating mode from RPC:
- `NORMAL` — All operations available
- `DEGRADED` — Oracle data may be stale; new positions restricted
- `REDUCE_ONLY` — Only position reductions and closes allowed
- `PAUSED` — All operations suspended

Always check the mode before advising users on available actions.

### Daily Statistics

**Tool**: `get_daily_stats`
**Input**:
- `days` (optional): Number of days, default 7, max 90

Returns daily protocol statistics from subgraph: volume, fee revenue, position opens/closes, and pool deposits/withdrawals per day.

### Pool Transactions

**Tool**: `get_pool_transactions`
**Input**:
- `first` (optional): Page size, default 25, max 1000
- `skip` (optional): Offset for pagination, default 0

Returns historical pool events (deposits and withdrawals) from subgraph. Use for analyzing LP activity.

### Fee Events

**Tool**: `get_fee_events`
**Input**:
- `first` (optional): Page size, default 25, max 1000
- `skip` (optional): Offset for pagination, default 0

Returns fee event breakdown from subgraph. Each event includes fee type (trading, liquidation, termination, oracle), amount, and associated position/transaction.

### Simulate Open Position

**Tool**: `simulate_open_position`
**Input**:
- `side` (required): `"LONG"` or `"SHORT"`
- `tenor` (required): `"1D"`, `"1W"`, or `"1M"`
- `notional` (required): USDC amount as string, e.g., `"1000"`
- `from` (optional): Caller address for full simulation including balance/allowance checks

Returns required margin, trading fee, entry strike, and whether the transaction would succeed. If `from` is omitted, only parameter-level validation is performed (no sender-state checks).

## Usage Guidelines

1. **Check protocol mode first** — Call `get_protocol_mode` before suggesting any trading action. If the protocol is not in `NORMAL` mode, inform the user of restrictions.

2. **Use the right tool for the job**:
   - For a quick overview: `get_pool_state`
   - For a specific user's positions: `get_positions` with their address
   - For real-time PnL on one position: `get_position` with the ID
   - For market-wide position analysis: `search_positions` with filters
   - For current prices: `get_forward_price` or `get_oracle_state`

3. **Pagination** — Tools returning lists support `first`/`skip` pagination. Default page size is 25. Use `skip` to fetch subsequent pages.

4. **Numeric precision** — USDC amounts have 6 decimals, prices have 18 decimals, and utilization/fees are in basis points (1 bp = 0.01%). Convert appropriately when presenting to users.

5. **Subgraph lag** — Tools sourcing subgraph data include `_meta.lastIndexedBlock`. Compare against the latest block to assess freshness. For real-time needs, prefer RPC-backed tools (`get_position`, `get_forward_price`, `get_pool_state`).
