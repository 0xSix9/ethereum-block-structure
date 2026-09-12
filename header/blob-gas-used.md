# Understanding `blobGasUsed` in the Ethereum Block Header
![blob gas used](https://github.com/0xSix9/ethereum-block-structure/blob/2a6709f30f392fbf4f08fbad98cdb09849cef007/images/blob-gas-used.jpeg)
## Introduction

Ethereum blocks contain a header with multiple fields that describe important information about the block and the state of the Ethereum network.

After the introduction of **EIP-4844**, also known as **Proto-Danksharding**, Ethereum introduced a new type of transaction data called **blobs**.

Blobs created a separate data market for Ethereum, mainly designed to reduce the cost of publishing data from Layer 2 rollups to Ethereum.

Because blobs consume a new resource that is separate from normal Ethereum gas, Ethereum needed a way to record how much blob-related resource was consumed inside each block.

This is the purpose of:

```text
blobGasUsed
```

The `blobGasUsed` field records the total amount of **blob gas consumed by all blob-carrying transactions included in a block**.

---

# What Is `blobGasUsed`?

`blobGasUsed` is a field in the Ethereum block header that represents the total amount of **blob gas used by the blob transactions contained in that block**.

Conceptually:

```text
blobGasUsed
=
Total blob gas consumed by all blobs in the block
```

Unlike the normal `gasUsed` field, `blobGasUsed` does not measure the computational execution of smart contracts.

Instead, it measures the consumption of Ethereum's separate **blob data resource**.

This distinction is extremely important.

Ethereum now effectively has two different resource markets:

```text
Execution Gas
        ↓
Smart contract execution
Transaction processing
EVM computation

Blob Gas
        ↓
Temporary blob data
Rollup data publication
Data availability for Layer 2
```

Therefore:

```text
gasUsed      → execution resource consumption
blobGasUsed  → blob data resource consumption
```

These two values represent different things.

---

# Why Did Ethereum Need Blob Gas?

Before EIP-4844, Layer 2 rollups usually published transaction data to Ethereum using **calldata**.

Calldata is permanently stored as part of Ethereum's blockchain history.

This provides strong data availability, but it is expensive.

For rollups, storing large amounts of transaction data permanently on Ethereum was unnecessary.

Rollups mainly need the data to be available long enough for the network to verify and reconstruct the rollup state.

They do not necessarily need Ethereum nodes to store that data forever.

This created the need for a new data structure:

```text
Blob
```

A blob is designed for large amounts of temporary data.

Blobs are:

* cheaper than calldata for rollup data
* not directly accessible by the EVM
* available for a limited period
* designed primarily for Layer 2 data publication

To manage this new resource, Ethereum introduced:

```text
Blob Gas
```

And to record how much of that resource was consumed inside a block:

```text
blobGasUsed
```

---

# How Does `blobGasUsed` Work?

When a block contains blob-carrying transactions, those transactions may include one or more blobs.

Each blob consumes a fixed amount of blob gas.

Therefore, the block's `blobGasUsed` value can be calculated based on the number of blobs included in the block.

Conceptually:

```text
blobGasUsed
=
Number of blobs × Blob gas per blob
```

In the current EIP-4844 design:

```text
1 Blob = 131,072 Blob Gas
```

Therefore:

```text
1 blob
=
131,072 blob gas
```

For example:

```text
Block contains 1 blob

blobGasUsed = 131,072
```

If the block contains three blobs:

```text
3 × 131,072

blobGasUsed = 393,216
```

If a block contains no blobs:

```text
blobGasUsed = 0
```

---

# Example

Imagine an Ethereum block contains the following transactions:

```text
Transaction 1
Normal ETH transfer
Blob count: 0

Transaction 2
Smart contract interaction
Blob count: 0

Transaction 3
Layer 2 rollup transaction
Blob count: 2

Transaction 4
Layer 2 rollup transaction
Blob count: 1
```

The total number of blobs is:

```text
2 + 1 = 3 blobs
```

Since each blob consumes:

```text
131,072 blob gas
```

The block header contains:

```text
blobGasUsed
=
3 × 131,072

=
393,216
```

Therefore:

```text
blobGasUsed: 393216
```

---

# `blobGasUsed` vs `gasUsed`

One of the most common sources of confusion is the difference between these two fields.

## `gasUsed`

```text
gasUsed
```

Represents the total amount of normal Ethereum execution gas consumed by transactions in the block.

It includes resources related to:

* EVM execution
* smart contract operations
* transaction execution
* computational work
* state changes

For example:

```text
gasUsed = 15,000,000
```

This means the transactions inside the block consumed a total of 15 million units of normal execution gas.

---

## `blobGasUsed`

```text
blobGasUsed
```

Represents the total blob gas consumed by blobs included in the block.

It is related to:

* blob-carrying transactions
* Layer 2 rollup data
* temporary data availability
* blob capacity

For example:

```text
blobGasUsed = 393,216
```

This represents three blobs.

---

# A Simple Comparison

```text
NORMAL ETHEREUM GAS

User Transaction
       │
       ▼
EVM Execution
       │
       ▼
gasUsed
```

```text
BLOB GAS

Rollup Transaction
       │
       ▼
Blob Data
       │
       ▼
blobGasUsed
```

These resources are related to the same block, but they belong to different resource markets.

---

# Why Isn't Blob Usage Included in Normal Gas?

Because blobs and EVM execution consume fundamentally different resources.

Normal Ethereum gas protects and prices resources such as:

```text
CPU
EVM computation
State access
Storage operations
Transaction execution
```

Blob gas is designed to price a different resource:

```text
Temporary data availability capacity
```

If blob data used the same gas market as EVM execution, large demand for rollup data could significantly interfere with smart contract transactions.

Ethereum therefore separates these resources.

Conceptually:

```text
                Ethereum Block

        ┌───────────────────────┐
        │                       │
        │   Execution Resource  │
        │                       │
        │      gasUsed          │
        │                       │
        ├───────────────────────┤
        │                       │
        │    Blob Data Resource │
        │                       │
        │    blobGasUsed        │
        │                       │
        └───────────────────────┘
```

This separation allows Ethereum to independently manage:

* execution demand
* blob data demand

---

# `blobGasUsed` and Layer 2 Rollups

The primary users of blobs are expected to be Layer 2 rollups.

A simplified rollup flow looks like this:

```text
Users
  │
  ▼
Layer 2
  │
  │ Batch transactions
  ▼
Rollup Sequencer
  │
  │ Create blob data
  ▼
Blob Transaction
  │
  ▼
Ethereum Layer 1
  │
  ├── Execution Gas
  │
  └── Blob Gas
          │
          ▼
     blobGasUsed
```

When rollups publish their transaction data using blobs, the Ethereum block records how much blob gas was consumed.

Therefore, `blobGasUsed` provides an important measurement of how much blob capacity was used in that block.

---

# `blobGasUsed` and Blob Capacity

Ethereum limits the amount of blob data that can be included in a block.

This is necessary because blobs consume network and data availability resources.

Without limits, an attacker could attempt to create extremely large amounts of blob data.

Therefore, Ethereum defines blob capacity limits.

A block can only contain a limited number of blobs.

As a result:

```text
blobGasUsed
```

cannot grow infinitely.

It represents the amount of available blob capacity that was actually consumed by the block.

Conceptually:

```text
Maximum Blob Capacity
        │
        ▼
┌───────────────────────────┐
│ ████████████████████      │
│ Used      │ Available     │
└───────────────────────────┘
        │
        ▼
   blobGasUsed
```

The more blobs included in a block, the higher the `blobGasUsed` value.

---

# `blobGasUsed` and the Blob Fee Market

Ethereum does not simply charge a fixed price for blobs.

Blob pricing is designed as a dynamic fee market.

This is important because demand for blob space can change.

For example:

```text
Low Rollup Demand
       ↓
Lower Blob Fees
```

And:

```text
High Rollup Demand
       ↓
Higher Blob Fees
```

Ethereum tracks blob demand over time using another important block header field:

```text
excessBlobGas
```

The relationship can be understood as:

```text
Current Block
│
├── blobGasUsed
│       ↓
│   How much blob resource
│   this block consumed
│
└── excessBlobGas
        ↓
    Tracks blob resource
    pressure over time
```

These two fields work together in Ethereum's blob fee mechanism.

---

# `blobGasUsed` vs `excessBlobGas`

These fields are related but represent different things.

## `blobGasUsed`

Measures:

```text
How much blob gas was used
in this block?
```

Example:

```text
3 blobs

blobGasUsed = 393,216
```

---

## `excessBlobGas`

Represents accumulated blob gas pressure relative to Ethereum's target over time.

It is used as part of the mechanism for determining the blob base fee.

Conceptually:

```text
Previous Blob Pressure
        +
Current Blob Usage
        -
Target Blob Usage
        =
New Excess Blob Gas
```

This means:

```text
blobGasUsed
```

is primarily about:

```text
Current block usage
```

While:

```text
excessBlobGas
```

is related to:

```text
Longer-term blob demand and fee adjustment
```

---

# Why Is `blobGasUsed` Stored in the Block Header?

The block header contains information that allows Ethereum clients to validate important properties of a block.

By including:

```text
blobGasUsed
```

Ethereum makes the blob resource usage part of the block's consensus-relevant information.

Ethereum clients can use this information when validating the block.

Conceptually:

```text
Ethereum Block

┌─────────────────────────────┐
│ Block Header                │
│                             │
│ gasUsed                     │
│ gasLimit                    │
│ baseFeePerGas               │
│                             │
│ blobGasUsed                 │
│ excessBlobGas               │
└─────────────────────────────┘
```

These fields help describe how Ethereum resources were used and priced.

---

# A Practical Block Example

Imagine the following simplified Ethereum block:

```text
Block Number: 20,000,000

Transactions: 150

Normal Execution Gas Used:
gasUsed = 20,000,000

Blobs:
3

Blob Gas Used:
blobGasUsed = 393,216
```

This tells us that the block consumed:

```text
20,000,000 units
```

of execution gas and:

```text
393,216 units
```

of blob gas.

These numbers should not be directly compared because they measure different resources.

For example:

```text
gasUsed ≠ blobGasUsed
```

They belong to separate accounting systems.

---

# The Bigger Picture: Ethereum's Multi-Resource Future

The introduction of `blobGasUsed` represents an important evolution in Ethereum's architecture.

Earlier versions of Ethereum primarily focused on a single major gas market:

```text
Execution Gas
```

With EIP-4844, Ethereum introduced another resource market:

```text
Blob Gas
```

This can be visualized as:

```text
                    Ethereum

          ┌──────────────────────┐
          │                      │
          │     Block Resources  │
          │                      │
          └──────────┬───────────┘
                     │
          ┌──────────┴───────────┐
          │                      │
          ▼                      ▼

   Execution Market         Blob Market

      gasUsed             blobGasUsed

      EVM                 Blob Data
      Contracts           Rollups
      Computation         Data Availability
```

This approach allows Ethereum to optimize different resources independently.

The long-term Ethereum roadmap continues moving toward an architecture where different types of resources can have different constraints and pricing mechanisms.

---

# Common Misunderstandings

## Misunderstanding 1: `blobGasUsed` is EVM gas

Incorrect.

```text
blobGasUsed
```

does not represent EVM execution.

It represents blob resource consumption.

---

## Misunderstanding 2: Blobs are smart contract storage

Incorrect.

Blobs are not permanent Ethereum storage.

They are temporary data objects designed primarily for data availability.

---

## Misunderstanding 3: `blobGasUsed` represents the number of transactions

Incorrect.

A single blob transaction may contain multiple blobs.

Therefore:

```text
Number of Transactions
≠
blobGasUsed
```

The value depends on the number of blobs.

---

## Misunderstanding 4: `blobGasUsed` and `gasUsed` should be added together

Incorrect.

They measure different resources.

Conceptually:

```text
gasUsed
+
blobGasUsed
```

does not represent a meaningful total amount of Ethereum gas.

They belong to separate gas markets.

---

# Final Summary

The `blobGasUsed` field is one of the new Ethereum block header fields introduced with **EIP-4844 (Proto-Danksharding)**.

Its purpose is simple:

```text
blobGasUsed
=
The total amount of blob gas consumed by blobs included in the current block
```

It allows Ethereum to track the usage of its new blob data resource.

The key idea is:

```text
gasUsed
        ↓
Execution and EVM computation
```

while:

```text
blobGasUsed
        ↓
Blob data consumption
```

Each blob consumes a fixed amount of blob gas, so the value of `blobGasUsed` depends on the total number of blobs included in the block.

Together with:

```text
excessBlobGas
```

this field plays an important role in Ethereum's separate blob fee market.

More importantly, `blobGasUsed` represents a major architectural change in Ethereum.

Ethereum is no longer managing only one type of gas resource.

It is evolving toward a multi-resource architecture where:

```text
Execution
Storage
Data Availability
```

can be managed and priced according to their different requirements.

For Layer 2 rollups, this evolution is particularly important.

Blobs provide a cheaper way to publish data to Ethereum while preserving Ethereum's role as a secure data availability and settlement layer.

And inside every block that uses this new resource, the block header records that usage through:

```text
blobGasUsed
```

A small field in the block header—but an important part of Ethereum's scaling roadmap.
