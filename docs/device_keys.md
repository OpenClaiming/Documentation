# Device and Session Keys

Many users operate multiple devices. OpenClaiming supports representing these relationships using claims.

A primary identity key may issue claims delegating authority to device-specific keys.

## Device Delegation

Example use case:

1. User identity key signs a claim granting authority to a device key.
2. Device key signs session or activity claims.
3. Verifiers check the delegation chain.

This allows secure device authentication without exposing the root identity key.

## Session Keys

Short-lived session keys can be created for temporary operations such as:

- web sessions
- API calls
- ephemeral device authentication

Session claims can include expiration times or usage constraints.

## Security Advantages

Delegating authority reduces risk by limiting exposure of long-term keys.
