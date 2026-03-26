---
name: nile-markets:integrate
description: Generate TypeScript or Rust integration code for the Nile Markets protocol using viem, wagmi, or alloy-rs with the official SDK
version: 0.3.0
---

# Integrate with Nile Markets Protocol

> **WARNING: Sepolia Testnet Only** — All contract addresses and examples target Ethereum Sepolia. Do not deploy integration code to mainnet. Contract addresses may change between deployments.

> **Unstable API** — SDK exports, ABI shapes, and contract interfaces may change without notice during the M2 milestone.

## Overview

Nile Markets is an on-chain FX market powered by the Open Nile Protocol, starting with non-deliverable forwards (NDFs). Integrations interact with on-chain contracts via the official SDKs:

- **TypeScript**: `@nile-markets/sdk` — ABI objects and typed constants for use with viem/wagmi
- **Rust**: `fx-contracts` crate — alloy-rs bindings generated from Foundry artifacts

Full documentation: [https://docs.nilemarkets.com](https://docs.nilemarkets.com)

## TypeScript Integration (viem + SDK)

### Installation

```bash
npm install viem @nile-markets/sdk
```

### Reading Pool State

```typescript
import { createPublicClient, http } from "viem";
import { sepolia } from "viem/chains";
import { poolVaultAbi } from "@nile-markets/sdk";

const client = createPublicClient({
  chain: sepolia,
  transport: http("https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY"),
});

// Read total assets in the vault
const totalAssets = await client.readContract({
  address: "0x...", // PoolVault address from deployment
  abi: poolVaultAbi,
  functionName: "totalAssets",
});

console.log("Total assets (USDC, 6 decimals):", totalAssets.toString());
```

### Reading Forward Prices

```typescript
import { oracleModuleAbi, PAIR_IDS } from "@nile-markets/sdk";

// EUR/USD pair ID
const pairId = PAIR_IDS.EUR_USD;

// Read spot price
const [spotPrice, spotTimestamp, spotValid] = await client.readContract({
  address: "0x...", // OracleModule address
  abi: oracleModuleAbi,
  functionName: "getSpot",
  args: [pairId],
});
```

### Opening a Position

```typescript
import { positionManagerAbi, Side, Tenor } from "@nile-markets/sdk";
import { parseUnits } from "viem";

// Simulate first
const { request } = await client.simulateContract({
  address: "0x...", // PositionManager address
  abi: positionManagerAbi,
  functionName: "openPosition",
  args: [
    PAIR_IDS.EUR_USD,       // pairId
    Side.LONG,              // side (0 = LONG, 1 = SHORT)
    parseUnits("1000", 6),  // notional in USDC (6 decimals)
    Tenor["1W"],            // tenor enum value
    parseUnits("50", 6),    // margin in USDC
  ],
  account: "0xYourAddress...",
});

// Then send via wallet client
const hash = await walletClient.writeContract(request);
```

### Key SDK Exports

| Export | Description |
|--------|------------|
| `positionManagerAbi` | Position open/close/increase operations |
| `poolVaultAbi` | LP deposit/withdraw, vault state reads |
| `oracleModuleAbi` | Spot and forward price reads |
| `configAbi` | Protocol parameter reads |
| `marginAccountsAbi` | Margin deposit/withdraw, balance reads |
| `modeControllerAbi` | Protocol mode reads |
| `settlementEngineAbi` | Settlement and liquidation operations |
| `Side` | `{ LONG: 0, SHORT: 1 }` |
| `Tenor` | `{ "1D": 0, "1W": 1, "1M": 2 }` |
| `PAIR_IDS` | `{ EUR_USD: "0x..." }` |

### Contract Addresses

Contract addresses are deployment-specific. For Sepolia, load from the deployment JSON:

```typescript
import addresses from "@nile-markets/sdk/deployments/sepolia/addresses.json";

const positionManager = addresses.positionManager; // "0x..."
const poolVault = addresses.poolVault;             // "0x..."
```

See deployment addresses at: [https://docs.nilemarkets.com/deploy/deployment-addresses](https://docs.nilemarkets.com/deploy/deployment-addresses)

## Rust Integration (alloy-rs + fx-contracts)

### Cargo.toml

```toml
[dependencies]
fx-contracts = { path = "../../crates/contracts" }
alloy = { version = "1", features = ["full"] }
tokio = { version = "1", features = ["full"] }
```

### Reading Pool State

```rust
use fx_contracts::generated::pool_vault::PoolVault;
use alloy::providers::ProviderBuilder;

let provider = ProviderBuilder::new()
    .on_http("https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY".parse()?);

let vault = PoolVault::new(pool_vault_address, &provider);

let total_assets = vault.totalAssets().call().await?;
let total_supply = vault.totalSupply().call().await?;
```

### Opening a Position

```rust
use fx_contracts::generated::position_manager::PositionManager;
use fx_contracts::{EUR_USD_PAIR_ID, Side, Tenor};
use alloy::primitives::U256;

let pm = PositionManager::new(pm_address, &provider);

let notional = U256::from(1_000_000_000u64); // 1000 USDC (6 decimals)
let margin = U256::from(50_000_000u64);       // 50 USDC

let tx = pm.openPosition(
    EUR_USD_PAIR_ID,
    Side::Long as u8,
    notional,
    Tenor::OneWeek as u8,
    margin,
).send().await?;

let receipt = tx.get_receipt().await?;
```

## MCP Server Integration

For AI agent integrations, connect to the Nile Markets MCP server instead of calling contracts directly:

```json
{
  "mcpServers": {
    "nile-markets": {
      "url": "https://mcp.nilemarkets.com/api/mcp",
      "transport": "streamable-http"
    }
  }
}
```

The MCP server provides 12 read tools and 6 write tools that handle ABI encoding, error decoding, and response formatting. See the `/nile-markets:query` skill for tool details.

## Documentation References

- **Quick Start**: [https://docs.nilemarkets.com/build/quick-start](https://docs.nilemarkets.com/build/quick-start)
- **TypeScript SDK**: [https://docs.nilemarkets.com/build/typescript-sdk](https://docs.nilemarkets.com/build/typescript-sdk)
- **Contract Overview**: [https://docs.nilemarkets.com/build/contract-overview](https://docs.nilemarkets.com/build/contract-overview)
- **Open Position Tutorial**: [https://docs.nilemarkets.com/build/open-position-tutorial](https://docs.nilemarkets.com/build/open-position-tutorial)
- **LP Deposit/Withdraw Tutorial**: [https://docs.nilemarkets.com/build/lp-deposit-withdraw-tutorial](https://docs.nilemarkets.com/build/lp-deposit-withdraw-tutorial)
- **Code Examples**: [https://docs.nilemarkets.com/build/code-examples](https://docs.nilemarkets.com/build/code-examples)
- **Deployment Addresses**: [https://docs.nilemarkets.com/deploy/deployment-addresses](https://docs.nilemarkets.com/deploy/deployment-addresses)
- **Errors Reference**: [https://docs.nilemarkets.com/build/errors-reference](https://docs.nilemarkets.com/build/errors-reference)

## Important Notes

1. **USDC Approval Required** — Before depositing margin or opening positions, the user must approve the relevant contract to spend their USDC. Use the standard ERC-20 `approve` flow.

2. **Protocol Mode** — Always check the protocol mode before sending write transactions. In `DEGRADED`, `REDUCE_ONLY`, or `PAUSED` modes, certain operations will revert.

3. **Numeric Precision** — USDC uses 6 decimals. Prices use 18 decimals. Basis points use integer values (e.g., 500 = 5%). Always use `parseUnits`/`formatUnits` with the correct decimal count.

4. **Error Handling** — Contract reverts use custom errors (e.g., `InsufficientMargin`, `OracleInvalid`). Decode with viem's `decodeErrorResult` using the relevant ABI. See the errors reference for the full list.
