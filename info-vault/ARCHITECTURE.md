# Deployment Architectures — Summary

Very short guidance for choosing where to run PQ verification components.

- On‑prem: full control and HSMs; best for strict data-residency or compliance. Higher ops cost.
- Cloud: elastic compute and managed services (KMS/HSM); good for heavy prover workloads but requires trust in provider.
- Hybrid: keep keys on-prem (HSM), run heavy compute in cloud; use secure private links.

Dev-mode notes (quick):
- Ethereum (Hardhat/Foundry/Ganache): use stubs or off-chain attestors for demos; use Foundry for fast gas testing.
- Cosmos (CosmWasm/wasmd): prefer Rust/WASM for on-chain PQ prototypes; local `wasmd` is convenient for testing.

Tips: use HSMs for private keys, containerize provers, and benchmark prover and gas costs.

