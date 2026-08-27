# Ethereum’s `gasUsed`: How Much Computation a Block Actually Consumed

When you look at the structure of an Ethereum block header, you will find a field called **`gasUsed`**.

This field tells us something very simple but important:

> **`gasUsed` tells us the total amount of gas actually consumed by all transactions in a block.**

To understand it properly, we should first distinguish it from `gasLimit`.

## `gasLimit` vs `gasUsed`

These two fields are closely related, but they mean different things.

**`gasLimit`** tells us the maximum amount of gas the block is allowed to consume.

**`gasUsed`** tells us how much gas the transactions in the block actually consumed.

Think of it like a container:

**`gasLimit` = maximum capacity**

**`gasUsed` = how much of that capacity was actually used**

For example:

```text
gasLimit: 30,000,000
gasUsed:  21,000,000
```

This means the block had a maximum capacity of 30 million gas, but its transactions actually consumed 21 million gas.

## How Is `gasUsed` Calculated?

Every transaction consumes some amount of gas when it is executed.

Imagine a block contains three transactions:

```text
Transaction 1 → 21,000 gas
Transaction 2 → 100,000 gas
Transaction 3 → 500,000 gas
```

The block's total gas used would be:

```text
21,000 + 100,000 + 500,000
= 621,000 gas
```

Therefore:

```text
gasUsed = 621,000
```

The value represents the total gas consumed by all transactions included in that block.

## `gasUsed` Cannot Be Greater Than `gasLimit`

One of the fundamental rules is:

> **`gasUsed` must be less than or equal to `gasLimit`.**

For example:

```text
gasLimit: 30,000,000
gasUsed:  25,000,000
```

This is possible.

But:

```text
gasLimit: 30,000,000
gasUsed:  35,000,000
```

would not be valid.

The block cannot contain more executed gas than its gas limit allows.

So you can remember:

**`gasUsed ≤ gasLimit`**

## Why Is `gasUsed` Important?

`gasUsed` tells us how much computational work was actually executed in a block.

This information is useful for understanding:

* How much of the block's capacity was used
* How busy the network was
* How much computation transactions required
* How Ethereum adjusts its base fee under EIP-1559

It therefore provides an important measurement of block activity.

## `gasUsed` Does Not Mean ETH Spent

This is a very important distinction.

**Gas is not the same thing as ETH.**

`gasUsed` is a measurement of computational work.

The amount of ETH paid for gas depends on the gas price.

For example:

```text
gasUsed = 100,000
```

does not tell you how much ETH was paid.

To calculate the transaction fee, you also need the applicable gas price.

Conceptually:

**Gas used × Gas price = Fee paid**

For a modern EIP-1559 transaction, the exact fee calculation also involves the **base fee** and the transaction's **priority fee (tip)**.

So:

> **`gasUsed` tells us how much computational gas was consumed, not directly how much ETH was spent.**

## `gasUsed` and EIP-1559

After Ethereum introduced **EIP-1559**, `gasUsed` became especially important because Ethereum uses block gas usage when adjusting the **base fee**.

The protocol has a target level of gas usage for blocks.

When blocks use more gas than the target, the base fee increases.

When blocks use less gas than the target, the base fee decreases.

Conceptually:

**High gas usage**

↓

**Base fee increases**

**Low gas usage**

↓

**Base fee decreases**

This allows Ethereum's base fee to respond to network demand.

## Gas Target vs Gas Limit

With EIP-1559, it is useful to understand another concept: the **gas target**.

The target is lower than the maximum gas limit.

For example, conceptually:

```text
gasLimit: 30,000,000
gas target: 15,000,000
```

If:

```text
gasUsed > gas target
```

the base fee tends to increase.

If:

```text
gasUsed < gas target
```

the base fee tends to decrease.

So `gasUsed` is not only a measurement of block activity; it also participates in Ethereum's mechanism for adjusting the base fee.

## `gasUsed` in the Block Header

`gasUsed` is part of the execution-layer block header.

When you inspect an Ethereum block, you may see something like:

```text
gasLimit: 30,000,000
gasUsed:  24,500,000
```

You can immediately understand:

> **This block was allowed to use up to 30 million gas, and its transactions actually consumed 24.5 million gas.**

The difference:

```text
30,000,000 - 24,500,000
= 5,500,000 gas
```

represents unused block gas capacity.

## Block `gasUsed` vs Transaction `gasUsed`

Another important distinction is that gas usage can be discussed at two different levels.

A **transaction** has its own gas usage.

A **block** has a total gas usage.

For example:

```text
Transaction A → 21,000 gas
Transaction B → 80,000 gas
Transaction C → 150,000 gas
```

The block's total is:

```text
gasUsed = 251,000 gas
```

So:

**Transaction gas used → Gas consumed by one transaction**

**Block `gasUsed` → Total gas consumed by all transactions in the block**

## What Happens If a Transaction Runs Out of Gas?

A transaction can consume gas during execution and eventually run out of its available gas.

Even if the transaction execution fails, the gas consumed by its execution still counts toward the block's gas usage.

This is important because gas is used to pay for computation regardless of whether the transaction ultimately succeeds.

Therefore, a failed transaction is not necessarily a transaction that used zero gas.

## A Simple Example

Imagine a block has:

```text
gasLimit = 30,000,000
```

And it contains these transactions:

```text
Tx 1 → 5,000,000 gas
Tx 2 → 8,000,000 gas
Tx 3 → 7,000,000 gas
```

The total is:

```text
5,000,000
+ 8,000,000
+ 7,000,000
----------------
20,000,000 gas
```

Therefore:

```text
gasUsed = 20,000,000
```

And:

```text
gasUsed < gasLimit
20,000,000 < 30,000,000
```

So the block used about two-thirds of its available gas capacity.

## Why Developers Care About `gasUsed`

For developers and blockchain applications, `gasUsed` can provide useful information about network activity and transaction execution.

For example, when analyzing blocks, you can compare:

```text
gasLimit
```

with:

```text
gasUsed
```

to understand how much computational capacity was actually consumed.

If blocks repeatedly have gas usage close to their limit, it generally indicates that available block capacity is being heavily utilized.

## The Most Important Idea

You can remember the entire concept with one simple relationship:

> **`gasLimit` tells us how much gas a block can use, while `gasUsed` tells us how much gas the block actually used.**

For example:

```text
gasLimit: 30,000,000
gasUsed:  24,000,000
```

means:

**Maximum capacity → 30 million gas**

**Actual consumption → 24 million gas**

And the most important rule is:

> **`gasUsed` can never exceed the block's `gasLimit`.**

So when you inspect an Ethereum block and see `gasUsed`, think:

**“This is the total computational gas actually consumed by the transactions in this block.”**

