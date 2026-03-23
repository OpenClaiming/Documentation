# 🔐 OpenClaiming EVM

Technical documentation for using the **OpenClaiming** smart contract as the canonical EIP-712 verifier and execution layer for standard OpenClaiming extensions on EVM-compatible blockchains.

**Canonical OpenClaiming contract address (all supported chains):**

```text
0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999
```

---

# 🚀 Overview

OpenClaiming supports multiple signing formats.

For EVM integrations, the relevant format is:

- `fmt = "EIP712"`

In this mode:

- claims remain OpenClaim JSON off-chain
- standard extensions are mapped into fixed typed EIP-712 structs
- signers sign typed digests
- the OpenClaiming contract verifies signatures, thresholds, and extension-specific rules
- claims may then be executed on-chain

This page is for **technical implementors** building:

- wallets
- relayers
- backend signers
- smart contract integrations
- indexers
- SDKs
- off-chain verifiers

---

# 🌐 What OpenClaiming Means on EVM

OpenClaiming is still fundamentally a protocol for **signed claims**.

On EVM, this does **not** mean the contract parses JSON.

Instead, the flow is:

```text
OpenClaim JSON
→ standard extension selected
→ extension-specific EIP-712 mapping
→ typed digest
→ signer recovery / multisig verification
→ extension-specific execution
```

The OpenClaiming contract is therefore:

- **not a JSON parser**
- **not a schema registry**
- **not a generic arbitrary EIP-712 engine**

It is a verifier/executor for **standardized OpenClaiming extensions** with predefined typed schemas.

In v1, the standard EIP-712 extensions are:

- `payments`
- `actions`

---

# 🧠 Core Model

There are three distinct layers:

## 1. OpenClaim layer

Off-chain JSON claims with fields such as:

- `ocp`
- `fmt`
- `iss`
- `sub`
- `stm`
- `key`
- `sig`
- `nbf`
- `exp`
- `nce`

Extensions are arrays of **nested OpenClaims**.

Example:

```json
{
  "ocp": 1,
  "payments": [
    {
      "ocp": 1,
      "fmt": "EIP712",
      "iss": "evm:56:address:0x1111111111111111111111111111111111111111",
      "sub": "evm:56:token:0x2222222222222222222222222222222222222222",
      "stm": {
        "chainId": "56",
        "verifyingContract": "0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999",
        "recipients": [
          "evm:56:address:0x3333333333333333333333333333333333333333"
        ],
        "max": "3000000",
        "line": "7"
      },
      "key": [
        "data:key/eip712,evm:56:address:0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
      ],
      "sig": [
        "0x..."
      ]
    }
  ]
}
```

## 2. EIP-712 mapping layer

A standard extension claim is converted into a fixed typed struct.

For example, a payment claim becomes a `Payment` struct.

## 3. Execution layer

The OpenClaiming contract verifies signatures and constraints, then either:

- executes directly
- partially executes
- aggregates claims
- delegates to another contract
- rejects execution

This depends on the extension and the integrating contracts.

---

# 🔑 Signers vs Issuer vs Subject

These must not be conflated.

## `key`

The `key` field identifies **who can sign**.

For EIP-712, `key[]` usually resolves to EVM signer addresses.

## `iss`

The issuer is the **semantic authority** behind the claim.

For payments, this is typically the payer authority.  
For actions, this is typically the authority on whose behalf an action is authorized.

## `sub`

The subject is the semantic object the claim is about.

For payments, this is usually the token or native asset.  
For actions, this is often the execution target, control object, or governed resource.

### Important

A signer does **not** have to equal `iss`.

Examples:

- `iss` may be an organization treasury
- `key[]` may contain board members or managers
- a contract may require those signers to be authorized to act for `iss`

This separation is intentional.

---

# 🧩 Standard Extensions

OpenClaiming v1 defines two standard extensions for EVM:

- `payments`
- `actions`

Each extension:

- is a nested OpenClaim
- defines its own `fmt`
- defines its own `key` and `sig`
- maps to its own fixed EIP-712 struct

---

# 🏷 Identifier Grammar

When `fmt = "EIP712"`, some claim identifiers are parsed into native EVM types.

Recommended grammar:

```text
evm:<chainId>:address:<0x...>
evm:<chainId>:token:<0x...>
evm:<chainId>:native
```

Examples:

```text
evm:56:address:0x1111111111111111111111111111111111111111
evm:56:token:0x2222222222222222222222222222222222222222
evm:56:native
```

## Semantics

- `evm:<chainId>:address:<0x...>` → EVM account or contract
- `evm:<chainId>:token:<0x...>` → ERC-20 style token
- `evm:<chainId>:native` → native chain currency, mapped to `address(0)`

### Validation rule

Implementors should reject claims if the chain id embedded in identifiers does not match:

- `stm.chainId`
- the actual execution chain
- the OpenClaiming deployment context

---

# 🧷 Key Format for EIP-712

For EIP-712 claims, the recommended key form is:

```text
data:key/eip712,evm:<chainId>:address:<0x...>
```

Example:

```text
data:key/eip712,evm:56:address:0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

This means:

- `fmt = EIP712`
- signer identity = the given EVM address

These keys identify who may sign. They do not embed raw public key bytes.

Remote URLs may also resolve to arrays of such key strings.

---

# ✍ Signature Format for EIP-712

For EIP-712, each signature should be encoded as:

- `0x`-prefixed hex
- 65-byte `r || s || v`

Example:

```json
"sig": [
  "0x8f4c..."
]
```

Each `sig[i]` corresponds to `key[i]`.

---

# 🔐 Multisignature Model

OpenClaiming supports multisignature natively.

For EIP-712:

- `key[]` defines the allowed signers
- `sig[]` contains the signatures
- each key resolves to an address
- each signature is recovered against the typed digest
- duplicate recovered addresses count once
- verification succeeds when quorum is satisfied

The required threshold may come from:

- policy passed by the caller
- claim semantics
- extension-specific contract logic

Typical default:

```text
minValid = 1
```

But integrations may require larger thresholds.

---

# 🧮 Numeric Encoding Rules

For EIP-712 claims, do **not** rely on JSON numbers for contract-significant values.

Implementors should encode values that map to `uint256` as strings:

- decimal strings such as `"3000000"`
- optionally hex strings such as `"0x2dc6c0"` if your implementation supports them consistently

Recommended v1 convention:

- use **decimal strings**
- avoid scientific notation
- avoid JSON numeric literals for anything that becomes `uint256`

Examples:

```json
"max": "3000000"
"line": "7"
"minimum": "2"
"fraction": "5000000000"
"delay": "3600"
"chainId": "56"
```

This avoids precision issues and makes cross-language behavior deterministic.

---

# 📜 Payments Extension

The `payments` extension standardizes payment authorization claims.

A payment claim authorizes:

- a semantic payer (`iss`)
- an asset (`sub`)
- one or more acceptable recipients
- a maximum cumulative authorization amount
- a replay / trustline bucket (`line`)

It does **not** force one specific execution policy.

Different contracts may interpret the same valid payment claim differently, as long as execution remains within the authorized set.

---

## Payment Claim Shape

```json
{
  "ocp": 1,
  "fmt": "EIP712",
  "iss": "evm:56:address:0x1111111111111111111111111111111111111111",
  "sub": "evm:56:token:0x2222222222222222222222222222222222222222",
  "stm": {
    "chainId": "56",
    "verifyingContract": "0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999",
    "recipients": [
      "evm:56:address:0x3333333333333333333333333333333333333333"
    ],
    "max": "3000000",
    "line": "7"
  },
  "key": [
    "data:key/eip712,evm:56:address:0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
  ],
  "sig": [
    "0x..."
  ]
}
```

---

## Payment EIP-712 Struct

The payment claim maps to a fixed Solidity struct:

```solidity
struct Payment {
	address payer;
	address token;
	bytes32 recipientsHash;
	uint256 max;
	uint256 line;
	uint256 nbf;
	uint256 exp;
}
```

---

## Field Conversion

### `iss` → `payer`

Parse:

```text
evm:<chainId>:address:<0x...>
```

into:

```solidity
address payer
```

### `sub` → `token`

Parse:

```text
evm:<chainId>:token:<0x...>
```

into:

```solidity
address token
```

If:

```text
sub = "evm:<chainId>:native"
```

then token becomes:

```solidity
address(0)
```

### `stm.recipients`

Convert all recipients to `address[]`.

### `stm.max`

Convert decimal string to `uint256`.

### `stm.line`

Convert decimal string to `uint256`.

### `nbf`, `exp`

If omitted, treat as zero.

---

## Recipients Hash

The full recipient array is not embedded directly in the struct.

Instead:

```solidity
recipientsHash = keccak256(abi.encode(recipients))
```

where `recipients` is `address[]`.

This:

- preserves order
- gives deterministic hashing
- lets contracts verify the authorized set compactly

---

## Payment Domain Separator

Use:

- `name = "OpenClaiming.payments"`
- `version = "1"`
- `chainId = stm.chainId`
- `verifyingContract = stm.verifyingContract`

The OpenClaiming contract address should normally be:

```text
0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999
```

on chains where Intercoin has deployed the canonical implementation.

---

## Payment Digest

The final digest is standard EIP-712:

```solidity
keccak256(
	abi.encodePacked(
		"\x19\x01",
		domainSeparator,
		structHash
	)
)
```

That is the digest signers sign and the contract verifies.

---

## Payment Verification Semantics

The OpenClaiming contract verifies:

- digest correctness
- signer recovery
- multisig threshold
- `nbf` / `exp`
- line capacity / replay state
- recipient compatibility
- claim max compatibility

Execution may still depend on integrating contracts.

Examples:

- ERC-20 transfer via allowance
- native coin settlement
- downstream payment routing
- integration with `IncomeContract`
- integration with `Community` policy

---

## Trustline / Line Model

OpenClaiming payments use line-based accounting.

Conceptually:

```text
payer → line → max → spent → open/closed
```

A payment claim references a line and defines claim-level authorization.

A contract computes remaining capacity using both claim and line state.

Typical mental model:

```text
remaining = min(claim.max, line.max) - line.spent
```

If line max is unlimited, line policy may defer entirely to claim max.

Implementors must follow the actual deployed contract semantics for:

- `lineOpen`
- `lineClose`
- `lineAvailable`
- claim max interpretation
- unlimited max behavior

---

## Multi-Line Aggregation

The OpenClaiming contract may support combining capacity across multiple claims / lines.

Typical requirements for aggregation:

- same payer
- same token
- same chosen recipient
- each claim individually valid
- total amount within combined remaining capacity

Integrators should use the contract’s explicit multi-payment execution functions where available.

---

## ERC-20 vs Native Coin

### ERC-20

Usually requires:

```solidity
token.approve(OPENCLAIMING, amount)
```

Then execution uses `transferFrom`.

### Native Coin

Represented by:

```solidity
token == address(0)
```

Native-coin execution typically uses `msg.value` and cannot be delegated in the same way as ERC-20 allowance-based spending.

Implementors must follow the deployed contract’s payment execution rules.

---

# ⚡ Actions Extension

The `actions` extension standardizes execution authorizations.

These claims authorize operations that may later be:

- invoked
- endorsed
- executed
- delayed
- rejected

An action claim represents **authorized intent**, not guaranteed immediate execution.

This maps naturally to workflow/governance contracts such as `ControlContract`.

---

## Action Claim Shape

```json
{
  "ocp": 1,
  "fmt": "EIP712",
  "iss": "evm:56:address:0x1111111111111111111111111111111111111111",
  "sub": "evm:56:address:0x4444444444444444444444444444444444444444",
  "stm": {
    "chainId": "56",
    "verifyingContract": "0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999",
    "contract": "evm:56:address:0x5555555555555555555555555555555555555555",
    "method": "a9059cbb",
    "params": "000000000000000000000000...",
    "minimum": "2",
    "fraction": "5000000000",
    "delay": "3600"
  },
  "key": [
    "data:key/eip712,evm:56:address:0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "data:key/eip712,evm:56:address:0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  ],
  "sig": [
    "0x...",
    "0x..."
  ]
}
```

---

## Action EIP-712 Struct

A fixed Solidity struct should be used, for example:

```solidity
struct Action {
	address authority;
	address subject;
	address contractAddress;
	bytes4 method;
	bytes32 paramsHash;
	uint256 minimum;
	uint256 fraction;
	uint256 delay;
	uint256 nbf;
	uint256 exp;
}
```

---

## Field Conversion

### `iss` → `authority`

Parse into EVM `address`.

### `sub` → `subject`

Parse into EVM `address`.

### `stm.contract`

Parse into EVM `address`.

### `stm.method`

Interpret as hex selector and convert to `bytes4`.

### `stm.params`

Interpret as hex-encoded ABI parameter bytes and hash:

```solidity
paramsHash = keccak256(paramsBytes)
```

### `stm.minimum`

Convert decimal string to `uint256`.

### `stm.fraction`

Convert decimal string to `uint256`.

### `stm.delay`

Convert decimal string to `uint256`.

### `nbf`, `exp`

If omitted, treat as zero.

---

## Action Domain Separator

Use:

- `name = "OpenClaiming.actions"`
- `version = "1"`
- `chainId = stm.chainId`
- `verifyingContract = stm.verifyingContract`

---

## Action Digest

Same EIP-712 construction:

```solidity
keccak256(
	abi.encodePacked(
		"\x19\x01",
		domainSeparator,
		structHash
	)
)
```

---

## Actions Execution Model

A valid action claim may still require multiple phases of execution.

Typical flow:

```text
invoke → endorse → quorum → execute
```

This matches systems like `ControlContract`, where:

- one actor proposes an operation
- additional actors endorse it
- quorum is evaluated
- execution may happen immediately or later
- delays may apply

So a valid action claim is best understood as:

> a standardized signed execution authorization envelope

not as an already-executed transaction.

---

## Interaction with Role-Based Systems

A contract may impose additional conditions before execution, such as:

- signer must hold an invoke role
- signer must hold an endorse role
- current group membership must be valid
- quorum must satisfy minimum and fraction thresholds
- delays must elapse

This is expected.  
OpenClaim verification does not replace contract-specific authorization logic.

---

# 🧾 EIP-712 Is Not Generic JSON

This is an important limitation.

EIP-712 is **not** a general-purpose extensible JSON encoding.

It requires:

- fixed schemas
- known field types
- deterministic ABI encoding
- no recursive arbitrary nested claims

Therefore:

- OpenClaim itself is extensible
- EIP-712 support is only defined for standard extensions with predefined mappings

In v1, that means:

- `payments`
- `actions`

Other extension ideas may still exist as OpenClaims, but unless they define a fixed EIP-712 projection and matching contract logic, they are not automatically supported on-chain.

---

# 🔧 What the OpenClaiming Contract Does

The canonical contract at:

```text
0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999
```

is expected to provide functionality such as:

- line management
- payment verification
- payment execution
- multisig signature verification
- action verification
- action execution / dispatch hooks
- typed digest verification for standard extensions

Exact method names and signatures should be taken from the deployed ABI for the specific release you integrate against.

This documentation describes the semantic contract model, not a substitute for the ABI.

---

# ❌ What the Contract Does Not Do

The OpenClaiming contract does **not**:

- parse OpenClaim JSON on-chain
- fetch remote keys
- construct typed payloads from JSON
- sign claims
- discover schemas dynamically
- act as a general arbitrary schema registry

Those responsibilities remain off-chain.

---

# 🛠 Implementation Checklist

If you are implementing an EVM integration, do all of the following:

## Claim Validation

- require `fmt == "EIP712"` for EVM path
- require `stm.chainId`
- require `stm.verifyingContract`
- ensure nested extension claim is structurally valid
- ensure numeric fields are strings and parse cleanly

## Identifier Validation

- parse `iss`, `sub`, recipients, contract targets, and signer keys
- ensure all embedded chain ids match
- reject unsupported identifier grammars

## Typed Struct Construction

- select extension (`payments` or `actions`)
- convert semantic identifiers into native EVM values
- hash arrays deterministically
- build exact struct values expected by the contract

## Domain Construction

- use extension-specific domain name
- use version `"1"`
- use chain id from claim
- use verifying contract from claim
- normally point to `0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999`

## Multisig Verification

- resolve `key[]`
- decode `sig[]`
- recover signer addresses
- deduplicate signers
- enforce quorum
- pass contract-level policy checks

## Execution

- for payments: select recipient and amount within authorized constraints
- for actions: submit invoke/endorse/execute flow as required
- handle contract-specific failure conditions
- do not assume valid signatures guarantee success

---

# 🧪 Frontend Example (ethers.js)

This is a conceptual payment example. Actual production code should validate the full OpenClaim and extension semantics before signing.

```javascript
import { ethers } from "ethers";

const OPENCLAIMING = "0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999";

const domain = {
	name: "OpenClaiming.payments",
	version: "1",
	chainId: 56,
	verifyingContract: OPENCLAIMING
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

const recipients = [
	"0x3333333333333333333333333333333333333333"
];

const recipientsHash = ethers.keccak256(
	ethers.AbiCoder.defaultAbiCoder().encode(["address[]"], [recipients])
);

const value = {
	payer: "0x1111111111111111111111111111111111111111",
	token: "0x2222222222222222222222222222222222222222",
	recipientsHash,
	max: "3000000",
	line: "7",
	nbf: 0,
	exp: 0
};

const signature = await signer.signTypedData(domain, types, value);
```

---

# 🧪 Solidity Integration Sketch

This is only a sketch. Use the actual ABI for production integration.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IOpenClaiming {
	// Use the exact ABI from the deployed contract release.
}

contract OpenClaimingConsumer {
	address public constant OPENCLAIMING =
		0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999;
}
```

---

# ⚠️ Security Notes

## Key resolution

Remote keys are possible in OpenClaim generally, but for EIP-712 execution paths, implementations usually resolve keys to signer addresses off-chain before submission.

## Replay protection

Replay protection may come from:

- `nbf`
- `exp`
- `line`
- contract state
- execution-specific nonces

Do not rely on time fields alone if replay resistance matters economically.

## Signature encoding

Ensure signatures are normalized and consistently decoded as 65-byte EVM signatures.

## Chain separation

Never ignore chain ids embedded in:

- `stm.chainId`
- identifiers
- signer keys

## Contract binding

Never sign or verify against the wrong `verifyingContract`.  
This should normally be:

```text
0x99996a51cc950d9822D68b83fE1Ad97B32Cd9999
```

for the canonical Intercoin deployment on supported chains.

---

# 📈 Summary

OpenClaiming EVM provides:

- portable signed payment authorizations
- portable signed action / invocation authorizations
- deterministic EIP-712 verification
- multisignature support
- line-based accounting for payments
- contract-friendly execution hooks
- composable integration with systems like `Community`, `IncomeContract`, and `ControlContract`

In short:

```text
OpenClaim JSON
→ standard extension
→ EIP-712 typed mapping
→ digest
→ signer recovery
→ multisig verification (e.g. with OpenClaiming contract)
→ contract-specific execution (e.g. via TrustedForwarder)
```

It turns signed claims into executable on-chain authorization primitives.
