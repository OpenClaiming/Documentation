# 🔐 OpenClaiming EVM

Technical documentation for using the **OpenClaiming** smart contract as the canonical EIP-712 verifier and execution layer for standard OpenClaiming extensions on EVM-compatible blockchains.

**Canonical OpenClaiming contract address (all supported chains):**

```text
0x99999febd42cad798fe10ab0b1c563002fc99999
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
        "verifyingContract": "0x99999febd42cad798fe10ab0b1c563002fc99999",
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
For actions, this is the ControlContract that will receive the invoke and endorse calls.

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

OpenClaiming is designed to work with any contract implementing the minimal `pay(address recipient, uint256 amount)` interface, including `IncomeContract`, via the EIP-2771 forwarding path.

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
    "verifyingContract": "0x99999febd42cad798fe10ab0b1c563002fc99999",
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

EIP-712 type string:

```text
Payment(address payer,address token,bytes32 recipientsHash,uint256 max,uint256 line,uint256 nbf,uint256 exp)
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

Convert decimal string to `uint256`. Use `"0"` or omit for the default line (always open, no contract-level cap beyond claim max).

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

## Payment Execution Paths

### Direct ERC-20 (`incomeContract = address(0)`)

The standard path. OpenClaiming calls `token.transferFrom(payer, recipient, amount)` directly.

Requires: `token.approve(OPENCLAIMING, amount)` from the payer beforehand.

### Via IncomeContract (`incomeContract != address(0)`)

For contracts implementing `pay(address recipient, uint256 amount)` — such as `IncomeContract`. OpenClaiming forwards the call via EIP-2771, so `_msgSender()` inside the target contract resolves to `p.payer` rather than OpenClaiming's address. This preserves on-chain identity and allows the target contract to apply payer-specific access controls.

Requires: OpenClaiming must be registered as TrustedForwarder on the target contract.

### Native Coin (`p.token = address(0)`)

`msg.sender` must be `p.payer` and `msg.value` must equal `amount`. Native coin cannot be delegated — the payer must call the contract directly.

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

---

## Trustline / Line Model

OpenClaiming payments use line-based accounting.

```text
payer → line → max → spent → open/closed
```

**Line 0 (default line):** Always open. No contract-level cap beyond the claim's own `max`. Use for most payment flows. No `lineOpen()` call required.

**Lines ≥ 1:** Optional budget-isolation buckets. Must be opened via `lineOpen()` before use. Each line has its own ceiling (`max`) tracked separately from the claim ceiling. The effective remaining capacity is `min(claim.max, line.max) - line.spent`.

All lines draw from the same underlying token balance and allowance.

### Line management functions

- `lineOpen(account, line, max)` — open a line or update its ceiling. Only the account itself or its Ownable owner may call this.
- `lineClose(account, line)` — close a line. Cannot close line 0. Spent history is preserved.
- `lineIsOpen(account, line)` — returns `true` if the line is open (always `true` for line 0).
- `lineAvailable(account, line, token, claimMax)` — returns the remaining capacity considering both line and claim ceilings.

---

## Multi-Line Aggregation

The OpenClaiming contract supports combining capacity across multiple claims via `paymentsExecuteSignatures`. Typical requirements:

- same payer
- same token
- same chosen recipient
- each claim individually valid
- total amount within combined remaining capacity

---

## ERC-20 vs Native Coin

### ERC-20

Requires:

```solidity
token.approve(OPENCLAIMING, amount)
```

Execution uses `transferFrom`.

### Native Coin

```solidity
token == address(0)
```

Uses `msg.value`. Cannot be delegated — payer must be `msg.sender`.

---

# ⚡ Actions Extension

The `actions` extension standardizes execution authorizations for governance operations.

These claims authorize operations that may be:

- invoked
- endorsed
- executed
- delayed
- rejected

An action claim represents **authorized intent**, not guaranteed immediate execution.

OpenClaiming is designed to work with any contract implementing the minimal `invoke(address, string, string)` and `endorse(uint256)` interface, including `ControlContract`.

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
    "verifyingContract": "0x99999febd42cad798fe10ab0b1c563002fc99999",
    "contract": "evm:56:address:0x5555555555555555555555555555555555555555",
    "method": "a9059cbb",
    "params": "000000000000000000000000...",
    "minimum": "2",
    "fraction": "5000000000",
    "delay": "3600",
    "invoker": "evm:56:address:0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
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

`stm.invoker` may be omitted or set to the zero address for the v1 execution path (see below).

---

## Action EIP-712 Struct

```solidity
struct Action {
    address authority;
    address subject;
    address contractAddress;
    bytes4  method;
    bytes32 paramsHash;
    uint256 minimum;
    uint256 fraction;
    uint256 delay;
    address invoker;
    uint256 nbf;
    uint256 exp;
}
```

EIP-712 type string:

```text
Action(address authority,address subject,address contractAddress,bytes4 method,bytes32 paramsHash,uint256 minimum,uint256 fraction,uint256 delay,address invoker,uint256 nbf,uint256 exp)
```

### The `invoker` field

`invoker` determines which execution path OpenClaiming uses:

- `invoker = address(0)` → `actionsExecute()` path. OpenClaiming calls `invoke()` and `endorse()` directly as `msg.sender`. The target ControlContract must grant OpenClaiming the invoke and endorse roles in its Community contract. Only one endorsement is contributed per transaction; if `minimum > 1`, remaining endorsements must come from separate EOA transactions.

- `invoker = <address>` → `actionsInvoke()` path. OpenClaiming forwards `invoke()` as the invoker and `endorse()` as each valid signer via EIP-2771. The target ControlContract must register OpenClaiming as TrustedForwarder and use `_msgSender()` throughout. Multi-endorser quorum can be met in one transaction.

`minimum`, `fraction`, and `delay` are committed in the signed digest for policy auditability and enforcement by v2 contracts. In v1 ControlContract these values are stored per-method in `addMethod()` and are not passed to `invoke()`.

---

## Field Conversion

### `iss` → `authority`

Parse into EVM `address`.

### `sub` → `subject`

Parse into EVM `address`. This is the ControlContract that will receive the `invoke()` and `endorse()` calls.

### `stm.contract` → `contractAddress`

Parse into EVM `address`. This is the target contract that ControlContract will ultimately call.

### `stm.method` → `method`

Interpret as 8-character hex selector (without `0x` prefix) and convert to `bytes4`.

### `stm.params` → `paramsHash`

Interpret as hex-encoded ABI parameter bytes and hash:

```solidity
paramsHash = keccak256(paramsBytes)
```

The raw `paramsBytes` must be passed separately to execution functions alongside the struct.

### `stm.minimum` → `minimum`

Convert decimal string to `uint256`.

### `stm.fraction` → `fraction`

Convert decimal string to `uint256`. Represents fractional quorum out of `1e10` (e.g. `"5000000000"` = 50%).

### `stm.delay` → `delay`

Convert decimal string to `uint256`. Seconds after quorum before execution is allowed.

### `stm.invoker` → `invoker`

Parse `evm:<chainId>:address:<0x...>` into EVM `address`. If omitted or zero address, use `actionsExecute()`. If non-zero, use `actionsInvoke()`.

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
- quorum is evaluated against minimum and fraction thresholds
- execution may happen immediately or after a delay
- execution calls the target contract with the committed params

A valid action claim is best understood as a **standardized signed execution authorization envelope**, not an already-executed transaction.

### Execution functions

**`actionsExecute(a, params, signers, signatures, minValid)`** — v1 ControlContract path. `a.invoker` must be `address(0)`. OpenClaiming acts as `msg.sender` for both `invoke()` and `endorse()`. The invokeID is re-derived from `keccak256(block.timestamp, block.difficulty, address(OpenClaiming))` to match the v1 ControlContract's `generateInvokeID()`. Use when the ControlContract does not support EIP-2771.

**`actionsInvoke(a, params, signers, signatures, minValid)`** — v2 ControlContract path. `a.invoker` must be non-zero and present in `signers` with a valid signature. OpenClaiming forwards `invoke()` as `a.invoker` and `endorse()` as each valid signer via EIP-2771. The invokeID is re-derived from `keccak256(block.timestamp, block.prevrandao, a.invoker)`. Multi-endorser quorum can be satisfied in one transaction. Use when the ControlContract registers OpenClaiming as TrustedForwarder and uses `_msgSender()` throughout.

---

## Interaction with Role-Based Systems

A contract may impose additional conditions before execution, such as:

- signer must hold an invoke role in Community
- signer must hold an endorse role in Community
- current group membership must be valid (heartbeat)
- quorum must satisfy minimum and fraction thresholds
- delays must elapse before `execute()` is called

OpenClaim verification does not replace contract-specific authorization logic. A valid OpenClaiming signature check is a necessary condition for execution, not a sufficient one.

---

# 🧾 EIP-712 Is Not Generic JSON

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
0x99999febd42cad798fe10ab0b1c563002fc99999
```

provides:

**Line management:** `lineOpen`, `lineClose`, `lineIsOpen`, `lineAvailable`

**Payment verification:** `paymentsVerify`, `paymentsVerifySignatures`, `paymentsPreFlight`

**Payment execution:** `paymentsExecute` (single signature), `paymentsExecuteSignatures` (multisig)

**Action verification:** `actionsVerify`, `actionsVerifySignatures`, `actionsPreFlight`

**Action execution:** `actionsExecute` (v1 ControlContract, direct call), `actionsInvoke` (v2 ControlContract, EIP-2771 forwarding)

**Digest utilities:** `paymentsDomainSeparator`, `paymentsHash`, `paymentsDigest`, `paymentsRecoverSigner`, `actionsDomainSeparator`, `actionsHash`, `actionsDigest`, `actionsRecoverSigner`, `actionsHashParams`, `paymentsHashRecipients`

**Signature utilities:** `recoverSigner`, `verify`, `verifySignatures`

Exact method signatures should be taken from the deployed ABI for the specific release you integrate against.

---

# ❌ What the Contract Does Not Do

The OpenClaiming contract does **not**:

- parse OpenClaim JSON on-chain
- fetch remote keys
- construct typed payloads from JSON
- sign claims
- discover schemas dynamically
- act as a general arbitrary schema registry
- verify that target contracts (ControlContract, IncomeContract, etc.) are genuine Intercoin deployments — callers are responsible for passing trusted addresses

Those responsibilities remain off-chain.

---

# 🛠 Implementation Checklist

## Claim Validation

- require `fmt == "EIP712"` for EVM path
- require `stm.chainId`
- require `stm.verifyingContract`
- ensure nested extension claim is structurally valid
- ensure numeric fields are strings and parse cleanly

## Identifier Validation

- parse `iss`, `sub`, recipients, contract targets, signer keys, and `stm.invoker`
- ensure all embedded chain ids match
- reject unsupported identifier grammars

## Typed Struct Construction

- select extension (`payments` or `actions`)
- convert semantic identifiers into native EVM values
- hash arrays and params deterministically
- build exact struct values expected by the contract
- for actions: set `invoker = address(0)` for v1 path, or a valid signer address for v2 path

## Domain Construction

- use extension-specific domain name (`"OpenClaiming.payments"` or `"OpenClaiming.actions"`)
- use version `"1"`
- use chain id from claim
- use verifying contract from claim — normally `0x99999febd42cad798fe10ab0b1c563002fc99999`

## Multisig Verification

- resolve `key[]`
- decode `sig[]`
- recover signer addresses
- deduplicate signers
- enforce quorum (`minValid`)
- for `actionsInvoke`: verify `invoker` is among the valid signers

## Execution

- for payments: select recipient and amount within authorized constraints; choose direct ERC-20 or IncomeContract path
- for actions: choose `actionsExecute` (v1) or `actionsInvoke` (v2) based on `stm.invoker`
- pass raw `params` bytes alongside the Action struct — the contract verifies `keccak256(params) == a.paramsHash`
- handle contract-specific failure conditions
- do not assume valid signatures guarantee success

---

# 🧪 Frontend Example (ethers.js)

Conceptual payment signing example. Production code should validate the full OpenClaim before signing.

```javascript
import { ethers } from "ethers";

const OPENCLAIMING = "0x99999febd42cad798fe10ab0b1c563002fc99999";

const domain = {
    name: "OpenClaiming.payments",
    version: "1",
    chainId: 56,
    verifyingContract: OPENCLAIMING
};

const types = {
    Payment: [
        { name: "payer",          type: "address" },
        { name: "token",          type: "address" },
        { name: "recipientsHash", type: "bytes32" },
        { name: "max",            type: "uint256" },
        { name: "line",           type: "uint256" },
        { name: "nbf",            type: "uint256" },
        { name: "exp",            type: "uint256" }
    ]
};

const recipients = ["0x3333333333333333333333333333333333333333"];

const recipientsHash = ethers.keccak256(
    ethers.AbiCoder.defaultAbiCoder().encode(["address[]"], [recipients])
);

const value = {
    payer:          "0x1111111111111111111111111111111111111111",
    token:          "0x2222222222222222222222222222222222222222",
    recipientsHash,
    max:            "3000000",
    line:           "7",
    nbf:            0,
    exp:            0
};

const signature = await signer.signTypedData(domain, types, value);
```

---

# 🧪 Action Signing Example (ethers.js)

```javascript
import { ethers } from "ethers";

const OPENCLAIMING = "0x99999febd42cad798fe10ab0b1c563002fc99999";

const domain = {
    name: "OpenClaiming.actions",
    version: "1",
    chainId: 56,
    verifyingContract: OPENCLAIMING
};

const types = {
    Action: [
        { name: "authority",       type: "address" },
        { name: "subject",         type: "address" },
        { name: "contractAddress", type: "address" },
        { name: "method",          type: "bytes4"  },
        { name: "paramsHash",      type: "bytes32" },
        { name: "minimum",         type: "uint256" },
        { name: "fraction",        type: "uint256" },
        { name: "delay",           type: "uint256" },
        { name: "invoker",         type: "address" },
        { name: "nbf",             type: "uint256" },
        { name: "exp",             type: "uint256" }
    ]
};

// Raw ABI-encoded params (without method selector)
const params = ethers.AbiCoder.defaultAbiCoder().encode(
    ["address", "uint256"],
    ["0x3333333333333333333333333333333333333333", "1000000"]
);
const paramsHash = ethers.keccak256(params);

const value = {
    authority:       "0x1111111111111111111111111111111111111111",
    subject:         "0x4444444444444444444444444444444444444444", // ControlContract
    contractAddress: "0x5555555555555555555555555555555555555555", // target
    method:          "0xa9059cbb",                                  // transfer(address,uint256)
    paramsHash,
    minimum:         "2",
    fraction:        "5000000000",
    delay:           "0",
    invoker:         "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa", // non-zero = v2 path
    nbf:             0,
    exp:             0
};

const signature = await signer.signTypedData(domain, types, value);

// For v1 path (actionsExecute), set invoker to ethers.ZeroAddress
// and ensure OpenClaiming holds invoke+endorse roles in Community.
```

---

# 🧪 Solidity Integration Sketch

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

interface IOpenClaiming {
    // Use the exact ABI from the deployed contract release.
    // Key functions:
    //   paymentsExecute(Payment, address[] recipients, bytes sig, address recipient, uint256 amount, address incomeContract)
    //   paymentsExecuteSignatures(Payment, address[] recipients, address recipient, uint256 amount, address[] signers, bytes[] signatures, uint256 minValid, address incomeContract)
    //   actionsExecute(Action, bytes params, address[] signers, bytes[] signatures, uint256 minValid)
    //   actionsInvoke(Action, bytes params, address[] signers, bytes[] signatures, uint256 minValid)
}

contract OpenClaimingConsumer {
    address public constant OPENCLAIMING =
        0x99999febd42cad798fe10ab0b1c563002fc99999;
}
```

---

# ⚠️ Security Notes

## Key resolution

Remote keys are possible in OpenClaim generally, but for EIP-712 execution paths, implementations usually resolve keys to signer addresses off-chain before submission.

## Replay protection

Replay protection may come from:

- `nbf` / `exp` time bounds
- `line` spending state (spent counter advances on each execution)
- `max` ceiling (claim cannot be executed beyond its stated maximum)
- contract state for actions (invokeID is consumed once endorsed/executed)

Do not rely on time fields alone if replay resistance matters economically. The `line` and `max` fields are the primary economic replay controls for payments.

## Signature encoding

Ensure signatures are normalized and consistently decoded as 65-byte EVM signatures (`r || s || v`). The contract enforces low-s canonicality to prevent signature malleability.

## Chain separation

Never ignore chain ids embedded in:

- `stm.chainId`
- identifiers
- signer keys

## Contract binding

Never sign or verify against the wrong `verifyingContract`. This should normally be:

```text
0x99999febd42cad798fe10ab0b1c563002fc99999
```

## Contract trust

OpenClaiming does not verify that target contracts (`subject`, `incomeContract`) are genuine Intercoin deployments. Callers are responsible for passing trusted addresses. A malicious contract at `a.subject` could behave arbitrarily when `invoke()` or `endorse()` is called.

---

# 📈 Summary

OpenClaiming EVM provides:

- portable signed payment authorizations
- portable signed action / invocation authorizations
- deterministic EIP-712 verification
- multisignature support
- line-based accounting for payments
- two action execution paths: direct (v1 ControlContract) and EIP-2771 forwarding (v2 ControlContract)
- composable integration with systems like `Community`, `IncomeContract`, and `ControlContract`

```text
OpenClaim JSON
→ standard extension (payments or actions)
→ EIP-712 typed mapping
→ digest
→ signer recovery + multisig verification
→ contract execution:
    payments  → transferFrom / IncomeContract.pay (EIP-2771)
    actionsExecute → ControlContract.invoke + endorse (direct, v1)
    actionsInvoke  → ControlContract.invoke + endorse (EIP-2771, v2)
```

It turns signed claims into executable on-chain authorization primitives.
