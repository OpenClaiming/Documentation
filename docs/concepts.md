# Concepts

OpenClaiming revolves around a small number of core concepts.

## Claim

A claim is a signed JSON document describing a statement. It includes metadata identifying the issuer, the subject, and the contents of the statement.

Claims are immutable once signed.

## Issuer

The issuer is the entity that signs the claim. The issuer controls the private key used to generate the signature.

The issuer may represent:

- a person
- an organization
- a service
- a device
- a software agent

## Subject

The subject is the entity the claim refers to. In many cases the issuer and subject are the same, but they may differ.

Examples:

- An organization issuing a claim about an employee
- A server issuing a claim about a device
- A person issuing a claim about their own identity

## Statement

The statement contains the actual information being asserted.

Examples include:

- identity ownership
- permission grants
- capability delegation
- device authentication
- cross-domain account linking

Statements are intentionally schema-flexible.

## Verification

Verification consists of:

1. canonicalizing the JSON
2. hashing the canonical form
3. verifying the cryptographic signature
4. checking validity constraints such as expiration

If verification succeeds, the verifier can be confident the issuer signed the statement.
