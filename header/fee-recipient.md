# Fee Recipient

## Definition

**Fee Recipient** (also known as **Coinbase** in older terminology) is the field in the Ethereum block header that holds the address entitled to receive the block reward — the account that gets credited for successfully producing the block.

- **Data type:** 20 bytes (a standard Ethereum address)
- **Position:** Third field in the block header
- **Present since:** The Genesis block
- **Alternate name:** Coinbase (used in pre-Merge terminology and in many client codebases)

## Role in the Block

The role of Fee Recipient is simple but essential: it tells the network **who gets paid** for producing this specific block. Every block that gets added to the chain results in some value being transferred to this address — historically the block reward plus transaction fees under Proof-of-Work, and today primarily priority fees (tips) under Proof-of-Stake.

Without this field, there would be no way for the protocol to know which account should be credited when a block is successfully proposed and included in the chain.

## How It Works

When a block is produced, the entity responsible for building it — a miner under Proof-of-Work, or a validator under Proof-of-Stake — specifies an address in the Fee Recipient field. The Ethereum protocol then routes certain payments to that address as part of block processing:

```
Block.FeeRecipient = <address chosen by the block producer>
```

This address doesn't have to belong to the actual miner or validator directly — it's common for staking pools or mining pools to set the Fee Recipient to a pool-controlled address, which later distributes rewards to individual participants.

## What Gets Paid to the Fee Recipient?

### Under Proof-of-Work (pre-Merge)
- **Static block reward** — a fixed amount of ETH for each mined block (this changed over time due to protocol upgrades like the "Ice Age" and various reward reductions).
- **Transaction fees** — the gas fees paid by users for the transactions included in the block.
- **Ommer rewards** — a smaller reward for referencing valid-but-excluded sibling blocks.

### Under Proof-of-Stake (post-Merge)
- **Priority fees (tips)** — Since EIP-1559, transaction fees are split into a base fee (which is burned) and a priority fee (which goes to the Fee Recipient). There is no more static block reward paid through this field; validator staking rewards are instead issued on the consensus layer (the Beacon Chain), separate from the execution-layer Fee Recipient mechanism.

## Why This Field Matters

### 1. Economic Incentive for Block Production

Fee Recipient is what makes block production economically worthwhile. Whether mining or validating, someone is doing real work (computational or capital-based) to secure the network, and this field ensures they get compensated.

### 2. Enables Pooled Rewards

Because Fee Recipient can point to any address, mining pools and staking pools can aggregate rewards from many blocks into a single address, then distribute them proportionally to their participants.

### 3. Transparency of Block Rewards

Since this field is public and part of the block header, anyone can verify exactly which address benefited from any given block — making block rewards fully auditable on-chain.

## Simple Example

```
Block #19,500,000:
  Fee Recipient: 0x4838B106fCe9647Bdf1E7877BF73cE8B0BAD5f97
  ...
```

If this block includes transactions paying a total of 0.05 ETH in priority fees, that amount is credited to the address `0x4838B1...` as part of processing this block.

## Relationship to Other Fields

Fee Recipient works closely with **Base Fee Per Gas** (introduced by EIP-1559): the base fee is burned and removed from circulation, while the priority fee — the portion users pay on top of the base fee to incentivize faster inclusion — is what actually reaches the Fee Recipient.

## References

- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [Ethereum execution-specs (GitHub)](https://github.com/ethereum/execution-specs)
- [EIP-1559: Fee Market Change](https://eips.ethereum.org/EIPS/eip-1559)
