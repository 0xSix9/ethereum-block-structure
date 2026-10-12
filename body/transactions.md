# Understanding Transactions in the Ethereum Block Body

![Ethereum block body](../images/body.png)

## Introduction

An Ethereum execution block contains two major parts: a **header** and a **body**. The header identifies the block and commits cryptographically to important block data. The body contains the ordered transaction list and other execution-payload data specified by the active Ethereum fork.

Transactions are signed requests that ask Ethereum to perform state transitions. They can transfer ETH, call smart contracts, deploy contracts, publish blob-related commitments for rollups, or carry authorizations that let an externally owned account (EOA) delegate execution to code.

> **Key idea:** the transaction list, the receipts produced by execution, and the resulting world state are different objects. The block header commits to them separately through `transactionsRoot`, `receiptsRoot`, and `stateRoot`.

## 1. Where Do Transactions Live?

At a high level, an execution block can be pictured like this:

~~~text
Ethereum Execution Block
├── Header
│   ├── parentHash
│   ├── stateRoot
│   ├── transactionsRoot  ──────────┐
│   ├── receiptsRoot                │ commits to
│   └── ... other header fields    │
└── Body                             │
    ├── Transactions  <──────────────┘
    ├── Ommer headers (historical format; empty post-Merge)
    └── Withdrawals (since Shanghai)
~~~

This is a conceptual diagram. The exact execution-payload structure depends on the fork. In modern Ethereum, ommers are no longer included, although legacy ommers-related fields remain in the execution header for protocol compatibility.

Transactions are stored in the body as an **ordered list**. Their order matters because each transaction executes against the state produced by earlier transactions in the same block. Reordering transactions can change account nonces, balances, contract storage, emitted logs, and whether later transactions succeed.

A block may contain zero transactions. An empty transaction list is valid and has a deterministic empty-trie root.

## 2. What Is an Ethereum Transaction?

An Ethereum transaction is a signed request to perform an operation on the network. Depending on its type, it can include fields such as:

- **Chain ID:** identifies the chain for which the transaction is intended and supports replay protection.
- **Nonce:** the sender account's transaction sequence number.
- **Destination (`to`):** the recipient address or contract. Some transaction types permit a null destination to request contract creation.
- **Value:** the ETH amount to transfer, denominated in wei.
- **Input data (`data` or `input`):** arbitrary bytes, often containing a contract function selector and encoded arguments.
- **Gas limit:** the maximum execution gas the sender allows the transaction to consume.
- **Fee fields:** legacy transactions use `gasPrice`; EIP-1559-style transactions use `maxFeePerGas` and `maxPriorityFeePerGas`.
- **Signature:** proves authorization by the sender's key. The sender address is recovered from the signature; it is not ordinarily stored as a separate signed field in the serialized transaction.

The field list varies with transaction type. Blob transactions add blob-fee fields and versioned blob hashes. Set-code transactions add an authorization list.

A transaction is not synonymous with a contract call. A plain ETH transfer is also a transaction, and a transaction sent to a smart contract may execute code, transfer tokens, emit events, or modify storage.

## 3. What Happens When a Transaction Is Executed?

A transaction is not accepted merely because a wallet has signed it. The protocol checks validity, and valid transactions are executed in block order as part of Ethereum's state transition.

~~~mermaid
flowchart TD
    A["Wallet or application constructs transaction"] --> B["Sign transaction"]
    B --> C["Broadcast to an execution client"]
    C --> D["Validate signature, nonce, fees, and format"]
    D --> E["Include transaction in a proposed block"]
    E --> F["Execute in block order"]
    F --> G{"EVM execution succeeds?"}
    G -->|Yes| H["Apply execution changes and logs"]
    G -->|No or revert| I["Revert execution effects; charge applicable gas"]
    H --> J["Create transaction receipt"]
    I --> J
    J --> K["Commit block data and resulting state"]
~~~

This is a simplified learning diagram rather than a literal pipeline for every client. Block building, transaction propagation, validation, and consensus involve several components.

### 3.1 Validation

Execution clients check properties such as transaction encoding, signature, nonce, chain ID where applicable, fee parameters, gas limit, and whether the sender can cover the required upfront cost. The details vary by type and active fork.

A transaction must also satisfy block-level constraints. For example, execution gas must fit within the block's remaining gas allowance, and a blob transaction must satisfy the active blob-gas rules.

### 3.2 Execution and state changes

Transactions execute sequentially against the evolving state. A transaction may transfer ETH, call contracts, create a contract, or update storage.

If execution reverts or fails, its EVM-level state changes and logs are rolled back. However, the transaction may still be included in a valid block and consume gas. The sender's nonce and fee accounting are not simply undone as though the transaction had never existed.

Therefore, **included** and **successful** are different statuses. Inclusion means the transaction is part of the block; success describes its execution result.

### 3.3 Receipts

After each transaction executes, Ethereum produces a receipt containing execution-result information such as status, cumulative gas used, logs, and a logs bloom filter. Receipts are committed through the header's `receiptsRoot`; they are not the transaction list itself.

## 4. Ethereum Transaction Types

Ethereum's transaction format has evolved over time. The typed transaction envelope introduced by [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718) allows new formats to coexist with the original legacy format.

| Type | Standard | Name | Main purpose |
|---|---|---|---|
| Legacy (untyped) | Original format; replay protection improved by EIP-155 | Legacy transaction | ETH transfers, contract calls, and contract creation |
| `0x01` | [EIP-2930](https://eips.ethereum.org/EIPS/eip-2930) | Access-list transaction | Explicitly lists addresses and storage keys expected to be accessed |
| `0x02` | [EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) | Dynamic-fee transaction | Uses a base-fee-aware maximum fee and priority-fee cap |
| `0x03` | [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844) | Blob transaction | Commits to blobs and pays for blob gas, primarily for rollup data availability |
| `0x04` | [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702) | Set-code transaction | Carries authorizations that let an EOA delegate execution to code |

Typed transactions include a type byte; the original legacy transaction does not begin with one of these type identifiers. The table describes the principal formats used by Ethereum execution, not every format that might exist on every EVM-compatible chain.

### Legacy transactions

The original transaction format includes fields such as nonce, gas price, gas limit, destination, value, input data, and signature values. It remains part of Ethereum's history and is still supported.

### Type 1 — Access-list transactions

EIP-2930 added an access list: a list of addresses and storage keys that the transaction expects to access. Access lists affect gas accounting for the specified accesses. They do not grant permissions, force code to access a key, or guarantee that every transaction will be cheaper.

### Type 2 — Dynamic-fee transactions

EIP-1559 introduced a fee market with a base fee that changes with block demand. A type 2 transaction specifies:

- **`maxFeePerGas`:** the maximum total price per unit of execution gas that the sender is willing to pay.
- **`maxPriorityFeePerGas`:** the maximum priority fee per unit of gas offered to the block proposer.

The effective gas price depends on the block's base fee and the transaction's fee caps. The base-fee portion is burned; the priority-fee portion is paid to the execution block's fee recipient under the protocol's rules. The sender does not necessarily pay the full `maxFeePerGas` cap.

### Type 3 — Blob transactions

EIP-4844 introduced blob transactions to support rollup data availability with a fee market separate from ordinary EVM execution gas. A blob transaction includes fields such as `maxFeePerBlobGas` and `blobVersionedHashes`.

The blob bytes themselves are **not embedded as ordinary input data in the execution block body**. They are propagated as blob sidecars through Ethereum's blob data-availability mechanism. The transaction commits to versioned hashes of the blobs, while the sidecar carries the blob data and associated proof material.

Blob transactions pay separate blob-gas fees in addition to normal execution gas. They require a destination address and cannot be used for contract creation.

### Type 4 — Set-code transactions

EIP-7702 introduced set-code transactions. They contain an `authorizationList` of signed authorization tuples. Processing an authorization can set a delegation indicator on an EOA, allowing calls to that account to execute code from a designated address in the EOA's context.

This mechanism is relevant to smart-account experiences and transaction-batching designs. It does not make an EOA identical to a conventional contract account in every respect. Applications must account for authorization scope, chain ID, nonce, and the security properties of delegated code.

## 5. How Are Transactions Serialized?

Transactions require an unambiguous byte representation for signing, networking, validation, and cryptographic commitment.

Legacy transactions use Recursive Length Prefix (RLP) encoding for their fields. Typed transactions follow the EIP-2718 envelope:

~~~text
TransactionType || TransactionPayload
~~~

For the typed transactions described above, the payload is encoded according to that type's specification, using RLP for the listed payload formats. The type byte identifies how the payload must be interpreted.

~~~text
Legacy transaction:
    RLP(transaction fields)

Typed transaction:
    Transaction type byte || RLP(type-specific fields)
~~~

Field order is protocol-defined and differs between transaction types. Implementations must use the correct schema for the type rather than treating all transactions as one universal RLP list.

Serialization matters because the transaction hash and signature rules depend on the prescribed byte representation. Changing the encoding or field order does not produce an equivalent signed transaction.

## 6. What Is transactionsRoot?

The header's **`transactionsRoot`** is the root hash of a Merkle Patricia Trie containing the transactions in that block. It commits to the exact ordered transaction list, not every transaction ever submitted to Ethereum.

~~~text
Ordered transaction list
        |
        v
Index each transaction by its list position
        |
        v
Build the transaction Merkle Patricia Trie
        |
        v
Hash the trie root
        |
        v
transactionsRoot in the block header
~~~

The trie uses each transaction's list index as its key, encoded according to the trie specification. The value is the transaction's canonical serialized representation. Legacy and typed transactions enter the trie using their prescribed encodings.

The root is deterministic: the same ordered list of correctly serialized transactions produces the same root. If a transaction changes, is inserted, is removed, or is moved to a different position, the root generally changes.

### Why commit to the transaction list in the header?

A block header is a compact commitment to important block data. Instead of repeating the full transaction list inside the header, Ethereum stores the trie root. A verifier can reconstruct the transaction trie from a supplied list and compare its root with `transactionsRoot`.

This is useful for block validation, transaction inclusion proofs, light-client designs, and systems that need to establish that a transaction belongs to a particular block.

### transactionsRoot vs. other roots

| Header field | What it commits to |
|---|---|
| **`transactionsRoot`** | The ordered transaction list in this block |
| **`receiptsRoot`** | The ordered transaction receipts for this block |
| **`stateRoot`** | The resulting Ethereum world state after block processing |

Transactions are inputs to execution. Receipts describe execution outcomes. The state root commits to resulting account state. These commitments complement one another and cannot be substituted for one another.

## 7. Transactions, Receipts, and Logs

A transaction and its receipt are distinct objects. The transaction contains the signed request; the receipt is produced after execution and reports the outcome and selected execution data.

Receipt information includes:

- **Status:** success or failure for modern receipt formats.
- **Cumulative gas used:** execution gas used in the block up through this transaction.
- **Logs bloom:** a probabilistic filter summarizing log addresses and topics.
- **Logs:** events emitted during successful execution.

Events are commonly used by applications to index contract activity. For example, a token transfer may emit a `Transfer` event. Logs from an execution that reverts are discarded along with its other EVM-level effects.

The ordered receipt list is committed to through `receiptsRoot`. The receipt at each position corresponds to the transaction at the same position, but `receiptsRoot` is not derived from `transactionsRoot`.

~~~text
Transaction at index i
        |
        v
Execute against current state
        |
        +----> updated state (or reverted execution effects)
        |
        v
Receipt at index i
        |
        +----> receiptsRoot

All transactions in order
        |
        +----> transactionsRoot

Final state after block processing
        |
        +----> stateRoot
~~~

## 8. What Does a Transaction Hash Identify?

A transaction hash is the cryptographic hash of the transaction's canonical serialized representation. For a typed transaction, its typed envelope is included in that representation.

The hash identifies the signed transaction data, not its eventual execution result. A hash alone does not prove that the transaction was included in a block or succeeded. To establish inclusion, a verifier needs block context and a proof—or trusted validation—under that block's `transactionsRoot`. To establish execution outcome, the corresponding receipt and its commitment under `receiptsRoot` are relevant.

A transaction can also be visible in a node's transaction pool before inclusion. Transaction-pool presence does not guarantee inclusion: transactions may be delayed, replaced, dropped, or become invalid as chain state changes.

## 9. Gas, Fees, and Failed Transactions

Execution gas measures computational resources charged for executing a transaction. A sender sets a gas limit, and the transaction is charged for gas consumed according to protocol rules, including intrinsic costs and applicable refunds.

For EIP-1559-style transactions, the effective execution-gas price is bounded by the sender's fee caps and incorporates the block's base fee. Blob transactions add a separate blob-gas fee based on the blob fee market.

A transaction that reverts can still consume significant gas. A failed receipt therefore does not mean the sender paid no fee. The transaction remains part of the block body and contributes to the transaction trie root.

## 10. Important Distinctions

**Transaction list vs. transaction pool:** the pool (often called the mempool) contains transactions a node knows about and may consider for inclusion. The body contains transactions actually included in that block.

**Transaction vs. receipt:** the transaction is the signed request; the receipt records the execution outcome and logs.

**Transaction hash vs. transactionsRoot:** a transaction hash identifies one serialized transaction. The root commits to the complete ordered transaction list in one block.

**Transaction input data vs. blob data:** ordinary input data is part of a transaction. Blob bytes are carried separately through blob sidecars; a type 3 transaction contains versioned hashes and fee-related fields.

**Included vs. successful:** a transaction can be included in a valid block while its execution fails.

**Transaction root vs. state root:** one commits to block inputs; the other commits to the resulting world state after protocol processing.

## 11. Why Smart-Contract Developers Should Understand Transactions

Understanding transaction structure helps developers debug failed calls, estimate gas, inspect block explorers and JSON-RPC responses, index events, reason about replay protection, and understand what a block header actually commits to.

It is especially useful to understand nonce and chain ID validation, legacy gas pricing versus EIP-1559 fee caps, typed envelopes, the separate data-availability mechanism for blobs, and the authorization lists introduced by EIP-7702. Developers should check receipt status and logs rather than assuming that inclusion means success.

When implementing infrastructure or verification logic, use the relevant execution specifications and active fork configuration rather than hard-coding assumptions from an older transaction format.

## Conclusion

Transactions are the ordered execution requests inside an Ethereum block body. They describe the operations Ethereum attempts to apply to its state, while receipts record outcomes and logs. The header commits separately to the transaction list through `transactionsRoot`, to receipts through `receiptsRoot`, and to resulting world state through `stateRoot`.

Ethereum's original format has evolved through typed transactions for access lists, dynamic fees, blobs, and set-code authorizations. Despite these additions, the core model remains:

~~~text
Signed transaction
       |
       v
Validity checks and ordered execution
       |
       +----> transaction serialized in block body
       |
       +----> execution receipt
       |
       +----> resulting world-state changes
                 |
                 v
        header commitments:
        transactionsRoot / receiptsRoot / stateRoot
~~~

The most important point is that **a transaction is input to Ethereum's state transition, not proof of a successful outcome**. Its inclusion, receipt, and effect on the resulting state are related but independently verifiable parts of Ethereum's block structure.

## References

- [EIP-2718: Typed Transaction Envelope](https://eips.ethereum.org/EIPS/eip-2718)
- [EIP-155: Simple replay attack protection](https://eips.ethereum.org/EIPS/eip-155)
- [EIP-2930: Optional access lists](https://eips.ethereum.org/EIPS/eip-2930)
- [EIP-1559: Fee market change for ETH 1.0 chain](https://eips.ethereum.org/EIPS/eip-1559)
- [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844)
- [EIP-7702: Set Code for EOAs](https://eips.ethereum.org/EIPS/eip-7702)
- [Ethereum execution specifications](https://github.com/ethereum/execution-specs)
- [Ethereum Execution APIs — Transaction schema](https://github.com/ethereum/execution-apis/blob/main/src/schemas/transaction.yaml)
- [Ethereum Yellow Paper](https://github.com/ethereum/yellowpaper)

*Last reviewed: October 12, 2026.*
