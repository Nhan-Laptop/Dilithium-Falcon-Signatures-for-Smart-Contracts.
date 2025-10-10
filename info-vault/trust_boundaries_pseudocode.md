# Trust Boundaries — Pseudocode & Flows

This file converts the UML trust-boundaries diagram into readable pseudocode and step sequences you can use when implementing and testing.

## 1) Public key registration (recommended pattern)

Inputs: account (EOA), pq_public_key

Client-side:
1. nonce = Contract.requestNonce(account)
2. sig = Account.sign(nonce)          // use normal chain key to prove ownership
3. tx = Contract.registerPublicKey(account, pq_public_key, sig)
4. broadcast(tx)

Contract.registerPublicKey(account, pq_public_key, sig):
1. require(validatePubKeyFormat(pq_public_key))
2. require(recoverAccountFromSig(nonce, sig) == account)
3. store PKReg[account] = pq_public_key
4. emit PublicKeyRegistered(account)

Trust boundary: crossing from client to on-chain (ownership proof required to avoid spoofing)


## 2) Direct on-chain verification (pure on-chain)

Client-side:
1. message = buildPayload(...)
2. signature = PQ.sign(message, pq_private_key)
3. tx = Contract.protectedCall(message, signature)
4. broadcast(tx)

Contract.protectedCall(message, signature):
1. pk = PKReg[msg.sender]
2. ok = DilithiumVerify(pk, message, signature)   // heavy call
3. require(ok, "invalid signature")
4. performProtectedLogic()

Trust boundary: contract runtime must run correct verifier; gas and DoS risk.


## 3) Off-chain verification + on-chain attestation (hybrid)

Off-chain attestor:
1. receive(payload, signature, signerAccount)
2. pk = fetchPKFromChain(signerAccount)
3. ok = PQVerify(pk, payload, signature)
4. if ok:
     attestation = buildAttestation(payloadHash, signerAccount, attestorId, timestamp)
     signed = Attestor.sign(attestation)
     submitAttestationOnChain(signed)

Contract.processAttestation(attestation, attestorSig):
1. require(validateAttestor(attestorId))    // check bonding/quorum or attestor list
2. require(verifyAttestorSig(attestation, attestorSig))
3. store attestationRoot[...] = attestation
4. emit AttestationRecorded(...)

Client then submits a cheap tx referencing attestationRoot to perform protected action.

Trust boundary: trust moves to attestor(s); mitigate with multi-attestor/quorum or fraud proofs.


## 4) zk-proof assisted verification (succinct)

Off-chain prover:
1. generate witness = {pk, message, signature}
2. proof = ZKProver.prove(witness, circuit)
3. submit tx -> Contract.verifyZK(proof, publicInputs)

Contract.verifyZK(proof, publicInputs):
1. require(VerifyProof(proof, publicInputs))
2. performProtectedLogic()

Trust boundary: trust is placed in the soundness of the zk system and correctness of circuit encoding.


## 5) Safety checks and attacker mitigation (pseudocode)

on any registration or attestation input:
1. validate lengths and formats
2. check for replay (nonces, timestamps)
3. enforce rate limits / deposit requirements for expensive ops
4. log events for external watchers to verify off-chain

Example helper:
function validatePubKeyFormat(pk):
  if len(pk) not in ALLOWED_LENGTHS: return false
  if not isBytes(pk): return false
  return true


## 6) Testing suggestions
- Integrate NIST PQC test vectors and failing-edge vectors into unit tests.
- Simulate malicious attestor by submitting invalid attestations and exercise challenge windows.
- Measure gas for Dilithium verify on target runtime (EVM vs WASM) and set limits accordingly.

---

If you'd like, I can generate a minimal Solidity contract (registration + attestation acceptance) and a small Node.js script that acts as an attestor to demonstrate the hybrid flow. Which demo would you prefer? (stubbed on-chain verify, external attestor pattern, or zk mock?)
