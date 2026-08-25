# `networks/ethereumclassic/mordor/`

Fixtures for **Mordor**, this chain's proof-of-work testnet. Same family as mainnet, same rules —
and almost none of the same numbers.

## Why nothing here is copied from `../mainnet/`

Mordor runs the same rule set. What differs is every parameter those rules read, and the table is
the reason a copied fixture would be wrong rather than merely imprecise:

| | mainnet | Mordor |
|---|---|---|
| chain id / network id | 61 / 1 | **63 / 7** |
| genesis | shared with Ethereum | **its own** |
| ECIP-1017 era length | 5,000,000 | **2,000,000** |
| ECIP-1010 difficulty pause | 3,000,000, length 2,000,000 | **never existed** — the config field is null |
| ECIP-1041 bomb removal | 5,900,000 | **0** — the bomb never ran here at all |
| ECIP-1099 Etchash | 11,700,000 | **2,520,000** |
| declined DAO fork | 1,920,000, excluded from the fork identifier | **does not arise** — Mordor postdates it |

**Every one of those was read from the production client's `MordorChainConfig`, not from a
specification.** That is this repository's standing rule and it matters more here than on mainnet:
a testnet's published parameters are revised more casually, and there is no economic activity to
make an error obvious.

## The defect these fixtures exist to catch is network confusion

Not a misread of a rule — **a correct implementation reading the wrong network's number.**

That error is invisible to code review, because the code is right. It is invisible to a
single-network test suite, because each network passes its own. And it is the error a client most
plausibly makes, since supporting a testnet usually means reusing the mainnet path with a different
config.

Measured against deliberately-wrong builds: a client applying mainnet's era length to Mordor scores
**2/11** on emission, and one applying mainnet's Etchash activation scores **6/12** on the epoch
schedule. Each fixture records its own scores in `_info`.

**A Mordor fixture copied from mainnet would agree with that broken client and certify it.** That is
the whole reason this directory exists rather than a `--network` flag on the mainnet files.

## What is here, and what is deliberately absent

| directory | holds |
|---|---|
| `forkid/` | fork identifiers across Mordor's six fork blocks |
| `blocks/` | ECIP-1017 emission at Mordor's era length |
| `pow/` | ECIP-1099 epoch schedule at Mordor's activation |

**There is no `difficulty/bomb_*` fixture here, and its absence is an assertion in waiting rather
than an omission.** Mordor's `DisposalBlock` is 0 and its `ECIP1010PauseBlock` is null: the
difficulty bomb never ran on this network. Mainnet's bomb fixture has no Mordor counterpart to
rebase — the correct Mordor fixture would assert that **no bomb term is ever added at any height**,
which is a different test with a different shape. Worth authoring; not yet authored.

**There is no `state/` here yet.** Mordor's rule sets are the same EIP sets mainnet reaches, so
state fixtures would largely restate `../mainnet/state/` with a different chain id — the exception
being `chainid_opcode`, which must return **63**, and which is exactly the kind of value a copied
fixture gets wrong. That is the first state fixture worth authoring here.

## Adding a fixture

Read `../../../FIXTURE-FORMAT.md` first, then this directory's rule: **derive every parameter from
Mordor's own configuration and say in `_info` that you did.** Never adapt a mainnet fixture by
editing numbers — the numbers are the whole content, and an edited copy carries mainnet's
provenance while claiming Mordor's.
