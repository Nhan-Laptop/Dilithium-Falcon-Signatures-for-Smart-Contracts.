Local Hardhat demo (external-attestor pattern)

1) Install dependencies

```bash
npm install
```

2) Start Hardhat node (new terminal)

```bash
npx hardhat node
```

3) Deploy contract (in another terminal)

```bash
npx hardhat run scripts/deploy.js --network localhost
```

Copy the deployed address and put it in `.env` as CONTRACT_ADDRESS.

4) Register a test PQ public key (use the first Hardhat account as user)

```bash
CONTRACT_ADDRESS=<addr> node scripts/register_pk.js
```

5) Set up `.env` with ATTESTOR_PRIV (one of the accounts printed by `npx hardhat node`) and CONTRACT_ADDRESS and USER_ADDRESS.

6) Run the attestor to submit an attestation (simulated PQ verify)

```bash
node attestor.js
```

7) Execute protected action (user calls protectedAction referencing attestation id 1)

```bash
CONTRACT_ADDRESS=<addr> npx hardhat run scripts/do_action.js --network localhost
```

Notes
- This demo uses a simulated PQ verify off-chain. Replace the simulation in `attestor.js` with your actual PQ verify step when available.
- Do not use production keys in dev; use the keys printed by Hardhat node.
