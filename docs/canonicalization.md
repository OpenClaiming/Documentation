# Canonicalization

Before a claim can be signed or verified, the JSON document must be transformed into a deterministic canonical form.

OpenClaiming uses the RFC 8785 JSON Canonicalization Scheme (JCS).

Canonicalization ensures that two parties produce identical byte representations of a JSON document prior to hashing and signing.

## Why Canonicalization Is Required

JSON allows multiple equivalent representations of the same data. For example:

- whitespace differences
- key ordering
- numeric formatting

If these differences were not normalized, the same claim could produce different hashes and signatures.

Canonicalization eliminates these ambiguities.

## RFC 8785 JSON Canonicalization Scheme

The JCS specification defines rules for producing a deterministic JSON representation.

Key properties include:

- lexicographic key ordering
- deterministic number formatting
- UTF-8 encoding
- no insignificant whitespace

## Signing Process

1. Construct the claim JSON.
2. Remove the `sig` field.
3. Canonicalize the JSON according to RFC 8785.
4. Hash the canonical representation.
5. Sign the hash.

Verification follows the same steps.
