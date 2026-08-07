# Parent Hash

## Definition

**Parent Hash** is one of the most important fields in the Ethereum block header. It stores the complete hash of the previous block's header (the parent block), and it is literally the mechanism that turns individual blocks into a **chain**.

- **Data type:** 32 bytes (Keccak-256 hash)
- **Position:** First field in the block header
- **Present since:** The Genesis block (the very first Ethereum block)

## How It Works

Every block computes the hash of the entire header of the block before it, and stores that value in its own Parent Hash field:

```
Block N.ParentHash = Hash(Block N-1's Header)
```

This means each block is directly and immutably linked to the block before it. If even a single bit of the previous block's data changes, that block's hash changes too — and the Parent Hash stored in the next block will no longer match it.

## Why This Field Matters

### 1. Chain Integrity

Parent Hash is exactly the mechanism that causes blocks to form a **linear chain**, rather than existing as a scattered collection of independent data.

### 2. Tamper Detection

If someone tries to alter the contents of an old block, that block's hash changes. Since every subsequent block references the original hash, this change is immediately detectable across the network — the chain "breaks" from that point forward.

### 3. Resolving Forks

When two miners or validators produce two different blocks at nearly the same height (Block Number) at roughly the same time, the network experiences a temporary **fork**. Parent Hash lets nodes trace the exact path of each branch of the chain and determine which branch is the "heaviest" (canonical) chain to follow.

## Simple Example

Suppose block #100 is defined as follows:

```
Block #100:
  Parent Hash: 0xabc123... (hash of block #99)
  Block Number: 100
  ...
```

If even a small transaction inside block #99 changes, the entire hash of block #99 changes. As a result, the Parent Hash stored in block #100 no longer matches the actual hash of block #99. This mismatch causes block #100 to be considered **invalid** by the network.

## Relationship to Other Fields

Parent Hash doesn't work alone — together with fields like **State Root**, **Transactions Root**, and **Receipts Root**, it forms a complete integrity-proofing system, sometimes referred to as a **Merkle Chain of Trust**.

## References

- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Ethereum execution-specs (GitHub)](https://github.com/ethereum/execution-specs)
