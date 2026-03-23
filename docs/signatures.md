# Signatures

OpenClaim v1 uses modern elliptic curve cryptography to provide secure digital signatures.

## Algorithms

The reference algorithm suite is:

- SHA-256 hashing
- ECDSA P-256 signing
- Base64 encoding for signature transport

## Signing Process

1. Construct claim JSON without the `sig` field.
2. Canonicalize JSON using RFC 8785.
3. Compute SHA-256 hash of the canonical representation.
4. Sign the hash using the issuer's private key.
5. Encode the signature in Base64.

## Verification

To verify a claim:

1. Remove `sig`.
2. Canonicalize JSON.
3. Recompute SHA-256 hash.
4. Verify ECDSA signature using issuer public key.

## Key Management

Issuers are responsible for protecting private keys. Compromise of the signing key allows an attacker to produce valid claims.

Rotation strategies and revocation mechanisms may be implemented at the application layer.

Keys specified as URLs (https://...) are revocable — updating the content at that URL effectively revokes or rotates the key. Keys specified as inline data:key/... URIs are non-revocable by design. Issuers should choose the form that matches their revocability intent.
