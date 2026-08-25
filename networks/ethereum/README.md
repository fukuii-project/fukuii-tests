# `networks/ethereum/`

Fixtures for the Ethereum family. **This directory is deliberately much smaller than
`../ethereumclassic/`, and the asymmetry is the design rather than a gap.**

## Why we author almost nothing here

The two families are in opposite positions, and coverage follows that rather than importance:

| | Ethereum Classic | Ethereum |
|---|---|---|
| upstream corpus | `etclabscore` — **unmaintained since 2023**, scheduled for deprecation | `ethereum/tests` — **alive and maintained** |
| its ETC-labelled tier | thin, and thinning at the edges | not applicable |
| this project's standing | **lead client maintainer** | a consumer of rules decided elsewhere |
| consequence | we author a complete suite | **we pin theirs and author only what it cannot hold** |

Re-authoring Ethereum's state, gas, opcode and precompile fixtures would duplicate a corpus far
larger than anything maintainable here, go stale as upstream moves, and carry no authority — the
rules are Ethereum's, and so is the corpus that defines them. `upstream/ethereum/tests` is pinned
for exactly that reason, and `AGENTS.md`'s "vendor, extract, or pin" rule already decides it: a live,
intact upstream is pinned, because a copy would only drift.

## What upstream structurally cannot hold

**Measured, not assumed.** The pinned corpus keys every expectation on a fork **name** and is
network-agnostic — it says what Byzantium *rules* do. It contains **zero fork-identifier vectors**
and **zero network activation heights**.

A network is precisely that missing half: a mapping from heights to rule sets, plus the facts that
belong to the chain rather than to the EVM. That is what this directory holds, and the test for
adding anything here is one question:

> **Could the upstream corpus express this?** If yes, it belongs upstream and we pin it. If no,
> it belongs here.

## What is here

| directory | holds |
|---|---|
| `mainnet/forkid/` | fork identifiers across the full schedule, both block-based and timestamp-based |
| `mainnet/blocks/` | the DAO irregular state change — an event rather than a rule |
| `mainnet/consensus/` | the Merge, keyed to accumulated difficulty rather than to a height |
| `sepolia/forkid/` | fork identifiers — one block-based boundary, and it is the Merge |

## Ethereum's schedule changes shape at the Merge

Boundaries through Gray Glacier are **block numbers**; Shanghai, Cancun and Prague are
**timestamps**, and EIP-6122 extends the fork-identifier checksum to accumulate them after the
block-based ones. Every vector here carries both a head and a time.

**Ethereum Classic has no time-based fork**, so a reader written only against `../ethereumclassic/`
has never met this case. That is worth knowing before assuming a shared reader works for both.

## Three shapes here that Ethereum Classic has no equivalent of

Worth listing, because a reader arriving from `../ethereumclassic/` will not have met any of them:

1. **Timestamp-based fork boundaries.** Shanghai onward activate by time, and EIP-6122 accumulates
   them into the fork identifier after the block-based ones.
2. **A transition keyed to accumulated difficulty.** The Merge answers neither "at what block" nor
   "at what time" — it answers "when total difficulty crosses a threshold", and the reference client
   records no merge height at all.
3. **An irregular state change.** A scripted mutation at one height that is not a rule.

## Sepolia is not a scaled-down mainnet

It runs the same rules and almost none of the same schedule. **Ten upgrades that mainnet reached
over twelve million blocks are Sepolia's starting state** — Homestead through London all activate at
block 0, and a zero contributes no entry to the fork-identifier checksum. So the network has exactly
**one** block-based boundary, and it is the Merge.

That inverts mainnet. There, the Merge is keyed to accumulated difficulty and produces **no**
fork-identifier entry; here it is `MergeNetsplitBlock`, whose name ends in `Block`, and identifiers
are gathered by reflection over exactly those names — so the same consensus transition **is** a
boundary and the fork hash changes at it.

**A client that learned the shape from mainnet looks for a difficulty threshold and finds a block;
one that learned it here looks for a block and finds a threshold.** Neither is wrong about its own
network, and that is what the roadmap means by a testnet expressing a mechanism differently from its
mainnet.

## Sharing with Ethereum Classic is an assertion, never a directory

The two networks share a genesis and every rule through Homestead. There is no `shared/` directory
and there will not be one: it cannot say *shared by whom*, and becomes a grab bag the moment a third
network joins.

The agreement is recorded instead as a **claim inside the fixture that names both networks and the
exact head at which it ends** — identical through 1,149,999, diverging at 1,150,000 where the fork
hash still matches and the *next* fork does not. That degrades correctly: at a future divergence you
delete a sentence with a reason, rather than quietly moving a file and leaving no record that the
two ever agreed.
