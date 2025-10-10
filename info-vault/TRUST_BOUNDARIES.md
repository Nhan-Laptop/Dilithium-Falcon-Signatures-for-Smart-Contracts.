# Trust Boundaries — Implementing Dilithium / Falcon in Smart Contracts

This document explains the trust boundaries, threat model, and recommended deployment options when adding post-quantum (PQ) signature verification (e.g., CRYSTALS-Dilithium or Falcon) to smart contracts. It is written to help you design a secure and cost-effective demonstration or prototype.

## Quick summary
- On-chain verification (verifying PQ signatures directly inside a smart contract) places trust in the contract runtime and gas budget; it moves cryptographic verification into the trust/attack surface of the blockchain VM.
- Off-chain verification with on-chain attestation minimizes gas at the cost of trusting off-chain verifiers or proving correctness via succinct proofs.
- Hybrid patterns (off-chain verification + on-chain merkle/attestation, zk-proof) can reduce costs while keeping verifiability.

## Actors
- User / Wallet: holds private PQ keys (Dilithium/Falcon) and signs messages.
- Smart Contract: stores public keys, enforces access control, optionally verifies signatures.
- Off-chain Verifier / Attestor: performs expensive verification off-chain and submits attestations to chain.
- Sequencer / Rollup: optional actor for L2/Rollup designs.
- Auditors / Watchers: external observers verifying correctness.

## Primary trust boundaries

1. User's Private Key (Client-side)
   - Boundary: between the user's device and the blockchain.
   - Trust requirements: private keys must be generated and stored securely (CSPRNG, hardware wallet if possible). If a user's environment is compromised, signatures can be forged.
   - Notes: PQ keys are larger than ECDSA; transport and storage considerations matter.

2. Public Key Registration (on-chain state)
   - Boundary: between the user's claim about a public key and the contract's stored authoritative copy.
   - Trust requirements: the registration transaction must be authenticated (signed by the account registering it) and the contract must validate format/length.
   - Risks: malicious registration (registering someone else's key) — mitigate by requiring ownership proofs (e.g., signed nonce) at registration.

3. On-Chain Verification (Smart Contract runtime)
   - Boundary: between contract execution and the blockchain VM (EVM/WASM).
   - Trust requirements: the correctness and constant-time properties of the verification implementation. On EVM, gas limits and excessive calldata/storage size are constraints.
   - Risks: incorrect implementation (bugs), side-channels in native or precompiled implementations, gas DoS. Falcon in particular may be side-channel sensitive if floating point is used in native code.

4. Off-Chain Verifier / Attestor
   - Boundary: between off-chain computation and the chain (attestations posted on-chain).
   - Trust requirements: attestors must be honest or proofs must be verifiable. If attestors are centralized, clients must trust them for availability and correctness.
   - Risks: a malicious attestor could submit false attestations; mitigate with multi-party attestations, challenge windows, and fraud proofs.

5. zk / Succinct Proof Verifier
   - Boundary: between an external prover and the on-chain verifier for zk proofs.
   - Trust requirements: soundness of the zk system and correct circuit encoding of the PQ verification.
   - Notes: constructing zk-circuits for PQ verification can be complex; circuits may be large (higher prover cost) but on-chain verification cost can be small.


## Attacker model / threat scenarios
- Key extraction: attacker compromises a user's device or private key store → can produce valid signatures. This is out-of-scope for on-chain defenses except key-revocation mechanisms.
- Registration spoofing: attacker registers a victim's address with a different public key. Mitigate by requiring signed ownership proof during registration (e.g., sign a nonce using the chain account key).
- Replay attacks: attacker replays previously valid attestations/transactions. Mitigate by using nonces, timestamps, chain IDs.
- Malicious attestor: an off-chain verifier posts false attestations; mitigate with multiple attestors, bond/stake, challenge periods, or zk proofs.
- Implementation bugs: incorrect verify implementation allows invalid signatures; mitigate with formal tests, test vectors (NIST PQC test vectors), and careful review.
- Denial-of-service via gas: PQ verify is expensive on EVM; attackers can push expensive calldata or cause repetitive expensive verifications. Mitigate with limits, batching, or off-chain verification.


## Deployment patterns and trust tradeoffs

1. Pure on-chain verification (full trust in contract correctness)
   - Pros: full verifiability without external parties; simple to audit logic in one place.
   - Cons: high gas cost on EVM L1; large calldata and storage; implementation must be constant-time and audited.
   - When to use: demonstration on L2/WASM platforms with cheaper computation (Substrate, CosmWasm), or small-scale L1 demos with stubbed/optimized primitives.

2. Off-chain verification + on-chain attestation
   - Pros: low on-chain gas; heavy lifting off-chain.
   - Cons: requires trust in attestor(s) unless you add fraud-proof or multi-attestor schemes.
   - Patterns: single attestor (trusted), multi-attestor quorum, economic bonds for attestors, challenge windows, or use of commit+reveal.

3. zk-proof assisted verification
   - Pros: low on-chain verifier cost, high trust (if zk system is sound); can provide strong non-repudiation and compact attestations.
   - Cons: complex to implement; prover time can be significant; PQ verification circuit generation is non-trivial.

4. Rollup / Sequencer-based verification
   - Pros: move compute to the rollup; L1 only sees compressed state/attestations.
   - Cons: trust assumptions about sequencer or availability guarantees; you still need fraud proofs/verification for strong security.


## Implementation recommendations

- Start with a prototype that uses a verified reference implementation off-chain and a small on-chain stub that checks a compact attestation (e.g., a hash/merkle root) rather than full PQ verification. This reduces immediate gas pain and lets you experiment.
- Provide robust registration flow:
  - Require the wallet/account to sign a short nonce with its ordinary chain key proving control over the account before accepting a PQ public key registration.
  - Validate public key length and format before storing.
- Use NIST PQC test vectors to validate your implementation (both Dilithium and Falcon) and include edge-case tests (invalid-length, truncated signatures, altered messages).
- If you implement on-chain verification:
  - Prefer running on WASM runtimes (Substrate/CosmWasm) or native precompiles to reduce gas and enable performant integer-only implementations.
  - Avoid floating-point in Falcon implementations; prefer integer/FFT-based verified libraries.
  - Rate-limit or require deposits for expensive verification calls to reduce gas DoS.
- For off-chain attestation:
  - Use multi-party attestation or slashing/bonding to discourage dishonest attestors.
  - Include a challenge window where other watchers can contest incorrect attestations.
- For zk-proof flows:
  - Start with smaller sub-circuits and iterate; benchmark prover time vs verifier cost.
  - Reuse existing zk toolchains where possible (Circom, Halo2, PLONK variants) and check community circuits for PQ primitives.


## Checklist (quick engineering checklist before deploying a demo)
- [ ] Key generation & storage guidance for users (recommend CSPRNG and hardware keys where possible).
- [ ] Registration flow requires ownership proof (signed nonce) and validates key formatting.
- [ ] Test vectors integrated and passing (NIST PQC vectors for Dilithium/Falcon).
- [ ] Gas budget/limits and rate-limiting for verification calls.
- [ ] If using off-chain attestors: multi-attestor/quorum or bond and challenge windows are implemented.
- [ ] Audit / review of any native or precompile code used for verification.
- [ ] Documentation in README describing trust assumptions.


## References & resources
- NIST PQC project and test vectors: https://csrc.nist.gov/projects/post-quantum-cryptography
- PQClean implementations (C reference implementations): https://github.com/PQClean/PQClean
- CRYSTALS-Dilithium specification: https://pq-crystals.org/dilithium/


If you want, I can:
- Add a small `TRUST_BOUNDARIES.md` entry to your README describing the demo trust model (I already created this file).
- Create a minimal Solidity contract skeleton with a `dilithiumVerify` stub and the registration flow (ownership proof) so you can run a demo on a local devnet.
- Generate test vectors and a small off-chain attestation script to demonstrate the hybrid flow.

Which of these should I do next? If you want the Solidity skeleton, tell me whether you prefer a full on-chain stub (verify always false/true) or a realistic placeholder with an external-verifier pattern.
