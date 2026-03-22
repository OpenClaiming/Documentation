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

## EVM Blockchains

Blockchains supporting EVM protocol (such as Ethereum Mainnet, Binance Smart Chain, Base, Arbitrum, Optimism, and many others) can be used to settle payments and close out payment lines with incremental micropayments. OpenClaiming Protocol publishes [ready-to-use smart contracts on the blockchains](https://github.com/OpenClaiming/EVM/blob/main/docs/evm.md) to facilitate this.