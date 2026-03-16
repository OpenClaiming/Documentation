# OpenClaim Format

OpenClaim documents follow a minimal JSON structure.

Example:

{
  "ocp": "1",
  "iss": "example.com",
  "sub": "alice",
  "stm": {
    "role": "admin"
  },
  "nbf": 1700000000,
  "exp": 1800000000,
  "sig": "..."
}

## Fields

### ocp

Protocol version identifier.

### iss

Issuer identifier. This identifies the entity that signed the claim.

### sub

Subject identifier. The entity that the statement refers to.

### stm

Statement payload. The actual data being asserted.

### nbf

Optional "not before" timestamp indicating when the claim becomes valid.

### exp

Optional expiration timestamp.

### sig

Cryptographic signature over the canonicalized claim.

## Extensibility

Additional fields may be added provided they do not interfere with canonicalization or signature verification.

The protocol intentionally avoids strict schemas to allow flexible usage.
