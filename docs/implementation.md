# Implementation Guide

Implementing OpenClaiming requires only a small number of cryptographic operations.

## Claim Creation

1. Construct the claim JSON without the `sig` field.
2. Canonicalize the JSON using RFC 8785.
3. Compute SHA-256 hash.
4. Sign using ECDSA P-256 private key.
5. Attach signature as Base64 in `sig`.

## Claim Verification

1. Remove `sig`.
2. Canonicalize JSON.
3. Compute SHA-256 hash.
4. Verify ECDSA signature.

## Reference Libraries

Implementations can be written in any language supporting:

- JSON canonicalization
- SHA-256
- ECDSA P-256

Example languages include JavaScript, Go, Rust, Python, PHP, Swift, and Java.
