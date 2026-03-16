# Blockchain Anchoring

OpenClaiming does not require blockchains, but claims may optionally be anchored to blockchain networks.

Anchoring provides an additional timestamped proof that a claim existed at a particular time.

## Anchoring Methods

Common approaches include:

- storing claim hashes on-chain
- embedding claims in transaction metadata
- anchoring Merkle roots of claim sets

## Benefits

Blockchain anchoring can provide:

- timestamp guarantees
- tamper-evident audit trails
- decentralized persistence

## Tradeoffs

Blockchains introduce additional cost, latency, and infrastructure complexity. Many use cases do not require them.

OpenClaiming therefore treats blockchain anchoring as optional.
