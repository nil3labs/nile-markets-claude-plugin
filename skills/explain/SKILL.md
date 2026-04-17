---
name: nile-markets:explain
description: Explain Nile Markets protocol concepts — margin, liquidation, modes, settlement, forward pricing, pool mechanics, and trading flows
version: 0.3.1
---

# Explain Nile Markets Protocol Concepts

> **WARNING: Sepolia Testnet Only** — Nile Markets is currently deployed on Ethereum Sepolia only. All examples and parameters reference testnet values. This is not financial advice.

Use this skill when users ask about how the Nile Markets protocol works. Provide clear, accurate explanations grounded in the protocol documentation. Always reference the relevant docs page for users who want deeper detail.

## Core Concepts

### What is Nile Markets?

Nile Markets is an onchain non-deliverable forward (NDF) protocol. It enables traders to take long or short positions on FX rates (currently EUR/USD and USD/JPY) at a future fixing date, settled in USDC on Ethereum. Unlike perpetual futures, positions have a fixed maturity date (tenor) and settle against a published forward price.

- **Docs**: [Introduction](https://docs.nilemarkets.com/nile-markets/introduction)
- **Docs**: [How It Works](https://docs.nilemarkets.com/nile-markets/how-it-works)
- **Docs**: [Forwards vs. Perpetuals](https://docs.nilemarkets.com/nile-markets/forwards-vs-perpetuals)
- **Docs**: [NDF FX Forwards](https://docs.nilemarkets.com/nile-markets/ndf-fx-forwards)

### Sides (Long and Short)

- **LONG**: Profits when EUR/USD rises (EUR strengthens vs USD). PnL = notional * (currentPrice - entryStrike) / PRICE_PRECISION.
- **SHORT**: Profits when EUR/USD falls (EUR weakens vs USD). PnL = notional * (entryStrike - currentPrice) / PRICE_PRECISION.

- **Docs**: [Sides](https://docs.nilemarkets.com/nile-markets/sides)

### Tenors

Tenors define the position's time to maturity:
- **1D** (1 Day) — Fixes the next business day
- **1W** (1 Week) — Fixes 7 calendar days from open
- **1M** (1 Month) — Fixes 30 calendar days from open

Each tenor has its own forward price derived from the spot rate and interest rate differential.

- **Docs**: [Tenors](https://docs.nilemarkets.com/nile-markets/tenors)

### Supported Pairs

M2 supports EUR/USD and USD/JPY. Each pair has a bytes32 pairId (keccak256 of the name) used across all contracts. Spot prices come from Pyth Network; forward prices are published by the protocol's publisher service.

- **Docs**: [Supported Pairs](https://docs.nilemarkets.com/nile-markets/supported-pairs)

## Margin System

### Margin Requirements

Positions require initial margin (IM) to open and must maintain maintenance margin (MM) to stay open:

- **Initial Margin (IM)**: The minimum collateral required to open a position. Calculated as a percentage of notional (e.g., 500 bps = 5% of notional for 1M tenor). IM rates vary by tenor -- longer tenors require more margin.
- **Maintenance Margin (MM)**: The minimum collateral to keep a position open. If equity falls below MM, the position becomes eligible for liquidation. MM is typically lower than IM (e.g., 250 bps = 2.5%).

- **Docs**: [Margin Requirements](https://docs.nilemarkets.com/protocol/margin-requirements)
- **Docs**: [Margin Model](https://docs.nilemarkets.com/protocol/margin-model)

### Margin Accounts

Each trader has a margin account that holds USDC collateral. Collateral is split into:
- **Available margin**: Free collateral that can be withdrawn or used for new positions
- **Locked margin**: Collateral locked against open positions (IM amount at open time)

Users can deposit and withdraw USDC from their margin account at any time, subject to available balance.

### Position & Account Limits

The protocol enforces maximum position size, maximum positions per account, and total open interest caps to manage risk.

- **Docs**: [Position & Account Limits](https://docs.nilemarkets.com/protocol/position-account-limits)

## Liquidation

A position is liquidated when its equity (locked margin + unrealized PnL) falls below the maintenance margin threshold. Liquidation:

1. Closes the position at the current forward price
2. Applies a liquidation penalty fee
3. Returns any remaining margin to the trader
4. Credits the pool with the position's loss + penalty fee

Liquidation is triggered by keeper bots monitoring position health.

- **Docs**: [Liquidation](https://docs.nilemarkets.com/protocol/liquidation)

## Protocol Modes

The protocol operates in one of four modes, escalating restrictions as conditions deteriorate:

| Mode | Description | Allowed Operations |
|------|-------------|--------------------|
| **NORMAL** | Standard operation | All operations |
| **DEGRADED** | Oracle may be stale | Close, reduce, deposit, withdraw only |
| **REDUCE_ONLY** | Risk reduction only | Close, reduce, withdraw margin only |
| **PAUSED** | Emergency halt | No operations |

Mode transitions are governed by the ModeController contract based on oracle staleness thresholds and admin actions.

- **Docs**: [Modes](https://docs.nilemarkets.com/nile-markets/modes)
- **Docs**: [Mode Escalation](https://docs.nilemarkets.com/protocol/mode-escalation)
- **Docs**: [Emergency Procedures](https://docs.nilemarkets.com/protocol/emergency-procedures)

## Settlement

When a position reaches its fixing date:

1. The publisher posts the fixing price for the position's tenor
2. Settlement calculates PnL: the difference between fixing price and entry strike, scaled by notional
3. Positive PnL is paid from the pool to the trader; negative PnL is transferred from the trader's locked margin to the pool
4. Remaining locked margin is returned to the trader's available balance
5. The position is closed with status `CLOSED` and close reason `MATURED`

- **Docs**: [Settlement](https://docs.nilemarkets.com/protocol/settlement)
- **Docs**: [Position Lifecycle](https://docs.nilemarkets.com/protocol/position-lifecycle)

## Forward Pricing

Forward prices are derived from the spot EUR/USD rate and published on-chain by the protocol's publisher service:

- **Spot price**: Sourced from Pyth Network oracle
- **Forward prices**: Computed by the publisher using spot + interest rate differentials, published per-tenor
- **Round system**: Each published price set constitutes a "round" with a unique round ID

The OracleModule contract stores the latest prices and validates staleness. If prices become too stale, the protocol transitions to DEGRADED mode.

- **Docs**: [Oracle Pricing](https://docs.nilemarkets.com/protocol/oracle-pricing)
- **Docs**: [Oracle Safeguards](https://docs.nilemarkets.com/protocol/oracle-safeguards)

## Pool Mechanics

### Liquidity Pool (Vault)

The PoolVault is an ERC-4626 vault where liquidity providers (LPs) deposit USDC and receive shares. The pool acts as the counterparty to all trader positions:

- When traders profit, the pool pays out
- When traders lose, the pool receives their margin
- The pool collects trading fees, liquidation penalties, and oracle fees

Share price = totalAssets / totalSupply. As fees accumulate and trader losses flow in, share price increases for LPs.

- **Docs**: [Vault Mechanics](https://docs.nilemarkets.com/deploy/vault-mechanics)
- **Docs**: [Share Price](https://docs.nilemarkets.com/deploy/share-price)
- **Docs**: [LP Risk & Reward](https://docs.nilemarkets.com/deploy/lp-risk-reward)

### Pool Exposure & Utilization

- **Net exposure**: Sum of all position notionals, with longs positive and shorts negative. Represents the pool's directional risk.
- **Gross notional**: Sum of absolute notional values across all positions.
- **Utilization**: Gross notional as a percentage of total assets. Higher utilization means more of the pool's capital is backing positions.

- **Docs**: [Pool Exposure Caps](https://docs.nilemarkets.com/protocol/pool-exposure-caps)
- **Docs**: [Pool Utilization](https://docs.nilemarkets.com/deploy/pool-utilization)

## Fees

The protocol charges several fee types:

| Fee | When | Calculation |
|-----|------|-------------|
| **Trading fee** | Position open | Basis points on notional |
| **Liquidation penalty** | Position liquidated | Basis points on notional |
| **Early termination fee** | Position closed before maturity | Basis points on notional |
| **Oracle fee** | Position open | Fixed USDC amount per position |

All fees flow to the pool vault, increasing share price for LPs.

- **Docs**: [Fees](https://docs.nilemarkets.com/protocol/fees)
- **Docs**: [Fee Distribution](https://docs.nilemarkets.com/deploy/fee-distribution)
- **Docs**: [Parameter Reference](https://docs.nilemarkets.com/protocol/parameter-reference)

## PnL Calculation

Position PnL depends on side:

- **LONG PnL** = notional * (currentPrice - entryStrike) / PRICE_PRECISION
- **SHORT PnL** = notional * (entryStrike - currentPrice) / PRICE_PRECISION

Unrealized PnL uses the current forward price. Realized PnL uses the fixing price at settlement. All PnL is denominated in USDC.

- **Docs**: [PnL](https://docs.nilemarkets.com/protocol/pnl)

## Trading Flows

### Opening a Position
1. Deposit USDC to margin account (if not already funded)
2. Approve PositionManager to spend USDC (for trading fee + oracle fee)
3. Call `openPosition` with pair, side, notional, tenor, and margin amount
4. Protocol validates mode, margin sufficiency, position limits
5. Position is created with entry strike set to current forward price

### Early Termination
Traders can close positions before maturity by calling `closePosition`. An early termination fee is charged. PnL is computed against the current forward price (not the fixing price).

### Position Reduction
Traders can reduce a position's notional without fully closing it. Proportional margin is released.

- **Docs**: [Trading Flows](https://docs.nilemarkets.com/protocol/trading-flows)
- **Docs**: [Early Termination](https://docs.nilemarkets.com/protocol/early-termination)
- **Docs**: [Position Reduction](https://docs.nilemarkets.com/protocol/position-reduction)

## Architecture

The protocol consists of 9 on-chain contracts:

| Contract | Purpose |
|----------|---------|
| **PositionManager** | Open, close, increase positions |
| **PoolVault** | ERC-4626 LP vault |
| **OracleModule** | Price storage and validation |
| **Config** | Protocol parameters (fees, margins, caps) |
| **MarginAccounts** | Trader collateral management |
| **ModeController** | Protocol mode state machine |
| **SettlementEngine** | Maturity settlement and liquidation |
| **RiskManager** | Exposure cap enforcement |
| **Types** | Shared enums and structs |

- **Docs**: [Architecture](https://docs.nilemarkets.com/nile-markets/architecture)
- **Docs**: [Contract Overview](https://docs.nilemarkets.com/build/contract-overview)
- **Docs**: [System Flows](https://docs.nilemarkets.com/protocol/system-flows)

## Explanation Guidelines

When explaining concepts to users:

1. **Start simple** — Give a one-sentence summary before diving into details.
2. **Use concrete examples** — "If you go LONG 1000 USDC notional at 1.0850 and the fixing price is 1.0900, your PnL is approximately +4.59 USDC."
3. **Reference docs** — Always include the relevant docs URL so users can read the full specification.
4. **Clarify testnet context** — Remind users this is Sepolia testnet; parameters and addresses may differ on mainnet.
5. **Distinguish unrealized vs realized** — Unrealized PnL changes with the forward price; realized PnL is locked in at settlement or early close.
6. **Note M2 limitations** — Only EUR/USD pair, three tenors, no cross-margin, no keeper automation yet.
