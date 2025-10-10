
# Legal & Regulatory — Summary

Key points:
- Avoid storing PII on-chain; keep sensitive data off-chain and encrypted.
- Check local crypto export controls and prefer NIST-selected PQ algorithms for interoperability.
- Keep audit trails (code versions, test vectors, logs) for compliance.
- If running attestor services or handling value, consider SLA/liability, bonding, and consult legal counsel for KYC/AML implications.

Dev-mode note: never use production keys in local demos and keep demo artifacts separate from production evidence.

Dev-mode notes
- When using Ethereum dev mode (Hardhat/Ganache/Foundry) or local Cosmos testnets, ensure no production keys or sensitive data are used in demos. Mark test fixtures clearly and segregate testnets from production environments.
- Keep legal / compliance evidence (audit logs, test vectors) available, even for demos, to ease later production audits.

