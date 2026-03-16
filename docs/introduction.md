# Introduction

OpenClaiming is a protocol for publishing cryptographically verifiable statements in a simple, portable format. A claim is a JSON document signed by a cryptographic key. Anyone possessing the public key can verify the authenticity and integrity of the claim without trusting the transport layer or the storage location.

The protocol is intentionally minimal. Rather than defining complex identity frameworks or credential lifecycles, OpenClaiming focuses on a single primitive:

A signed statement.

This primitive can be used to represent identity assertions, permissions, cross-domain links, device attestations, governance decisions, system state announcements, or cryptographic proofs.

## Design Goals

OpenClaiming was designed with the following principles:

- Minimal primitives
- Human-readable JSON format
- Cryptographic verification without infrastructure dependencies
- Transport and storage independence
- Interoperability with existing web systems

Claims may be stored in files, APIs, databases, distributed systems, or blockchains. Verification always relies solely on cryptography.

## Why OpenClaiming Exists

Many existing identity and credential systems require complex infrastructure or centralized authorities. These systems often introduce layers of indirection that obscure the fundamental operation being performed: verifying that a keyholder signed a statement.

OpenClaiming makes this explicit.

If an entity can sign a claim, it can assert something. If another entity trusts that key, the claim can be accepted.

This allows trust relationships to emerge organically across systems without requiring a global registry or authority.
