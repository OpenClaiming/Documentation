<img src="https://repository-images.githubusercontent.com/1182806847/6f5f3db0-f5c5-40ab-ac26-8c4feb1ef6ba">

# 📜 OpenClaiming Protocol Documentation

![License](https://img.shields.io/badge/license-MIT-blue.svg)  
![Protocol](https://img.shields.io/badge/protocol-OCP%20v1-green.svg)  
![Status](https://img.shields.io/badge/status-draft-orange.svg)  
![Spec](https://img.shields.io/badge/spec-open-red.svg)

Welcome to the **official documentation for the OpenClaiming Protocol (OCP)**.

OpenClaiming defines a simple way to create **cryptographically signed claims** that anyone can verify — across systems, organizations, and blockchains.

---

# ✨ What is an OpenClaim?

An **OpenClaim** is a signed JSON document that states something.

Example:

```json
{
  "ocp": 1,
  "iss": "example.com/alice",
  "stm": {
    "role": "admin"
  },
  "key": ["data:key/es256;base64,..."],
  "sig": ["BASE64_SIGNATURE"]
}
```

The signatures prove **authorized parties made the statement**.

Learn more at **https://openclaiming.org**.

---

# 🚀 Design Goals

OpenClaiming was designed to be:

- Simple
- Human-readable
- Decentralized
- Implementation-friendly
- Independent of blockchains
- Independent of identity providers
- Easy to publish anywhere
- Extensible across execution environments

The protocol intentionally focuses on one primitive:

> A signed claim

Everything else is built on top of this.

---

# 🧩 Extensions

OpenClaiming supports **standardized extensions** for common use cases.

These are:

- 💸 **Payments** — spending authorization
- ⚡ **Actions** — execution / invocation authorization

Extensions are:

- arrays of **nested OpenClaims**
- independently signed and verifiable
- able to use different formats (`ES256`, `EIP712`)

Example:

```json
{
  "ocp": 1,
  "payments": [ ... ],
  "actions": [ ... ]
}
```

➡️ https://github.com/OpenClaiming/Documentation/blob/main/docs/extensions.md

---

# 🔐 Multisignature Model

OpenClaiming supports native multisignature:

```json
{
  "key": [
    "data:key/es256;base64,...",
    "data:key/es256;base64,..."
  ],
  "sig": [
    "BASE64_SIGNATURE",
    "BASE64_SIGNATURE"
  ]
}
```

- each signature corresponds to a key
- multiple signers may be required
- threshold is defined by application or contract

---

# 📚 Documentation Overview

## Core Protocol

- Introduction  
- Concepts  
- OpenClaim Format  
- Signatures  
- Canonicalization  
- Publishing Claims  

---

## Extensions

- Extensions Overview  
- Payments  
- Actions (Invocations)  

---

## Identity & Trust

- Identity Linking  
- Device & Session Keys  

---

## Distributed Systems

- Intercloud Claims  
- Blockchain Anchoring  

---

## ⛓ EVM Blockchains

OpenClaiming is **blockchain-agnostic**, but integrates cleanly with EVM chains.

EIP-712 support enables:

- on-chain verification of claims
- multisignature execution
- payment authorization
- governance and workflow execution

➡️ https://github.com/OpenClaiming/Documentation/blob/main/docs/evm.md

---

# ⚙ Example Usage

```javascript
import OpenClaim from "openclaiming";

const claim = {
  ocp: 1,
  iss: "example.com/alice",
  stm: { role: "admin" }
};

const signed = OpenClaim.sign(claim, privateKey);

const valid = OpenClaim.verify(signed);

console.log(valid);
```

---

# 🌐 Publishing Claims

Convention:

.well-known/openclaiming/<domain>/<identity>.json

Example:

example.com/.well-known/openclaiming/example.com/alice.json

---

# 🔁 Key Concepts Summary

- OpenClaim = signed statement  
- key = who can sign  
- sig = signatures  
- iss = authority  
- sub = subject  
- extensions = standardized semantics  
- formats = execution environments (ES256, EIP712)  

---

# 🧩 Ecosystem Repositories

- https://github.com/OpenClaiming/spec  
- https://github.com/OpenClaiming/Documentation  
- https://github.com/OpenClaiming/test-vectors  
- https://github.com/OpenClaiming/examples  
- https://openclaiming.org  

---

# 📜 License

MIT License
