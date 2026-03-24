---
name: nile-markets:execute
description: Orchestrate on-chain operations on the Nile Markets protocol — positions, margin, vault, and token operations with step-by-step signing guidance
version: 0.3.0
metadata:
  filePattern: []
  bashPattern: ["nile\\s+(position|account|pool|token|vault)"]
  priority: 80
---

# Execute On-Chain Operations

Guide users through complete on-chain operations on the Nile Markets protocol.
This skill orchestrates MCP tool calls in the correct sequence and provides
signing guidance via OWS-compatible wallets.

## Pre-Flight Checks

Before any write operation:
1. **Check protocol mode** — `get_protocol_mode`. Stop if not NORMAL.
2. **Check token balance** — `get_token_balance` with user's address.
3. **Simulate first** — Always call `simulate_open_position` before `open_position`.

## Position Operations

### Open Position

1. `get_protocol_mode` — verify NORMAL
2. `simulate_open_position` — preview margin, fee, entry strike
3. `get_token_balance` — check USDC balance (testnet: `mint_test_token` if needed)
4. `get_account` — check available margin
5. If margin insufficient: `check_allowance` → `approve_token` → `deposit_margin`
6. `open_position` — generate calldata
7. Sign and submit (see Signing Paths below)

### Close Position

1. `get_position` — verify position exists and is OPEN
2. `close_position` — generate calldata (early termination)
3. Sign and submit

## Margin Operations

### Deposit Margin

1. `get_token_balance` — check USDC balance
2. `check_allowance` with `spender: "marginAccounts"`
3. If allowance insufficient: `approve_token` with `spender: "marginAccounts"` → sign
4. `deposit_margin` — generate calldata → sign

### Withdraw Margin

1. `get_account` — check available (unlocked) margin
2. `withdraw_margin` — generate calldata → sign

## Vault Operations

### Deposit to Vault (LP)

1. `get_token_balance` — check USDC balance
2. `check_allowance` with `spender: "poolVault"`
3. If allowance insufficient: `approve_token` with `spender: "poolVault"` → sign
4. `vault_deposit` — generate calldata → sign

### Withdraw from Vault

1. `get_pool_state` — check pool share balance
2. `vault_withdraw` — generate calldata → sign

## Token Operations

### Get Test USDC (Testnet Only)

1. `mint_test_token` — generate mint calldata → sign
2. This only works on Sepolia/local testnets. On mainnet, USDC must be acquired externally.

### Approve Token Spending

1. `check_allowance` — check current allowance
2. `approve_token` — generate approve calldata → sign

## Signing Paths

Write tools return unsigned transaction calldata. To execute:

### Path A: Nile CLI + OWS Wallet (CLI users)

Prerequisite: install an OWS-compatible wallet (see https://docs.openwallet.sh/).
Wallet setup (passphrase, key import, policies) is the user's responsibility.

```bash
nile config set-wallet <wallet-name>
nile position open --side LONG --tenor 1M --notional 10000
```

The CLI builds the transaction, calls the OWS signer (separate process), and broadcasts.

### Path B: MCP + OWS MCP Server (AI agents)

Add both servers to your agent:
```bash
claude mcp add nile-markets --transport http https://mcp.nilemarkets.com/api/mcp
claude mcp add ows --transport http http://localhost:<port>/mcp  # ows serve --mcp
```

Agent calls Nile MCP for calldata, then OWS MCP to sign and broadcast.

### Path C: MCP + Any Signing Tool

Use Nile MCP calldata with any wallet: Foundry `cast`, Frame, MetaMask, Rabby,
or any other signing MCP server.

## Safety Rules

- **NEVER** accept private keys or passphrases in conversation
- **NEVER** store wallet credentials — all key management is the user's responsibility
- **ALWAYS** simulate before generating write calldata
- **ALWAYS** check balance and allowance before deposit/open
- **ALWAYS** inform users that testnet operations have no real-world value
