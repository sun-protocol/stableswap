## StableSwap Contracts Repository

This repository contains a set of StableSwap-based AMM contracts, mainly inspired by Curve Finance, for low-slippage swapping and liquidity provision between stable assets.

- **Network**: TRON / EVM-compatible chains (using the `ITRC20` interface)
- **Languages**: Solidity (`0.5.12`), Vyper
- **Core Features**:
  - 3-pool stablecoin AMM: low-slippage swaps between three stable assets
  - 2-pool stablecoin AMM: low-slippage swaps between two stable assets
  - USDD / USDC meta pools and fee converters
  - Admin controls for fee / A parameter management and fee distribution

---

## Project Structure

- **`contracts/3pool`**
  - `StableSwap3Pool.sol`: Core 3-asset stable pool contract, implementing swaps, add/remove liquidity, A-parameter ramping and fee accrual.
  - `3USDLiquidityToken.sol`: LP token implementation for the 3-pool.
  - `SSPLiquidityToken.sol`, `SSPToken.sol`: LP / reward token contracts for other pools or legacy deployments.
  - `ValuesAggregator.sol`, `ValuesAggregator_tiny.sol`, `ValuesAggregator_rewards.sol`: Read‑only helper contracts for aggregating pool and reward data.
  - `utils/ITRC20.sol`, `utils/TransferHelper.sol`, `utils/SafeMath.sol`, `utils/ReentrancyGuard.sol`: TRC20 interface, safe transfer helpers, math library and reentrancy guard.

- **`contracts/usdd`**
  - `StableSwap2Pool.sol`: Core 2-asset stable pool contract, API similar to `StableSwap3Pool.sol` with support for swaps, liquidity operations and single-coin withdrawal.
  - `SSP2PoolToken.sol`: LP token for the 2-pool.
  - `ValuesAggregator2Pool.sol`: Aggregator for 2-pool state.
  - `FeeConverter2Pool.sol`: Fee collection and conversion logic for the 2-pool.
  - `Factory.vy`: Vyper factory contract for creating pools or interacting with upstream Curve-style meta pools.

- **`contracts/usdc`**
  - `usdc.vy`: Curve-style USDC meta pool contract integrating with a base pool, handling virtual price, underlying swaps, etc.
  - `usdc3SUN.sol`: LP / reward token contract for the USDC meta pool.
  - `MetapoolFeeConverter.sol`, `MetapoolFeeConverter2.sol`: Fee collection and conversion contracts for meta pools.
  - `depositer.vy`: Helper/adapter contract to deposit assets into the USDC meta pool.

- **`contracts/staker/utils`**
  - `ERC20.sol`, `IERC20.sol`: Standard ERC20 implementation and interface.
  - `SafeMath.sol`, `SafeERC20.sol`, `Address.sol`, `Math.sol`: Utility libraries for arithmetic, safe token operations and address handling.
  - `Ownable.sol`, `Context.sol`, `ReentrancyGuard.sol`: Ownership, execution context and reentrancy protection.

---

## Feature Overview

- **Stablecoin Swaps**
  - Key functions: `get_dy`, `get_dy_underlying`, `exchange`
  - Provide low-slippage swaps between assets inside each pool.

- **Liquidity Management**
  - Add liquidity: `add_liquidity`
  - Proportional withdrawal: `remove_liquidity`
  - Single-coin withdrawal: `remove_liquidity_one_coin`, `calc_withdraw_one_coin`
  - Imbalanced withdrawal: `remove_liquidity_imbalance`

- **Fees and A-Parameter**
  - Fee configuration: `commit_new_fee`, `apply_new_fee`, `revert_new_parameters`
  - A-parameter ramping: `ramp_A`, `stop_ramp_A`
  - Admin fee withdrawal: `admin_balances`, `withdraw_admin_fees`

- **Safety and Controls**
  - Reentrancy protection via `ReentrancyGuard`
  - Emergency kill / unkill: `kill_me`, `unkill_me`

---

## Development and Compilation

This repository currently provides contract source code only. It does not include a full Hardhat / Foundry / Truffle project configuration. You can integrate these contracts into your own tooling or compile them directly from the command line.

### Requirements

- **Solidity**
  - Compiler version: `0.5.12` (exact match recommended to avoid ABI / syntax differences)
  - Recommended flags: `--optimize`

- **Vyper**
  - Version compatible with Curve 2020 contracts (typically `0.2.x` series). Please confirm locally.

### Example: Compile with `solc`

```bash
solc --optimize --bin --abi \
  --evm-version istanbul \
  -o build \
  contracts/3pool/StableSwap3Pool.sol
```

### Example: Compile with `vyper`

```bash
vyper -f bytecode,abi contracts/usdd/Factory.vy
vyper -f bytecode,abi contracts/usdc/usdc.vy
```

You can wire these outputs into your own deployment scripts (e.g. using web3.js, ethers.js, TronWeb, or other tooling).

---

## Deployment and Integration Notes

- **Before Deployment**
  - Make sure all stablecoin contract addresses (TRC20 / ERC20 compatible) are deployed and verified.
  - Prepare deployment parameters for LP tokens (e.g. `3USDLiquidityToken`, `SSPLiquidityToken`, `SSP2PoolToken`, etc.).
  - Decide initial amplification parameter `A`, fee `fee` and admin fee `admin_fee`.

- **Typical Flow (3-pool example)**
  1. Deploy the LP token contract (e.g. `3USDLiquidityToken`).
  2. Deploy `StableSwap3Pool` with:
     - Admin address `owner`
     - Stablecoin addresses array `coins`
     - LP token address
     - Initial `A`, `fee`, `admin_fee`
     - `fee_converter` address (if you want fee conversion)
  3. Have the admin / strategy contracts configure follow-up parameters (fee converter, ownership transfers, etc.).
  4. Integrate frontends or other contracts that call `add_liquidity`, `exchange`, and related functions.

- **FeeConverter Integration**
  - Pools call `FeeConverter.convertFees(...)` and `notify(...)`. Ensure your FeeConverter contracts implement the expected interface and behavior.

---

## Security and Audits

- Several Vyper contracts are based on Curve’s published implementations (see `(c) Curve.Fi, 2020` headers), but this repository as a whole does not yet declare a unified open-source license.
- **Before any production deployment you should:**
  - Perform a full audit for the target chain, asset set, and configuration.
  - Carefully review all admin flows (including `ramp_A`, `kill_me`, `withdraw_admin_fees`, etc.) to ensure they match your governance and risk policies.
  - Re-verify bytecode with the latest compiler versions and best practices for security.

---

## Disclaimer

This repository is provided primarily for research, experimentation and integration reference.  
The authors and contributors are **not** responsible for any financial loss, security incident, or other damage caused by deploying or interacting with these contracts.  
If you intend to use this code in production, you must perform your own audits, testing, monitoring and risk management.
