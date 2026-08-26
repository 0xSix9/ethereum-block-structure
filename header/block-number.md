# Ethereum's `blockNumber`: The Identity of a Block's Position in the Chain

When you look at the structure of an Ethereum block, you will find a field called **`blockNumber`** (often written as `number` in the execution-layer block header).

It has a very simple job:

> **`blockNumber` tells us the position of a block in the Ethereum blockchain.**

Ethereum's official documentation lists `block_number` as the **number of the current block** in the execution payload header.

## What Is a Block Number?

Think of the block number as a **counter** for Ethereum blocks.

The first block of Ethereum is:

`Block 0`

The next block is:

`Block 1`

Then:

`Block 2`

Then:

`Block 3`

And so on.

So if you see:

```text
blockNumber: 20,000,000
```

you know that this block is at position **20,000,000** in Ethereum's canonical chain.

This is why block number is also commonly called **block height**.

## Why Does Ethereum Need a Block Number?

Ethereum needs a way to identify where a block sits in the chain.

The `blockNumber` provides a simple numerical reference that applications, developers, nodes, and smart contracts can use to refer to a block.

For example, instead of saying:

> "I mean that block somewhere after another block..."

you can simply say:

> "I mean block `20,000,000`."

This makes working with the blockchain much easier.

## How Does the Number Increase?

For the canonical chain, each new execution block normally has a block number one greater than its parent.

Conceptually:

**Block 100**

↓

**Block 101**

↓

**Block 102**

↓

**Block 103**

The relationship is therefore:

> **Current block number = Parent block number + 1**

The block number is not randomly generated.

It is part of the block's execution-layer data and is checked as part of block validity.

## Block Number vs Block Hash

It is important not to confuse these two.

A **block number** tells you the block's position.

A **block hash** identifies a particular block cryptographically.

For example:

```text
Block Number
    ↓
20,000,000
```

tells you **where the block is**.

While:

```text
Block Hash
    ↓
0x8a7f...c921
```

helps identify the exact block by its cryptographic hash.

You can think of it like this:

**Block number = position**

**Block hash = cryptographic identity**

## Is the Block Number Unique?

The block number itself is **not a globally unique identifier**.

Two competing blocks on different forks can have the same block number because they can both be the next block at the same height.

For example:

**Block 100**

can theoretically have two competing versions:

**Block A → number 101**

**Block B → number 101**

Both have the same block number, but they have different hashes.

This is why applications that need to identify one exact block should use the **block hash**, not the block number alone.

## Block Number and the Parent Block

Every Ethereum block contains a reference to its parent through `parentHash`.

The parent block has a lower block number, and the current block follows it in the chain.

Conceptually:

```text
Parent Block
Block #100
     |
     | parentHash
     ↓
Current Block
Block #101
```

The `parentHash` answers:

> **"Which exact block came before me?"**

The `blockNumber` answers:

> **"What is my position in the chain?"**

These two fields therefore provide very different kinds of information.

## How Can Smart Contracts Use It?

The block number is also available to the EVM.

In Solidity, a smart contract can access the current block number using:

```solidity
block.number
```

Ethereum's documentation describes `NUMBER` as an EVM instruction that provides block-header information, and Solidity exposes it as `block.number`.

This allows smart contracts to use the current block number in their logic.

For example, a contract could say:

> "This action is available only after block 1,000,000."

Or:

> "This operation expires after a certain number of blocks."

## Block Number Is Not the Same as Time

A very important point:

**Block number is not a timestamp.**

For example:

```text
block.number → 20,000,000
```

tells you the block's position.

While:

```text
timestamp → a Unix timestamp
```

tells you the block's recorded time.

They represent different things.

You should therefore think:

**Block number → Where?**

**Timestamp → When?**

## Does Block Number Determine Consensus?

No.

The block number is an identifier for the block's position; it is not what makes Ethereum's Proof-of-Stake consensus secure.

Modern Ethereum uses Proof-of-Stake consensus, and the execution payload includes `block_number` alongside other execution information such as `parent_hash`, `state_root`, `receipts_root`, and `timestamp`.

So don't think:

> "The block number is what decides which block is valid."

Instead:

> **The block number is one piece of information used to describe and validate the block's position in the chain.**

## A Simple Example

Imagine Ethereum has this sequence:

```text
Block #500
     ↓
Block #501
     ↓
Block #502
     ↓
Block #503
```

If you are looking at Block #503:

* Its `blockNumber` is `503`
* Its parent should be Block #502
* Its `parentHash` identifies the exact Block #502
* Its position is after Block #502

So the number gives you a simple way to understand where that block belongs in the chain.

## The Most Important Idea

You can remember the entire concept with one sentence:

> **`blockNumber` tells us the position or height of an Ethereum block in the blockchain.**

And remember the difference:

> **`blockNumber` tells you where the block is, while `blockHash` identifies the exact block.**

So when you inspect an Ethereum block and see:

```text
blockNumber: 20,000,000
```

you can immediately understand:

**"This block is at height 20,000,000 in the canonical chain."**

