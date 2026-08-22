# What Is the State Root in an Ethereum Block?
![State Root](https://github.com/0xSix9/ethereum-block-structure/blob/2f11d5d1e2d7f96dd0cc9f0bebff4cf3ad8ed37c/images/state-root.png)
The **`stateRoot`** is one of the most important fields in the **Ethereum block header**. It is a 32-byte Keccak-256 hash that represents the root of Ethereum’s global **state trie** after all transactions and other state-changing operations in the block have been executed.

## What Is Ethereum’s State?

The Ethereum state represents the current condition of the network. It contains information such as:

* Account balances
* Account nonces
* Smart contract code
* Smart contract storage

Ethereum organizes this information using a **Merkle Patricia Trie (MPT)**. The root of this trie acts as a cryptographic summary of the entire state.

You can think of it like this:

**Ethereum State → State Trie → Root Hash → `stateRoot`**

## How Is the State Root Created?

When a block contains transactions, Ethereum executes those transactions through the EVM.

For example:

1. Alice sends 1 ETH to Bob.
2. Ethereum executes the transaction.
3. Alice's balance decreases.
4. Bob's balance increases.
5. The state trie is updated with these changes.
6. A new root hash is calculated from the updated trie.
7. This root hash is stored in the block header as **`stateRoot`**.

Therefore, the `stateRoot` represents the **state of Ethereum after processing the block**.

## Why Is the State Root Important?

The important property of a Merkle-based structure is that changing even a small piece of data changes the resulting root hash.

For example, if Alice's balance changes:

```text
Alice: 10 ETH → 9 ETH
```

this change propagates through the trie and ultimately produces a different root hash.

Therefore:

```text
Any state change
      ↓
State Trie changes
      ↓
Root Hash changes
      ↓
stateRoot changes
```

This allows Ethereum clients to verify that the resulting state is exactly what it should be. A node can execute the transactions in a block, calculate the resulting state root, and compare it with the `stateRoot` recorded in the block header. If they do not match, something is wrong with the block execution or state calculation.

## `stateRoot` vs `transactionsRoot`

These two fields are easy to confuse, but they represent different things:

**`transactionsRoot`** represents the transactions included in the block.

**`stateRoot`** represents the resulting global Ethereum state **after those transactions have been executed**.

For example:

```text
Transactions
     ↓
Execute transactions in EVM
     ↓
State changes
     ↓
New State Trie
     ↓
stateRoot
```

So, `transactionsRoot` answers **"Which transactions are in this block?"**, while `stateRoot` answers **"What is the resulting state after executing this block?"**

## A Simple Analogy

Imagine Ethereum as a huge Excel spreadsheet containing the balances and data of every account and smart contract.

The entire spreadsheet is too large to put inside every block.

Instead, Ethereum calculates a cryptographic fingerprint of the entire current state.

That fingerprint is the **`stateRoot`**.

If even one important piece of the state changes, the fingerprint changes as well.

## Conclusion

The **`stateRoot` is a cryptographic commitment to Ethereum's global state after a block has been executed**. It allows Ethereum clients to verify that the state produced by executing a block matches the state committed to by the block header.

In one sentence:

> **`stateRoot` is the Keccak-256 root hash of Ethereum's state trie after executing a block, serving as a cryptographic commitment to the resulting global state.**

