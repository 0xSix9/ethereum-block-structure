# Ommers Hash

## Definition

**Ommers Hash** (also known as **Uncles Hash**) is a field in the Ethereum block header that stores the hash of the list of "ommer" blocks — blocks that were mined validly but did not become part of the canonical chain. This field was central to Ethereum's original Proof-of-Work consensus mechanism.

- **Data type:** 32 bytes (Keccak-256 hash)
- **Position:** Second field in the block header
- **Present since:** The Genesis block
- **Status:** Deprecated after the Merge — always set to a fixed constant value

## Role in the Block

The role of Ommers Hash is to **commit to a list of "sibling" blocks** that were valid competitors to earlier blocks in the chain but lost the race to be included. Under Proof-of-Work, it wasn't unusual for two miners to solve a valid block at nearly the same time; only one could become part of the canonical chain, but the "losing" block (the ommer) wasn't necessarily worthless — referencing it inside a later block allowed the ommer's miner to still receive a partial reward.

After Ethereum's transition to Proof-of-Stake (the Merge), ommer blocks no longer exist, since block production works differently under a validator-based system. As a result, this field no longer has any real function — it is preserved for structural and backward-compatibility reasons and is always set to a constant value.

## How It Works (Pre-Merge, Proof-of-Work)

Each block could reference up to two ommer blocks. Ommers Hash was calculated as:

```
Block.OmmersHash = Hash(RLP-encoded list of ommer block headers)
```

If a block included no ommers, this field held the hash of an empty list.

## Why This Field Mattered (Historically)

### 1. Rewarding "Almost-Winning" Miners

Mining is probabilistic, and it was common for two miners to find valid blocks within seconds of each other. Ommers Hash allowed the network to acknowledge blocks that were valid but excluded from the main chain, giving their miners a smaller reward instead of nothing.

### 2. Improving Network Security

By rewarding near-miss blocks, Ethereum reduced the incentive for miners to abandon smaller/weaker chains just because their block wasn't selected — this helped keep mining more decentralized and discouraged large mining pools from dominating block production.

### 3. Reducing Wasted Work

Without ommer rewards, all the computational effort spent by "losing" miners would have been completely wasted. Ommers Hash gave partial credit to that work.

## Current Status (Post-Merge)

Since the Merge (September 2022), Ethereum uses Proof-of-Stake, where validators are selected deterministically to propose blocks — there's no more competitive mining, and therefore no more ommer blocks. The Ommers Hash field is still present in the block header for backward compatibility, but it is always set to the fixed value:

```
0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d4934
```

(This is the Keccak-256 hash of an empty list — `RLP([])`.)

## Relationship to Other Fields

Ommers Hash historically worked alongside the block reward logic and Fee Recipient field, since ommer miners received rewards routed through the block that referenced them. Today, it exists purely as a structural remnant next to fields like **Nonce** and **Difficulty**, which are similarly frozen at constant values after the Merge.

## References

- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Ethereum execution-specs (GitHub)](https://github.com/ethereum/execution-specs)
