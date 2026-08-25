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

## Ethereum's schedule changes shape at the Merge

Boundaries through Gray Glacier are **block numbers**; Shanghai, Cancun and Prague are
**timestamps**, and EIP-6122 extends the fork-identifier checksum to accumulate them after the
block-based ones. Every vector here carries both a head and a time.

**Ethereum Classic has no time-based fork**, so a reader written only against `../ethereumclassic/`
has never met this case. That is worth knowing before assuming a shared reader works for both.

## Sharing with Ethereum Classic is an assertion, never a directory

The two networks share a genesis and every rule through Homestead. There is no `shared/` directory
and there will not be one: it cannot say *shared by whom*, and becomes a grab bag the moment a third
network joins.

The agreement is recorded instead as a **claim inside the fixture that names both networks and the
exact head at which it ends** — identical through 1,149,999, diverging at 1,150,000 where the fork
hash still matches and the *next* fork does not. That degrades correctly: at a future divergence you
delete a sentence with a reason, rather than quietly moving a file and leaving no record that the
two ever agreed.
