# Understanding Blob Transactions in Ethereum

![Ethereum blob gas usage](../images/blob-gas-used.jpeg)

## Introduction

A **blob transaction** is an Ethereum transaction type introduced by [EIP-4844](https://eips.ethereum.org/EIPS/eip-4844), commonly called **Type 3**. It lets a transaction reference large amounts of data intended mainly for Layer 2 rollups, while keeping that data outside ordinary EVM execution.

Before blobs, rollups commonly published their data through regular transaction calldata. Calldata competes with regular execution for block gas and can be expensive when a rollup needs to publish a lot of data. EIP-4844 introduced a separate data-availability resource and its own fee market.

> **The key distinction:** a blob transaction commits to blob data, but the EVM cannot read the blob bytes directly. The transaction contains versioned hashes that bind it to the blobs; the blob data and its proof material are propagated through Ethereum's blob data-availability mechanism.

Blob transactions are part of Ethereum's scaling roadmap because rollups can use Ethereum to make data available without requiring every byte to be executed by the EVM as calldata.

## 1. What Is a Blob?

A **blob** is a large, fixed-size data object designed for Ethereum's data-availability mechanism. In the context of rollups, it commonly carries compressed transaction data or other information needed by the Layer 2 system.

Blobs are designed for data that must be made available to network participants but does not need to be read as ordinary EVM input during L1 execution. Ethereum checks cryptographic commitments and proofs associated with the blobs, while the rollup's own software interprets the underlying data.

This separates two jobs:

- **Execution:** run transactions and update Ethereum's state.
- **Data availability (DA):** make rollup data available so participants can retrieve it and verify the rollup's claims.

A blob is not a smart contract, a transaction by itself, or a separate block. A blob transaction is the signed transaction that references one or more blobs.

## 2. Why Did Ethereum Introduce Blob Transactions?

Rollups process transactions on Layer 2 and use Ethereum Layer 1 for settlement and data availability. For a rollup to be independently verified, the data needed to reconstruct or validate its state transitions must be available according to the rollup's design.

Historically, much rollup data was posted as calldata in ordinary Ethereum transactions. Calldata is directly available to EVM execution and remains a valid choice, but it uses the normal execution gas market.

EIP-4844 introduced a separate fee market for blob data. This lets Ethereum price data availability separately from ordinary EVM computation.

~~~text
Before blobs:
Rollup data
    |
    v
Regular L1 transaction calldata
    |
    v
Execution gas fee market

With blobs:
Rollup data
    |
    v
Blob transaction + blob sidecar
    |
    +----> normal execution gas fee market
    |
    +----> separate blob gas fee market
~~~

Blobs were designed as a step toward future data-availability scaling, not as a replacement for all calldata. Contracts can still use calldata when their data must be available to EVM execution.

## 3. Where Do Blob Transactions Appear in a Block?

A Type 3 transaction is part of the block's ordered transaction list in the **execution payload body**. Its signed transaction payload includes the fields required to identify and price its blobs, including the list of blob versioned hashes.

The actual blob bytes are not embedded as ordinary transaction input bytes in the execution block body. They are distributed with associated commitments and proofs through Ethereum's blob data-availability networking and consensus-layer structures.

~~~text
Ethereum execution block
├── Header
│   ├── transactionsRoot
│   ├── blobGasUsed
│   ├── excessBlobGas
│   └── other header fields
└── Body
    ├── Transactions
    │   └── Type 3 blob transaction
    │       ├── ordinary transaction fields
    │       ├── maxFeePerBlobGas
    │       └── blobVersionedHashes
    ├── Ommers = [] (post-Merge)
    └── Withdrawals (since Shanghai)

Associated blob data-availability material
├── Blob data
├── KZG commitments
└── Proof material (format depends on the protocol fork)
~~~

The diagram is conceptual: it distinguishes the execution block body from the blob data-availability material associated with the block. The transaction is included in the execution transaction list; the blob bytes are handled separately.

This distinction explains why an execution block explorer can show a blob transaction's hashes and fee fields without displaying the blob contents as normal transaction input.

## 4. The Type 3 Transaction Fields

EIP-4844 defines blob transactions as a typed transaction under the envelope mechanism introduced by EIP-2718. The transaction type identifier is **0x03**.

Its payload includes the following fields:

| Field | Meaning |
|---|---|
| **chainId** | Identifies the chain for which the transaction is valid |
| **nonce** | Sender account's transaction sequence number |
| **maxPriorityFeePerGas** | Maximum priority fee per unit of execution gas |
| **maxFeePerGas** | Maximum total price per unit of execution gas |
| **gasLimit** | Maximum execution gas the sender allows |
| **to** | Destination address; it must not be null |
| **value** | ETH value transferred by the transaction |
| **data** | Ordinary transaction input data, which the EVM can access |
| **accessList** | Optional access-list entries as defined by EIP-2930 |
| **maxFeePerBlobGas** | Maximum blob-gas price the sender is willing to pay |
| **blobVersionedHashes** | Versioned hashes for the blobs associated with the transaction |
| **yParity, r, s** | Signature values |

The encoded field order is defined by EIP-4844. Implementations must follow the exact schema instead of assuming that Type 3 has the same payload structure as a Type 2 dynamic-fee transaction.

### Why must to be present?

Unlike ordinary transactions, a blob transaction cannot be a contract-creation transaction. Its destination must be a 20-byte address. This is a protocol constraint in EIP-4844.

That does not prevent a blob transaction from calling a rollup contract or another smart contract. It means that contract deployment itself cannot be performed by a Type 3 transaction.

## 5. What Are Blob Versioned Hashes?

The **blobVersionedHashes** field is a list of 32-byte versioned hashes. Each entry identifies a blob commitment using a version byte and a hash-derived value. For the KZG commitment scheme introduced by EIP-4844, the version byte identifies that commitment version.

Conceptually, the commitment is hashed and converted into a versioned hash:

~~~text
Blob data
    |
    v
KZG commitment
    |
    v
SHA-256 commitment hash
    |
    v
Replace the first byte with the KZG version byte
    |
    v
blobVersionedHash
~~~

More precisely, the versioned hash is formed by setting its first byte to the KZG version constant and taking the remaining bytes from the SHA-256 hash of the commitment. The versioned hash lets the transaction refer to the commitment in a compact, fixed-length form.

The transaction does **not** put the complete blob data inside the versioned hash field. Nor is the versioned hash the blob data itself. It is a commitment identifier that can be checked against the commitment supplied with the blob.

A transaction may reference multiple blobs. The number of versioned hashes must match the number of corresponding blobs and commitment/proof entries in the associated blob data.

## 6. How Do Blob Transactions and Sidecars Work Together?

A blob transaction has a normal signed transaction representation and a network representation that can be accompanied by blob data.

Under EIP-4844's original networking design, the associated blob sidecar material includes:

- **Blobs:** the raw blob data.
- **KZG commitments:** cryptographic commitments to each blob.
- **Proofs:** proof material that allows a node to verify the data against its commitment.

The execution transaction includes the versioned hashes, while the sidecar carries the large data and verification material. Nodes verify that the number of blobs, commitments, proofs, and versioned hashes agrees and that the commitments and proofs are valid.

~~~mermaid
flowchart TD
    A["Rollup constructs data"] --> B["Encode data into blob(s)"]
    B --> C["Create KZG commitment(s) and proof material"]
    C --> D["Derive blobVersionedHashes"]
    D --> E["Sign Type 3 transaction"]
    C --> F["Propagate blob data-availability material"]
    E --> G["Transaction included in execution block"]
    F --> H["Nodes check blob data and commitment proofs"]
    G --> I["Execution block commits to transaction list"]
    H --> J["Blob data is made available under protocol rules"]
~~~

This illustrates the main relationship, not every networking message or consensus validation step. Transaction propagation and blob propagation use protocol-defined mechanisms, and a node validates the transaction and its associated blob data before accepting them under the relevant rules.

## 7. Blob Gas and the Separate Fee Market

Blob transactions pay **two kinds of fees**:

1. **Execution gas fees**, covering ordinary transaction processing and EVM execution.
2. **Blob gas fees**, covering the use of Ethereum's blob data-availability capacity.

The regular execution fee fields are similar to those of a Type 2 transaction. The separate field **maxFeePerBlobGas** is the maximum blob-gas price the sender accepts.

The actual blob fee is based on the blob base fee for the block, subject to the sender's fee cap. A transaction must provide a sufficient blob-fee cap to be valid for inclusion under the applicable rules.

~~~text
Total transaction cost
    =
execution gas used × effective execution gas price
    +
blob gas used × applicable blob gas price
~~~

This is a simplified cost model; the final amount follows the transaction and protocol rules. The sender does not necessarily pay the maximum fee caps specified in the transaction.

The blob base fee follows its own update mechanism based on blob demand. Therefore, execution gas can be relatively cheap while blob gas is expensive, or vice versa. The two resource markets are related to the same block but price different types of resource consumption.

![Blob gas used](../images/blob-gas-used.jpeg)

## 8. blobGasUsed and excessBlobGas

The execution block header includes two blob-related fields:

- **blobGasUsed:** the total amount of blob gas consumed by the blobs included in the block.
- **excessBlobGas:** a value that tracks blob-gas demand relative to the protocol's target and feeds into the blob base-fee calculation.

These are block-level accounting fields. They are not the same as the transaction's **maxFeePerBlobGas**, which is a sender-provided price cap.

~~~text
Individual Type 3 transaction
    ├── blobVersionedHashes
    └── maxFeePerBlobGas

Block header
    ├── blobGasUsed
    └── excessBlobGas
            |
            v
     Blob base-fee calculation
~~~

The target, maximum blob capacity, and related parameters are protocol settings that can change with Ethereum upgrades. Avoid hard-coding assumptions from a past fork; use the rules applicable to the block's fork.

For more detail, see the repository articles on [blobGasUsed](../header/blob-gas-used.md) and [excessBlobGas](../header/excess-blob-gas.md).

## 9. Can the EVM Read Blob Data?

**No. The EVM cannot directly read the blob bytes.** EIP-4844 intentionally keeps the blob data outside ordinary EVM execution so the data can be priced and handled differently from execution inputs.

The EVM can access a transaction's blob versioned hashes through the **BLOBHASH** opcode. The **BLOBBASEFEE** opcode exposes the current block's blob base fee. These operations expose identifiers or fee information, not the blob contents.

That means a smart contract cannot simply load the raw blob bytes from a Type 3 transaction and parse them during execution. An application that needs to use blob contents must design a verification or communication path appropriate to its system. For rollups, off-chain or Layer 2 software processes the data, while L1 contracts can verify the commitments or proofs available to them.

Do not confuse a blob's **availability** with its **direct accessibility from the EVM**. The network can make blob data available for retrieval while contracts remain unable to read the blob bytes as normal calldata.

## 10. Data Availability and Blob Data Retention

Blob data is intended for data availability, not as a permanently stored execution-state object. Under Ethereum's protocol, nodes are required to serve or retain blob data according to specified availability and retention rules; that period is limited compared with the long-term history of execution transactions.

This gives rollups an important responsibility: they must design their own data retrieval, archiving, and reconstruction strategy. Long-term users and indexers may need archival infrastructure if they need blob contents after ordinary network retention windows have elapsed.

Data availability means that participants have a way to obtain the data within the protocol's availability model. It does not mean that every execution client stores all blob data forever.

### PeerDAS and the distribution of blob data

Ethereum's data-availability mechanism can evolve independently of the Type 3 transaction's high-level purpose. [EIP-7594 (PeerDAS)](https://eips.ethereum.org/EIPS/eip-7594) describes peer data-availability sampling: blob data is extended with erasure coding and distributed in smaller cells organized into columns. Nodes sample and custody assigned portions rather than every node needing to receive every full blob in the same way.

This is a data distribution and availability mechanism. It does not turn blobs into calldata, and it does not make the blob bytes directly readable by EVM bytecode. The precise sidecar and proof formats depend on the active protocol fork; modern specifications may refer to cell proofs rather than only the original one-proof-per-blob representation.

## 11. Blob Transactions vs. Calldata

Blob data and calldata both let rollups publish data to Ethereum, but they are not interchangeable.

| Property | Calldata | Blob data |
|---|---|---|
| Where it is carried | In ordinary transaction input data | Through the blob data-availability mechanism, referenced by a Type 3 transaction |
| Readable by the EVM | Yes, as transaction input data | No, only the versioned hash is accessible through the relevant EVM opcode |
| Fee market | Ordinary execution gas | Separate blob gas, in addition to execution gas for the transaction |
| Intended use | General transaction inputs and data required by execution | Large data-availability payloads, especially rollup data |
| Long-term availability | Part of normal execution transaction history, subject to data access choices | Blob contents are subject to a limited protocol retention period and may need separate archives |

Blob transactions do not eliminate calldata. Developers should choose the data path that matches their application's execution and data-availability requirements.

## 12. How Does transactionsRoot Commit to a Blob Transaction?

The Type 3 transaction appears in the block's ordered transaction list, and that list is committed by the header's **transactionsRoot**.

The transaction's canonical typed encoding contains its signed payload, including **blobVersionedHashes** and fee fields. The blob contents and associated sidecar data are not inserted as raw bytes into the execution transaction trie as though they were ordinary transaction input.

This leads to two different verification questions:

- **Was this blob transaction included in the execution block?** Verify the transaction against the block's transactionsRoot or trust a correctly validating execution client.
- **Does the associated blob data match the transaction's commitments, and was it available under the protocol rules?** Check the versioned hashes, KZG commitments/proofs or fork-specific data-availability proofs, and the applicable consensus-layer rules.

The transaction hash identifies the signed transaction encoding. It does not by itself prove that the associated blob bytes remain retrievable indefinitely.

## 13. How Developers Can Inspect Blob Transactions

When inspecting a Type 3 transaction through JSON-RPC or an execution API, useful fields include:

- **type:** expected to be 0x3 for a blob transaction.
- **maxFeePerBlobGas:** sender's cap for the blob-gas price.
- **blobVersionedHashes:** the versioned commitments associated with the transaction.
- **blobGasUsed** and **blobGasPrice:** available in relevant receipt or transaction API representations, depending on the interface.
- **Block header fields:** especially blobGasUsed and excessBlobGas.

A standard transaction object does not necessarily return the complete raw blob data as a property of the transaction itself. Blob contents and proofs are handled through separate data-availability mechanisms and APIs.

For indexers and infrastructure developers:

- Parse Type 3 fields using the official execution API schema.
- Distinguish the execution transaction from the blob sidecar.
- Do not interpret **maxFeePerBlobGas** as the actual fee paid.
- Do not assume the current blob capacity or target is a permanent protocol constant.
- Follow the data-availability and proof format defined by the active fork.
- Plan for the limited retention of blob contents if your application needs historical data.

## 14. Key Takeaways

- A blob transaction is an EIP-4844 Type 3 transaction, introduced primarily to provide a more efficient data-availability path for rollups.
- The transaction includes blob versioned hashes and fee fields, while blob bytes are propagated as associated data-availability material.
- Blob data is not directly accessible to EVM execution; contracts can access relevant versioned hashes and the blob base fee.
- Blob transactions pay ordinary execution fees and a separate blob-gas fee.
- **blobGasUsed** and **excessBlobGas** are block header fields; **maxFeePerBlobGas** is a transaction fee cap.
- **transactionsRoot** commits to the serialized transaction, not raw blob bytes embedded in the execution block body.
- Blob retention is limited; rollups and data consumers may need separate archival strategies.
- PeerDAS changes how blob data is distributed and sampled, while the core distinction between execution data and blob data remains essential.

## References

- [EIP-4844: Shard Blob Transactions](https://eips.ethereum.org/EIPS/eip-4844) — transaction format, blob gas, versioned hashes, KZG commitments, and networking.
- [EIP-7594: PeerDAS — Peer Data Availability Sampling](https://eips.ethereum.org/EIPS/eip-7594) — data availability sampling and cell/column distribution.
- [Ethereum.org: PeerDAS](https://ethereum.org/roadmap/fusaka/peerdas/) — overview of blob scaling and data availability sampling.
- [Ethereum Execution APIs: Transaction schema](https://github.com/ethereum/execution-apis/blob/main/src/schemas/transaction.yaml) — Type 3 transaction fields and associated sidecar schema.
- [Ethereum Execution APIs](https://github.com/ethereum/execution-apis) — current JSON-RPC and execution data schemas.
- [Repository article: Blob Gas Used](../header/blob-gas-used.md)
- [Repository article: Excess Blob Gas](../header/excess-blob-gas.md)

*Last reviewed: October 12, 2026.*
