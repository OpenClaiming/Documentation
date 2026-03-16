# Capability Claims

Claims may represent permissions or capabilities granted by one entity to another.

Example capabilities include:

- administrative privileges
- API access
- service delegation
- authorization tokens

## Delegation

An issuer may grant a capability to another key by signing a claim describing the permission.

Example statement:

{
  "capability": "write-access",
  "resource": "example-database"
}

## Delegation Chains

Capabilities may be further delegated through chains of claims.

Verification requires evaluating the entire chain to ensure authority originates from a trusted root.
