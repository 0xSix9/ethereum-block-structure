# Ethereum’s `gasLimit`: The Block Header Field That Controls Block Capacity
![gas limit](https://github.com/0xSix9/ethereum-block-structure/blob/0dcf64da98650ad2cd7b0e817e8a53bb71509865/images/gaslimit.png)
When you look at the structure of an Ethereum block header, you will find a field called **`gasLimit`**.

This field has an important role in controlling how much computation can be included inside a single Ethereum block.

The simplest way to understand it:

> **`gasLimit` defines the maximum total amount of gas that all transactions inside a block are allowed to consume.**

It helps Ethereum balance network capacity, transaction processing, and security.

## What Is Gas in Ethereum?

Before understanding `gasLimit`, we need to understand **gas**.

In Ethereum, every transaction requires computational work.

For example:

* Sending ETH requires computation.
* Calling a smart contract requires computation.
* Executing a token transfer requires computation.

Ethereum measures this computational work using a unit called **gas**.

Gas is not the cryptocurrency itself.

It is a measurement of how much computational effort an operation requires.

For example:

A simple ETH transfer consumes less gas.

A complex smart contract execution consumes more gas.

## What Is `gasLimit`?

The `gasLimit` field defines the maximum amount of gas that can be consumed by all transactions inside one block.

For example, imagine a block has:

```text
gasLimit: 30,000,000
```

This means the total gas used by all transactions inside that block cannot exceed:

```text
30,000,000 gas
```

If transactions together require more than this amount, they cannot all fit into the same block.

The remaining transactions must wait for future blocks.

## Why Does Ethereum Need a Gas Limit?

Without a block gas limit, attackers could create transactions that require enormous amounts of computation.

For example:

A malicious user could create a smart contract that consumes extremely large computational resources.

This could:

* Slow down nodes
* Increase block verification time
* Harm network performance

The `gasLimit` prevents this by placing a maximum computational boundary on every block.

So:

> **`gasLimit` protects Ethereum from unlimited computation inside a single block.**

## How Does a Block Use Gas?

Each transaction has its own gas usage.

When transactions are included in a block, their gas usage is added together.

Example:

Transaction A:

```text
Gas used: 21,000
```

Transaction B:

```text
Gas used: 100,000
```

Transaction C:

```text
Gas used: 500,000
```

Total:

```text
621,000 gas
```

The block is valid only if:

```text
Total gas used ≤ Block gasLimit
```

The block cannot exceed its gas capacity.

## `gasLimit` vs Transaction Gas Limit

This is a very important distinction.

Ethereum has two different concepts:

### Transaction Gas Limit

This is the maximum gas a user allows their transaction to consume.

Example:

```text
Transaction gas limit: 100,000
```

It means:

> "This transaction can use at most 100,000 gas."

### Block Gas Limit

This is the maximum gas that all transactions inside the block can consume together.

Example:

```text
Block gasLimit: 30,000,000
```

It means:

> "All transactions in this block combined cannot exceed 30 million gas."

So:

**Transaction gas limit → One transaction**

**Block gasLimit → Entire block**

## How Does `gasLimit` Affect Block Size?

The gas limit does not directly define the size of a block in bytes.

Instead, it limits the amount of computation inside the block.

A block with many simple transactions may contain many transactions.

A block with complex smart contract calls may contain fewer transactions.

For example:

* Simple ETH transfers use little gas.
* DeFi operations may use much more gas.

Therefore:

> **The gas limit controls computational capacity, not the exact byte size of a block.**

## How Is `gasLimit` Changed?

The Ethereum network can adjust the block gas limit.

Validators can signal changes according to Ethereum's consensus rules.

However, it cannot change instantly without limits.

The purpose is to allow the network capacity to grow or shrink gradually while maintaining stability.

## `gasLimit` After The Merge

After Ethereum moved from Proof-of-Work to Proof-of-Stake, the concept of `gasLimit` remained important.

The consensus mechanism changed, but Ethereum still needs a limit on how much computation each block can contain.

Validators now propose blocks, but those blocks must still respect the gas limit.

So:

**Before The Merge:**

Miners produced blocks and followed the gas limit.

**After The Merge:**

Validators produce blocks and follow the gas limit.

The purpose remains the same.

## Relationship Between `gasLimit` and Scalability

The block gas limit creates a balance between two goals:

### Higher gas limit

Advantages:

* More transactions can fit in each block.
* More computation can be processed.

Disadvantages:

* Nodes need more resources.
* Block verification can become slower.

### Lower gas limit

Advantages:

* Easier for nodes to process.
* Lower hardware requirements.

Disadvantages:

* Fewer transactions can be processed.

Ethereum must find a balance between scalability and decentralization.

## Where Is `gasLimit` Stored?

`gasLimit` is part of the Ethereum block header.

When a node receives a new block, it checks that:

* The block does not exceed the allowed gas limit.
* The transactions inside the block respect this limit.
* The execution results are valid.

This makes `gasLimit` part of Ethereum's block validation rules.

## Simple Example

Imagine Ethereum has a block with:

```text
gasLimit: 30,000,000
```

The transactions included are:

```text
Transaction 1 → 10,000,000 gas
Transaction 2 → 8,000,000 gas
Transaction 3 → 12,000,000 gas
```

Total:

```text
30,000,000 gas
```

The block is exactly at its limit.

But adding another transaction requiring:

```text
1,000,000 gas
```

would make the block invalid because:

```text
31,000,000 > 30,000,000
```

## The Most Important Idea

You can remember the entire concept with one sentence:

> **`gasLimit` is a field in the Ethereum block header that defines the maximum amount of computational work, measured in gas, that can be included in a single block.**

And remember:

> **Transaction gas limit controls one transaction, while block `gasLimit` controls the total capacity of the entire block.**

So when you inspect an Ethereum block and see:

```text
gasLimit: 30,000,000
```

you should understand:

**"This block can include transactions whose combined gas usage does not exceed 30 million gas."**
