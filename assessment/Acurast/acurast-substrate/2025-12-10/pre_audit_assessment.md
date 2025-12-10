# Pre-audit Assessment

## 1. Repository Overview
- **Summary**: Acurast is a Substrate-based parachain designed to orchestrate decentralized computation. It acts as a marketplace matching "Job Creators" (consumers) with "Processors" (compute providers). The system supports cross-chain communication via its "Hyperdrive" protocol and integrates with Ink! and Vara/Gear smart contracts.
- **Tech Stack**:
  - **Language**: Rust
  - **Framework**: Substrate (Polkadot SDK v1.18.5)
  - **Smart Contracts**: Ink!, Gear/Vara (Wasm)
  - **Crypto**: Custom integrations for P-256 and P-384 elliptic curves.
- **Key Components**:
  - **`pallets/acurast`**: Core logic for job registration and attestation management.
  - **`pallets/marketplace`**: Economic logic for matching jobs, managing budgets, and distributing rewards.
  - **`pallets/hyperdrive`**: Cross-chain messaging protocol.
  - **`p256-crypto` & `pallets/acurast/p384`**: Custom cryptographic primitives.
  - **`hyperdrive/vara`**: Components for integration with the Vara network.

## 2. Code Quality and Maintainability
- **Strengths**:
  - **Modularity**: The codebase follows standard Substrate patterns, effectively separating concerns into distinct pallets (`acurast`, `marketplace`, `compute`, `hyperdrive`).
  - **Testing**: Core pallets (`acurast`, `marketplace`) have comprehensive unit tests covering happy paths, edge cases (e.g., scheduling overlaps), and economic flows.
  - **Standards**: Uses `rustfmt` and `clippy` (enforced in CI). Storage versions are explicitly managed, facilitating upgrades.
- **Weaknesses**:
  - **Custom Cryptography**: The repository contains in-tree implementations or forks of cryptographic primitives (`pallets/acurast/p384`), which increases the maintenance burden and security risk compared to using standard libraries.
  - **Complexity**: The `marketplace` pallet handles complex state transitions involving matching, scheduling, and payments, making it a critical area for logic errors.
- **Notable Patterns**:
  - **Attestation**: The system relies heavily on validating hardware attestations (X.509 certificate chains) from processors. This logic is central to the security model.

## 3. Obvious Issues and Red Flags
- **Security/Red Flag Patterns**:
  - **Custom Crypto Implementation**: `pallets/acurast/p384` and `p256-crypto` implement or wrap low-level cryptographic operations. Rolling your own crypto or maintaining forks is a major red flag and requires a specialized cryptographic audit.
  - **Manual ASN.1 Parsing**: `pallets/acurast/common/src/attestation.rs` performs manual parsing of X.509 certificates using the `asn1` crate. X.509 parsing is historically prone to vulnerabilities; any error here could allow fake hardware attestations.
  - **Unsafe Code**: `unsafe` blocks are present in `hyperdrive/vara` (e.g., `vara-consumer`, `vara-proxy`). While `static mut` is a common pattern in single-threaded Wasm contracts (like Gear), it bypasses Rust's safety guarantees and requires careful review.
- **Potential Bugs**:
  - **Deprecated Calls**: `finalize_job` and `finalize_jobs` in `pallets/marketplace` are marked deprecated but still accessible. They should be removed or strictly gated to prevent confusion or misuse.
- **Dependency Concerns**:
  - The project relies on specific tags of `polkadot-sdk`. Ensuring these are kept up-to-date with security patches is crucial.

## 4. Documentation and Clarity
- **Existing Documentation**:
  - Root `README.md` provides a good high-level introduction and build instructions.
  - Some pallets have their own `README.md`, but coverage is inconsistent.
  - Rustdoc comments are present on many public interfaces but vary in detail.
- **Gaps**:
  - **Architecture Diagrams**: High-level diagrams explaining the interaction between the Parachain, Hyperdrive, and external networks (Vara, AlephZero) would significantly aid auditors.
  - **Threat Model**: No explicit threat model was found. Understanding the assumed trust model for Processors and the Hyperdrive relays is essential for a proper audit.

## 5. Tests and Tooling
- **Tests**:
  - **Unit Tests**: Strong presence in `src/tests.rs` for main pallets.
  - **Mocking**: Extensive use of `mock.rs` to simulate the runtime environment.
  - **Coverage**: Appears high for business logic, but the cryptographic primitives need their own rigorous test vectors (some are present in `p384`, but more is better).
- **Tooling**:
  - **CI/CD**: GitHub Actions and GitLab CI are configured for testing, linting (`fmt`, `clippy`), and building Docker images.
  - **Benchmarking**: Benchmarking infrastructure is present (`runtime/acurast-mainnet/src/benchmarking.rs`), which is good for weight analysis.

## 6. Audit Readiness Summary
- **Readiness Level**: **Partially Ready**
- **Key Blockers**:
  1.  **Crypto Review**: The custom P-384 implementation and X.509 parsing logic are high-risk. A general audit might not cover these deeply enough.
  2.  **Documentation**: Lack of detailed architectural documentation and a threat model will slow down auditors.
- **Recommended Next Steps (Prioritized)**:
  1.  **High Impact**: Isolate the custom cryptography and attestation logic. Ensure it is covered by standard test vectors (e.g., Wycheproof) where possible. Consider replacing in-tree crypto with standard crates (`sp-core`, `p384` from RustCrypto) if they have matured sufficiently.
  2.  **High Impact**: Document the Attestation flow and Hyperdrive protocol in detail. Explain *why* specific crypto choices were made.
  3.  **Medium Impact**: Review `unsafe` usage in `hyperdrive/vara` to ensure it strictly adheres to Gear/Vara safety guidelines.
  4.  **Medium Impact**: Remove deprecated extrinsics to reduce attack surface.
