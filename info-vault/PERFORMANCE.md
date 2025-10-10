# Performance & Latency — Summary

Key metrics: verification time, prover time, throughput, gas/on-chain cost, finality latency.

Demo targets:
- Off-chain Dilithium verify: aim <100ms on modern server.
- zk-proofs: prover seconds→minutes; on-chain verifier cost should be small.
- EVM on-chain verify: expensive — prefer WASM or precompiles for production.

Dev-mode tips: measure gas with Hardhat/Foundry (`anvil`, `forge test --gas-report`) and benchmark WASM verify in `wasmd` for CosmWasm.

Recommendations: benchmark on target runtime, profile memory, batch verifications, and autoscale prover pools.
