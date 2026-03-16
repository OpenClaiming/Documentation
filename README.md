<img src="https://repository-images.githubusercontent.com/1182806847/6f5f3db0-f5c5-40ab-ac26-8c4feb1ef6ba">

# 📜 OpenClaiming Protocol Documentation

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Protocol](https://img.shields.io/badge/protocol-OCP%20v1-green.svg)
![Status](https://img.shields.io/badge/status-draft-orange.svg)
![Spec](https://img.shields.io/badge/spec-open-red.svg)

Welcome to the **official documentation for the OpenClaiming Protocol (OCP)**.

OpenClaiming defines a simple way to create **cryptographically signed claims** that anyone can verify.

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
    "sig": "BASE64_SIGNATURE"
}
```

The signature proves the **issuer made the statement**.

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

The protocol intentionally focuses on one primitive:

> A signed claim

Everything else can be built on top of this.

---

# 📚 Documentation Overview

## Core Protocol

| Topic | Description |
|------|-------------|
| 📘 Introduction | https://github.com/OpenClaiming/Documentation/blob/main/docs/introduction.md |
| 🧠 Concepts | https://github.com/OpenClaiming/Documentation/blob/main/docs/concepts.md |
| 📄 OpenClaim Format | https://github.com/OpenClaiming/Documentation/blob/main/docs/openclaim_format.md |
| 🔐 Signatures | https://github.com/OpenClaiming/Documentation/blob/main/docs/signatures.md |
| 🧾 Canonicalization | https://github.com/OpenClaiming/Documentation/blob/main/docs/canonicalization.md |
| 🌐 Publishing Claims | https://github.com/OpenClaiming/Documentation/blob/main/docs/publishing_claims.md |

---

## Identity & Authorization

| Topic | Description |
|------|-------------|
| 🔗 Identity Linking | https://github.com/OpenClaiming/Documentation/blob/main/docs/identity_linking.md |
| 📱 Device & Session Keys | https://github.com/OpenClaiming/Documentation/blob/main/docs/device_keys.md |
| 🛡 Capability Claims | https://github.com/OpenClaiming/Documentation/blob/main/docs/capabilities.md |

---

## Distributed Systems

| Topic | Description |
|------|-------------|
| ☁️ Intercloud Claims | https://github.com/OpenClaiming/Documentation/blob/main/docs/intercloud.md |
| ⛓ Blockchain Anchoring | https://github.com/OpenClaiming/Documentation/blob/main/docs/blockchain.md |

---

## Implementation

| Topic | Description |
|------|-------------|
| ⚙ Implementation Guide | https://github.com/OpenClaiming/Documentation/blob/main/docs/implementation.md |
| 🔒 Security | https://github.com/OpenClaiming/Documentation/blob/main/docs/security.md |
| 📊 Comparisons | https://github.com/OpenClaiming/Documentation/blob/main/docs/comparisons.md |
| ❓ FAQ | https://github.com/OpenClaiming/Documentation/blob/main/docs/faq.md |

---

# 🧪 Reference Implementations

All implementations expose the same core interface:

```
canonicalize(claim)
sign(claim, privateKey)
verify(claim, publicKey)
```

---

## Pick your language

| Language | Repository |
|---------|-----------|
| JavaScript | https://github.com/OpenClaiming/javascript |
| Python | https://github.com/OpenClaiming/python |
| Go | https://github.com/OpenClaiming/go |
| Rust | https://github.com/OpenClaiming/rust |
| PHP | https://github.com/OpenClaiming/php |
| Java | https://github.com/OpenClaiming/java |
| Swift | https://github.com/OpenClaiming/swift |

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

const valid = OpenClaim.verify(signed, publicKey);

console.log(valid);
```

---

# 🔁 Interoperability

Official test vectors:

https://github.com/OpenClaiming/test-vectors

---

# 🌐 Publishing Claims

Convention:

```
.well-known/openclaiming/<domain>/<identity>.json
```

Example:

```
example.com/.well-known/openclaiming/example.com/alice.json
```

---

# 🧩 Ecosystem Repositories

| Repository | Purpose |
|-----------|--------|
| https://github.com/OpenClaiming/spec | Normative protocol specification |
| https://github.com/OpenClaiming/Documentation | Human-readable documentation |
| https://github.com/OpenClaiming/test-vectors | Interoperability tests |
| https://github.com/OpenClaiming/examples | Example claims |
| https://openclaiming.org | Official website |

---

# 📜 License

MIT License
