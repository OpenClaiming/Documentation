# Publishing Claims

OpenClaim documents may be distributed through many channels.

Because verification depends only on cryptography, storage location does not affect validity.

## Common Locations

Claims may be published via:

- web servers
- REST APIs
- blockchain transactions
- distributed databases
- peer-to-peer networks

## Well-Known Location

A common convention is:

.well-known/openclaiming/<domain>/<identity>.json

This allows automated discovery of claims associated with a domain or user.

## Caching and Replication

Since claims are immutable once signed, they may be freely cached or mirrored.

Applications may maintain indexes or claim registries to speed discovery.
