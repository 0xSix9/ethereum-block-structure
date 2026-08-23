## Transactions Root

The **Transactions Root** is a field in the Ethereum block header that contains the root hash of the **Merkle Patricia Trie (MPT)** storing all transactions included in the block.

Each transaction in a block is organized inside the transaction trie. The trie structure is then hashed, producing a single value called the **Transactions Root**. This root hash is stored in the block header.

The Transactions Root allows Ethereum clients to verify that the transactions associated with a block have not been modified. If even a small part of a transaction changes, the resulting root hash will also change.

For example:

**Transactions → Transaction Trie → Root Hash → Transactions Root**

It is important to note that the `transactions-root` represents the **transactions included in that specific block**, not the entire history of transactions on Ethereum.

### Why is Transactions Root important?

The Transactions Root provides a compact cryptographic commitment to all transactions in a block. Instead of storing every transaction directly inside the header, Ethereum stores only their root hash. This allows nodes to verify that the transactions they received correspond exactly to the transactions committed to by the block header.

