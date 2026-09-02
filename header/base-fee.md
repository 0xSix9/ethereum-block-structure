# Base Fee Per Gas in Ethereum Block Headers

Ethereum blocks contain a block header with important information that allows nodes to verify and understand the state of the blockchain. One of the fields introduced by **EIP-1559** is `base_fee_per_gas`, which fundamentally changed how transaction fees work on Ethereum.

## What Is `base_fee_per_gas`?

`base_fee_per_gas` is the **minimum amount of ETH that must be paid per unit of gas** for a transaction to be included in a block.

In simple terms, it represents the protocol-defined price of using Ethereum's execution layer.

Unlike the transaction's priority fee, the base fee is **not chosen by the user or validator**. It is calculated automatically by the Ethereum protocol based on how much gas was used in the previous block.

The field is stored directly in the block header, meaning every block contains its own base fee value.

For example:

```text
base_fee_per_gas: 20 gwei
```

This means that transactions included in that block must effectively pay at least 20 gwei per gas toward the protocol's base fee.

---

## Why Was the Base Fee Introduced?

Before EIP-1559, Ethereum mainly used a first-price auction model. Users submitted a `gasPrice`, and validators (formerly miners) generally preferred transactions offering higher fees.

This could make transaction fees difficult to predict, especially when the network became congested.

EIP-1559 introduced a different mechanism. Instead of allowing the entire transaction fee to behave like an auction, Ethereum separates the fee into:

**Base Fee + Priority Fee**

The base fee is determined by the protocol, while the priority fee acts as a tip for the validator.

---

## How Does the Base Fee Change?

The base fee changes from block to block according to network usage.

Ethereum compares the amount of gas used in the previous block with its **target gas usage**.

If the previous block used more gas than the target, the network is experiencing higher demand, so the base fee increases.

If the previous block used less gas than the target, the base fee decreases.

Conceptually:

```text
High gas usage → Base fee increases

Low gas usage  → Base fee decreases
```

Under the current EIP-1559 mechanism, the base fee can change by up to **12.5% per block**. The target is generally half of the block gas limit, allowing blocks to temporarily grow beyond the target when demand is high.

This creates a feedback mechanism between network demand and transaction pricing.

---

## Base Fee and Transaction Fees

For a typical EIP-1559 transaction, the user specifies:

* `maxFeePerGas` — the maximum amount the user is willing to pay per gas.
* `maxPriorityFeePerGas` — the maximum tip offered to the validator.

The actual effective gas price is based on the block's base fee and the priority fee.

For example, suppose:

```text
Base fee = 20 gwei
Priority fee = 2 gwei
```

The effective price becomes:

```text
20 + 2 = 22 gwei per gas
```

If the transaction uses 21,000 gas:

```text
21,000 × 22 gwei = 462,000 gwei
```

which equals:

```text
0.000462 ETH
```

The important point is that the **20 gwei base fee is burned**, while the **2 gwei priority fee goes to the validator**.

---

## Why Is the Base Fee Burned?

One of the most important characteristics of `base_fee_per_gas` is that the ETH paid as the base fee is **burned**.

Burning means that the ETH is permanently removed from circulation rather than being transferred to the validator.

For example, if a transaction consumes:

```text
21,000 gas
```

and the base fee is:

```text
20 gwei
```

the burned amount is:

```text
21,000 × 20 gwei
= 420,000 gwei
= 0.00042 ETH
```

This mechanism can reduce Ethereum's circulating supply when the amount of ETH burned exceeds the amount of ETH issued.

EIP-1559 also removes an incentive for block producers to manipulate the base fee for their own benefit because they do not receive the base fee.

---

## Why Is It Part of the Block Header?

`base_fee_per_gas` is not simply a value used by wallets.

It is part of the block's consensus-critical data. Ethereum clients use the value when validating blocks and transactions.

The block header therefore records the base fee associated with that block.

Because the base fee affects transaction validity and fee accounting, nodes must be able to verify that the value included in a block follows the protocol's rules.

This also means that the base fee contributes to the information that defines the block and, consequently, the block hash.

---

## A Simple Example

Imagine Ethereum has these consecutive blocks:

```text
Block 100
Gas used: 95% of target
Base fee: 20 gwei
```

Because the block was heavily used, the protocol increases the base fee for the next block:

```text
Block 101
Base fee: higher than 20 gwei
```

Now imagine network activity falls:

```text
Block 102
Gas used: 40% of target
```

The protocol responds by reducing the base fee for the following block.

Therefore, the base fee continuously reacts to network demand.

---

## Base Fee vs Priority Fee

It is important not to confuse these two values.

**Base Fee**

* Determined by the Ethereum protocol
* Depends on previous block gas usage
* Required for transaction inclusion
* Burned after being paid

**Priority Fee**

* Chosen by the transaction sender
* Acts as a tip for the validator
* Used to incentivize transaction inclusion
* Paid to the validator

Together, they form the core of Ethereum's post-EIP-1559 transaction fee mechanism.

---

## Why Is `base_fee_per_gas` Important for Developers?

For Ethereum developers, understanding this field is important because transaction fees, RPC responses, wallets, and smart-contract interactions all operate within this fee mechanism.

Developers can also access the current block's base fee from Solidity using:

```solidity
block.basefee
```

This makes the block's base fee available to smart contracts when they need to reference the current protocol-defined gas price.

It can also be retrieved through Ethereum RPC methods when querying block data.

---

## Conclusion

`base_fee_per_gas` is a fundamental field in Ethereum's block header introduced with **EIP-1559**.

It represents the protocol-defined base price for gas, changes dynamically according to network demand, and is burned when transactions are executed.

The key idea is simple:

```text
Network demand
      ↓
Previous block gas usage
      ↓
Base fee adjustment
      ↓
Transaction cost
      ↓
Base fee is burned
```

Understanding `base_fee_per_gas` is therefore essential for anyone studying Ethereum's block structure, transaction fee mechanism, or smart-contract development.

