# 🔐 OpenClaiming EVM

Canonical EIP-712 payment execution contract for OpenClaiming.

---

# 🚀 Overview

Implements:

- Delegated payments
- Trustline (line-based) accounting
- Deterministic EIP-712 verification (off-chain)
- Multi-line aggregation

OpenClaiming turns signed messages into portable, executable payment permissions.

---

# 🌐 What is OpenClaiming?

OpenClaiming is a protocol for expressing signed claims that can be:

- verified across systems
- executed on-chain or off-chain
- reused as portable authorization primitives

Instead of sending transactions, users sign claims, which can later be executed.

---

# 🧠 Core Model

Each payer has independent trustlines:

payer → line → max → spent → open/closed

Each claim:

- references a line
- defines a max
- defines allowed recipients

Execution enforces:

remaining = min(claim.max, line.max) - line.spent

---

# 🔐 Signing Model

OpenClaiming supports two modes:

## 1) System / Server Signing (Primary)

- Claims are generated and signed by backend systems
- Wallet UI is not relied upon
- Uses recipientsHash for deterministic verification

## 2) User Wallet Signing (Optional)

- Claims signed by users
- Requires including full recipient arrays for visibility
- May include both recipients and recipientsHash

---

# 📜 Payment Struct (EIP-712)

Payment(
    address payer,
    address token,
    bytes32 recipientsHash,
    uint256 max,
    uint256 line,
    uint256 nbf,
    uint256 exp
)

---

# 🔐 Signature Flow

## Off-chain

1. Build OpenClaiming JSON
2. Convert to typed struct
3. Compute hashes
4. Sign with wallet or server

## On-chain

1. Recompute struct hash
2. Recover signer
3. Enforce limits
4. Execute payment

---

# ⚙️ Usage

## 1. Open a line

openClaiming.lineOpen(account, line, max);

## 2. Sign claim off-chain

## 3. Execute on-chain

openClaiming.executePayment(...)

---

# 🧱 Contract Features

## Line management

- lineOpen(account, line, max)
- lineClose(account, line)
- lineAvailable(account, line)

## Execution

- executePayment(...)
- executePaymentMulti(...)

## Verification

- verifyPayment(...)

---

# 🔓 Delegation Model

## ERC20

Requires:

token.approve(OpenClaiming, amount);

Then:

transferFrom(payer → recipient)

---

## Native coin (ETH)

- token == address(0)
- must use msg.value
- cannot be delegated

---

# 🔄 Multi-Line Payments

Allows combining capacity across multiple claims:

- same payer
- same token
- same recipient

---

# 🧪 Solidity Integration Example

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./interfaces/IOpenClaiming.sol";

contract OpenClaimingConsumer {

    address public constant OPENCLAIMING_ADDRESS = 0x0000000000000000000000000000000000000000;

    IOpenClaiming public constant oc = IOpenClaiming(OPENCLAIMING_ADDRESS);

    uint256 public constant PRICE = 1e18;

    function purchase(
        IOpenClaiming.Payment calldata payment,
        address[] calldata recipients,
        bytes calldata signature
    ) external payable {

        bool ok = oc.verifyPayment(payment, signature);
        require(ok, "invalid signature");

        bool success = oc.executePayment{value: msg.value}(
            payment,
            recipients,
            signature,
            payment.line,
            address(this),
            PRICE
        );

        require(success, "payment failed");

        _handlePurchase(payment.payer);
    }

    function _handlePurchase(address buyer) internal {}
}

---

# 🌐 Frontend Example (ethers.js)

import { ethers } from "ethers";

const domain = {
    name: "OpenClaiming.payments",
    version: "1",
    chainId: 1,
};

const types = {
    Payment: [
        { name: "payer", type: "address" },
        { name: "token", type: "address" },
        { name: "recipientsHash", type: "bytes32" },
        { name: "max", type: "uint256" },
        { name: "line", type: "uint256" },
        { name: "nbf", type: "uint256" },
        { name: "exp", type: "uint256" }
    ]
};

const recipients = ["0xRecipientAddress..."];

const recipientsHash = ethers.keccak256(
    ethers.AbiCoder.defaultAbiCoder().encode(["address[]"], [recipients])
);

const value = {
    payer: signer.address,
    token: "0xTokenAddress...",
    recipientsHash,
    max: ethers.parseUnits("100", 18),
    line: 0,
    nbf: 0,
    exp: 0
};

const signature = await signer.signTypedData(domain, types, value);

---

# 🔍 Recipients Hash

Must be computed as:

keccak256(abi.encode(recipients))

---

# ⚠️ Important Notes

- Only EOAs sign claims
- No JSON parsing on-chain
- Off-chain must prepare all data correctly
- line 0 is valid
- max = 0 means unlimited
- claim max is enforced relative to line usage

---

# 🚫 What This Contract Does NOT Do

- does not parse JSON
- does not construct EIP-712 payloads
- does not sign messages
- does not store claims

All of that is handled off-chain.

---

# 📈 Summary

OpenClaiming EVM provides:

- portable signed payments
- deterministic EIP-712 verification
- trustline-based accounting
- composable execution layer

It turns signatures into executable economic primitives.
