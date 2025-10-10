# Network Zones — Summary

Short segmentation guidance for PQ deployments.

- Zone A: Trusted (HSM/KMS) — isolate and restrict access.
- Zone B: Compute/Attestation — provers and attestors; audited APIs to Zone A.
- Zone C: Public/API — internet-facing frontends; no private keys.

Best practices: use mTLS, least-privilege IAM, audit logs, and private links for hybrid setups.

Dev-mode quick tips: run local nodes/attestors in separate containers and never use production keys in dev.

