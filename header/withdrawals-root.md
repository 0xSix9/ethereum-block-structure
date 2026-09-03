# Withdrawals Root in Ethereum Block Header

The **Withdrawals Root** (`withdrawalsRoot`) is a field in the Ethereum execution block header that provides a cryptographic commitment to the validator withdrawals included in a block.

It was introduced by **EIP-4895** to support validator withdrawals after Ethereum transitioned to **Proof of Stake**.

## What Is a Withdrawal?

A withdrawal represents ETH being withdrawn from a validator's balance and transferred to an execution-layer address.

Each withdrawal contains four fields:

* `index` — the global withdrawal index
* `validatorIndex` — identifies the validator
* `address` — the recipient execution-layer address
* `amount` — the withdrawn amount, denominated in Gwei

Withdrawals are **protocol-level operations**, not normal user-signed transactions.

## How Is the Withdrawals Root Created?

The `withdrawalsRoot` is created using the execution layer's existing **Merkle-Patricia Trie (MPT)** structure.

The withdrawal data is first encoded using **RLP (Recursive Length Prefix)**. Each withdrawal is then inserted into a withdrawals trie using its index as the key.

Conceptually:

```text
Withdrawal Data
      ↓
RLP Encoding
      ↓
Withdrawal Index
      ↓
Merkle-Patricia Trie
      ↓
Keccak-256
      ↓
Withdrawals Root
```

The resulting 32-byte root is stored in the execution block header.

This creates a compact cryptographic commitment to the withdrawals contained in the block.

## Why Is It Needed?

A block may contain multiple withdrawals, but storing the entire withdrawal list in the block header would be inefficient.

Instead, Ethereum stores a single root hash.

This root commits to the withdrawal data and its structure. If a withdrawal is modified, removed, added, or its position changes, the resulting root changes.

Therefore, the `withdrawalsRoot` acts like a **cryptographic fingerprint of the block's withdrawals**.

## Can We Verify an Individual Withdrawal?

Yes.

Because the withdrawals are stored in a **Merkle-Patricia Trie**, an individual withdrawal can be verified using a **proof** containing the relevant trie nodes along the path to that withdrawal.

Conceptually:

```text
Withdrawal
     +
MPT Proof
     ↓
Reconstruct Trie Path
     ↓
Verify Root
     ↓
Withdrawals Root
```

If the calculated root matches the `withdrawalsRoot` committed in the block header, the withdrawal can be verified against that commitment.

This allows a verifier to prove that a particular withdrawal belongs to the withdrawal set committed by the block without needing to trust the source of the withdrawal data.

## Withdrawals Root vs State Root

Both fields are cryptographic roots, but they commit to different types of data.

The **State Root** commits to Ethereum's execution state, including accounts, balances, contract storage, and other state information.

The **Withdrawals Root** commits specifically to the withdrawals included in the block.

In simple terms:

```text
State Root
→ Commitment to Ethereum's execution state

Withdrawals Root
→ Commitment to the block's withdrawals
```

## Is Withdrawals Root Based on SSZ?

No—not in the original execution-layer design introduced by **EIP-4895**.

Ethereum's **Consensus Layer** uses **SSZ (Simple Serialize)** and SSZ Merkleization extensively, but the `withdrawalsRoot` in the execution block uses the execution layer's **RLP + Merkle-Patricia Trie** approach.

This distinction is important because Ethereum uses different data structures and serialization methods across its Execution and Consensus Layers.

## Why Does It Matter?

The Withdrawals Root demonstrates an important principle in Ethereum:

**A large amount of data can be represented by a single cryptographic commitment.**

Instead of placing every withdrawal into the block header, Ethereum places only the root. Anyone with the appropriate data and proof can verify that the withdrawal belongs to the committed structure.

This makes the block header compact while preserving data integrity and verifiability.

## Conclusion

The **Withdrawals Root** is a 32-byte cryptographic commitment to the validator withdrawals included in an Ethereum execution block.

In the EIP-4895 design, the process can be summarized as:

**Withdrawal Data → RLP Encoding → Merkle-Patricia Trie → Keccak-256 → Withdrawals Root**

The root is stored in the block header and acts as a **cryptographic fingerprint of the block's withdrawals**, allowing individual withdrawals to be verified against the commitment.

> **The Withdrawals Root is the cryptographic commitment that allows Ethereum to securely and efficiently represent the withdrawals included in a block.**

