# Understanding Ommer Blocks in Ethereum

![Ethereum block body](../images/body.png)

## Introduction

An **ommer block** (often called an *uncle block* in older Ethereum material) is a valid Proof-of-Work block that was not included in Ethereum's canonical chain, but whose header could be referenced by a later canonical block. Before The Merge, Ethereum allowed a block to include a limited number of ommer headers in its body and paid protocol-defined rewards for doing so.

Ommers were a feature of Ethereum's former Proof-of-Work (PoW) block-production system. They helped recognize some of the work done by miners whose blocks lost a race to be included in the canonical chain.

**Important for modern Ethereum:** since The Merge, Ethereum uses Proof of Stake (PoS). Current execution blocks must contain an empty ommers list. The ommers body field remains in the execution-block data structure for compatibility, but it no longer contains ommer headers.

## 1. Why Did Ommer Blocks Exist?

In a Proof-of-Work network, multiple miners can discover valid blocks at nearly the same time. Because blocks take time to propagate through the peer-to-peer network, different nodes may temporarily learn about different valid next blocks.

For example, two miners may produce blocks that both extend the same parent. The canonical chain's fork-choice process eventually favors one branch. The block on the other branch is not part of the canonical chain, even though it may have been valid when mined.

Historically, Ethereum called a referenced non-canonical PoW block an **ommer**. The term is gender-neutral and describes a sibling in the family tree of blocks; many older explanations use “uncle block.”

Conceptual illustration:

    Parent block P
       ├── Block A ── Canonical child C
       └── Block B ── loses the chain race
                         ↑
                         └── C may reference B's header as an ommer

The diagram is conceptual. The ommer header could be included only if it met protocol rules, including ancestry distance and non-duplicate constraints.

Without any ommer mechanism, a miner whose block lost this race could receive no direct block reward for that work. Ommer inclusion provided a partial reward path and modestly reduced the disadvantage of mining a valid block that did not become canonical.

## 2. Where Did Ommers Appear in an Ethereum Block?

Before The Merge, an execution block body conceptually contained:

    Ethereum PoW Block
    ├── Header
    ├── Transactions
    └── Ommers
        ├── Ommer header 1 (optional)
        └── Ommer header 2 (optional)

A PoW block could include **zero, one, or two ommer headers**. It did not embed a complete second block for each ommer. The body included the ommer headers, not each ommer's full transaction list and execution result.

The canonical block header also contained the ommersHash field, which committed to the ommer-header list. The body list and its header commitment had to agree under the block's validation rules.

After The Merge, the execution-block format retained the field but its ommer list became empty:

    Modern Ethereum Execution Block
    ├── Header
    │   └── ommersHash = hash of the empty list
    └── Body
        ├── Transactions
        ├── Ommers = []
        └── Withdrawals (since Shanghai)

The exact execution payload has other fields defined by the active fork. This diagram highlights only the ommer-related portion.

## 3. Which Blocks Could Be Included as Ommers?

The historical PoW rules placed strict limits on ommer inclusion. In simplified terms, an eligible ommer had to:

- Have a valid block header under the applicable PoW-era rules.
- Be a sibling of a block on the including block's recent ancestor chain, within the protocol's allowed ancestry window.
- Fit the historical maximum distance of six blocks in the relevant height relationship.
- Not be the including block itself or one of its ancestors, and not violate the rules against duplicate or repeated ommer inclusion.
- Respect the limit of at most two ommers in any one block.

These conditions meant that not every stale or orphaned block could be included as an ommer. The block had to satisfy the defined relationship to the canonical chain and all validity checks.

Ommer inclusion did **not** make the ommer itself part of the canonical chain. It allowed the canonical block to reference the ommer header and apply the historical reward rules.

## 4. How Did Ommer Rewards Work?

Before The Merge, ommers had an economic role in Ethereum's PoW incentive model.

### Reward for the ommer's miner

A qualifying ommer miner could receive a partial block reward. The amount depended on the ommer's height relative to the including block: more recent eligible ommers received a larger reward, and the reward declined with distance.

The base block reward itself changed across Ethereum's historical upgrades, so a reward amount quoted for one era should not be assumed to apply to every PoW block.

### Reward for the including block's miner

The miner of the block that included an ommer also received an inclusion reward. Under the historical rules, this was an additional **1/32 of the base block reward per included ommer**. A block could include at most two ommers, so the maximum inclusion bonus from this rule was 2/32 of the base reward.

These were historical execution-layer PoW rewards. They are not current Ethereum validator rewards.

## 5. What Changed at The Merge?

The Merge replaced Ethereum's PoW consensus mechanism with PoS consensus driven by the Beacon Chain. EIP-3675 specifies that from the transition to PoS:

- The ommers list must be empty.
- The ommersHash header field is fixed to a constant value.
- PoW-era ommer validation is removed.
- Block and ommer rewards under the old execution-layer PoW rules are discontinued.

In PoS Ethereum, validators propose and attest to blocks according to consensus-layer rules. There is no mining race in the old PoW sense and no current ommer-inclusion reward mechanism.

A proposed block can still fail to become part of the canonical chain, but such events are handled by PoS fork-choice and consensus mechanisms rather than by adding ommer headers to execution blocks.

## 6. The Constant ommersHash

Although modern blocks contain no ommers, the header retains ommersHash. Under EIP-3675, its value is fixed to the hash of the RLP-encoded empty list:

    ommers = []
    RLP([]) = 0xc0

    ommersHash =
    0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347

The constant is calculated as:

    Keccak256(RLP([]))

This is **not** the hash of an arbitrary empty byte string. It is the Keccak-256 hash of the specific RLP encoding of an empty list, 0xc0.

For developers inspecting modern execution block headers, the important point is that ommersHash remains present for structural continuity but no longer varies according to included ommers.

## 7. Ommers Are Not the Same as Transactions

An ommer was a block-header reference, not a transaction bundle that gets automatically executed by the canonical block.

Including an ommer header did not execute the transactions that had appeared in that non-canonical block. A transaction from a non-canonical branch might independently be included later in a canonical block, provided it remained valid under the canonical chain's state and transaction rules.

Keep these objects separate:

- **Ommer header:** historical reference to an eligible non-canonical PoW block.
- **Transaction list:** ordered transaction inputs included in the canonical execution block.
- **Receipt list:** execution outcomes and logs for the canonical block's transactions.
- **State root:** commitment to the resulting state after canonical block processing.

This distinction matters when reading old block explorers, historical data, execution-client structures, or protocol documentation written before The Merge.

## 8. Why Should Developers Still Learn About Ommers?

Ommers are no longer produced in modern Ethereum blocks, but the concept remains useful for understanding protocol history and block structure.

Smart-contract developers and infrastructure engineers may encounter ommer-related fields in:

- Historical blocks and block explorers.
- JSON-RPC responses for older blocks.
- Ethereum's execution specifications and formal descriptions of the block format.
- The ommersHash field, which remains in the execution header.
- Educational material describing the transition from PoW to PoS.

When building software, do not infer that the presence of the field means current blocks may contain ommers. Validate behavior against the fork rules and the execution/consensus specifications relevant to the network and block being processed.

## 9. Key Takeaways

- An ommer was a valid non-canonical PoW block whose header could be referenced by a later canonical block.
- Historically, a block could include at most two eligible ommer headers.
- Ommer inclusion provided partial rewards to the ommer miner and a smaller inclusion bonus to the including miner.
- Referencing an ommer did not make the ommer canonical or execute its transaction list.
- Since The Merge, Ethereum execution blocks require an empty ommers list; historical ommer rewards and validation no longer apply.
- The ommersHash header field remains, fixed to Keccak256(RLP([])).

## References

- [EIP-3675: Upgrade consensus to Proof-of-Stake](https://eips.ethereum.org/EIPS/eip-3675) — specifies the empty ommer list, constant ommersHash, and removal of PoW-era block and ommer rewards.
- [Ethereum Yellow Paper](https://github.com/ethereum/yellowpaper) — formal description of the historical block structure and the deprecated ommer field.
- [Ethereum Whitepaper](https://ethereum.org/whitepaper/) — historical overview of uncle/ommer blocks and their role in the former PoW system.
- [Ethereum execution specifications](https://github.com/ethereum/execution-specs) — execution-layer protocol specifications.

*Last reviewed: October 12, 2026.*
