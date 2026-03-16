<img src="https://repository-images.githubusercontent.com/1182806847/6f5f3db0-f5c5-40ab-ac26-8c4feb1ef6ba">

# 📜 OpenClaiming Protocol Documentation

Welcome to the official documentation for the **OpenClaiming Protocol (OCP)**.

OpenClaiming is a simple protocol for creating **cryptographically signed claims** that anyone can verify.

An **OpenClaim** is simply a signed JSON document stating something that can be verified using a public key.

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

The signature proves that the **issuer made the statement**. See more at **[OpenClaiming.org](https://openclaiming.org)**.

---

# 🚀 Goals of OpenClaiming

OpenClaiming aims to provide a **minimal primitive** for verifiable statements.

Key goals:

- simple to implement
- human readable
- decentralized
- independent of blockchains
- independent of identity providers
- easy to verify
- easy to publish anywhere

OpenClaiming intentionally focuses on the **core primitive**:

> A signed claim.

Everything else can be built on top of this.

---

# 📚 Documentation Index

## Core Concepts

| Topic | Description |
|------|-------------|
| 📘 Introduction | Overview of the protocol |
| 🧠 Concepts | Core terminology and model |
| 📄 OpenClaim Format | Structure of claim documents |
| 🔐 Signatures | How claims are signed |
| 🧾 Canonicalization | Deterministic JSON signing |
| 🌐 Publishing Claims | How claims are distributed |

---

## Identity and Authorization

| Topic | Description |
|------|-------------|
| 🔗 Identity Linking | Proving the same identity across services |
| 📱 Device & Session Keys | Multiple signing keys per user |
| 🛡 Capability Claims | Roles and permissions |

---

## Distributed Systems

| Topic | Description |
|------|-------------|
| ☁️ Intercloud Claims | Distributed systems coordination |
| ⛓ Blockchain Anchoring | Optional blockchain anchoring |

---

# 🧪 Reference Implementations

OpenClaiming includes minimal **reference implementations** to help developers integrate the protocol quickly.

These implementations demonstrate:

- canonicalization
- signing
- verification
- interoperability with test vectors

All libraries expose the same core interface:

```
canonicalize(claim)
sign(claim, privateKey)
verify(claim, publicKey)
```

---

## 🌐 Pick your language

All implementations are designed to be **interoperable**. A claim signed in one language must verify in another.

| Language | Repository |
|--------|------------|
| JavaScript | https://github.com/OpenClaiming/Javascript |
| Python | https://github.com/OpenClaiming/Python |
| Go | https://github.com/OpenClaiming/Go |
| Rust | https://github.com/OpenClaiming/Rust |
| PHP | https://github.com/OpenClaiming/PHP |
| Java | https://github.com/OpenClaiming/Java |
| Swift | https://github.com/OpenClaiming/Swift |

---

## ⚙ Example Usage

Example (JavaScript):

```javascript
import OpenClaim from "openclaiming";

const claim = {
  ocp: 1,
  iss: "example.com/alice",
  stm: {
    role: "admin"
  }
};

const signed = OpenClaim.sign(claim, privateKey);
const valid = OpenClaim.verify(signed, publicKey);

console.log(valid);
```

---

# 🌍 Why OpenClaiming?

Many systems require **verifiable statements**.

Existing solutions often involve:

- complex credential frameworks
- blockchain dependencies
- identity infrastructure
- heavy libraries

OpenClaiming instead focuses on a single primitive:

> **A signed claim that anyone can verify.**

Suitable for:

- identity linking
- permissions
- distributed coordination
- audit logs
- device authentication
- software signing
- blockchain anchoring
- intercloud protocols

---

# 🛠 Contributing

OpenClaiming is an open standard.

Contributions are welcome:

- specification improvements
- documentation clarity
- security review
- example claims
- language implementations

---

# 📜 License

MIT License
