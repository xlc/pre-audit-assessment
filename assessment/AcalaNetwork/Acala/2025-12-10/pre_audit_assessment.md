# Pre-Audit Assessment Report

## 1. Project Overview
Acala is a decentralized finance network and liquidity hub for Polkadot. It includes the Acala (Polkadot), Karura (Kusama), and Mandala (Testnet) runtimes. The codebase is built on Substrate and includes a suite of custom pallets to support DeFi activities such as decentralized exchange (DEX), stablecoin issuance (Honzon), and liquid staking (Homa).

## 2. Scope of Audit
The audit covers the entire repository, with a specific focus on the custom logic implemented in the `modules/` directory and the runtime configurations.

### Key Directories:
- **Runtimes**: `runtime/acala`, `runtime/karura`, `runtime/mandala`
- **Modules (Pallets)**: `modules/*`
- **Primitives**: `primitives/*`
- **EVM Layer**: `modules/evm`, `modules/evm-accounts`, `modules/evm-utility`

## 3. Codebase Statistics
- **Languages**: Primarily Rust, with TypeScript used for integration/EVM tests.
- **Framework**: Substrate (Polkadot SDK).
- **Testing**: 
  - Unit tests within each module (`src/tests.rs`).
  - Integration tests in `runtime/integration-tests`.
  - TypeScript-based EVM tests in `ts-tests`.

## 4. Key Components & Risk Areas

### 4.1. EVM Compatibility (`modules/evm*`)
The EVM implementation is a critical component allowing Ethereum-compatible smart contracts to run on Acala.
- **Risk**: High.
- **Focus**: 
  - Precompiles (`modules/evm/src/precompiles`): Ensure gas costs are accurate and logic is secure.
  - Runner (`modules/evm/src/runner`): Verify state transition logic.
  - Account mapping (`modules/evm-accounts`): Check address mapping security.

### 4.2. DeFi Primitives
The core value proposition of Acala lies in its DeFi protocols.
- **DEX** (`modules/dex`, `modules/aggregated-dex`): Trading logic, liquidity pools.
- **Stablecoin** (`modules/honzon`, `modules/cdp-engine`, `modules/loans`): Collateral management, liquidation logic, interest rate models.
- **Liquid Staking** (`modules/homa`): Staking logic, redemption, and exchange rates.
- **Incentives** (`modules/incentives`): Reward distribution mechanisms.

### 4.3. Cross-Chain Integration (XCM)
- **Modules**: `modules/xcm-interface`, `modules/xnft`.
- **Runtime Config**: `xcm_config.rs` in runtimes.
- **Risk**: Medium-High.
- **Focus**: Asset transfers, remote execution permissions, and barrier configurations.

### 4.4. Oracle & Prices
- **Modules**: `modules/prices`, `modules/dex-oracle`.
- **Risk**: High (Economic security).
- **Focus**: Price feeding mechanisms, fallback logic, and resistance to manipulation.

## 5. Preliminary Observations
- **Structure**: The codebase follows a standard Substrate workspace structure.
- **Dependencies**: Heavy reliance on `orml` (Open Runtime Module Library) for tokens, currencies, and other utilities.
- **Complexity**: The interaction between the native Substrate layer and the EVM layer (especially regarding assets and precompiles) adds significant complexity.

## 6. Recommendations for Audit
1.  **EVM Precompile Security**: Rigorously test custom precompiles for gas metering attacks and logic errors.
2.  **Economic Attack Vectors**: Analyze the DEX and CDP modules for flash loan attacks or price manipulation vulnerabilities.
3.  **Privilege Escalation**: Review `EnsureOrigin` checks in runtime configurations to ensure governance or admin powers are correctly constrained.
4.  **Resource Exhaustion**: Check weights and benchmarking in `benchmarking.rs` files to prevent block stuffing or DoS attacks.
5.  **Cross-Layer Interactions**: Verify that assets bridged between the native layer and EVM layer maintain consistent state and cannot be double-spent or locked indefinitely.
