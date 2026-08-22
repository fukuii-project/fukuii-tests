# `schedule/` — one body, asserted across the whole upgrade sequence

A fixture here carries a `post` entry **per upgrade**, not one. The format is keyed by upgrade
label, and the harness is told which to read, so a single test body can pin behavior across the
entire schedule.

## Why this is the shape, rather than one fixture per upgrade

**An upgrade's rule set is cumulative.** Spiral is not four EIPs; it is everything from Frontier
onward plus four. A fixture that only exercises what an upgrade *added* leaves every inherited
rule unverified at that upgrade — and an inherited rule silently regressing at a later upgrade is
exactly the defect a conformance suite exists to catch.

Filling one body across the sequence closes that by construction. Where the value is unchanged
between neighbours, the fixture asserts the rule did not move. Where it changes, the change is
the test.

## What the first fixture demonstrates

`coinbase_balance_access_cost.json` stores the gas consumed by `BALANCE(COINBASE)`:

| upgrade | gas | what moved it |
|---|---:|---|
| Frontier, Homestead | 26 | the original cost |
| Gas Reprice | 406 | the EIP-150 repricing |
| Atlantis, Agharta | 406 | unchanged — asserted, not assumed |
| Phoenix | 706 | account reads repriced again |
| Magneto, Mystique | 2606 | cold-access accounting introduced |
| Spiral | 106 | the coinbase becomes warm |

Four repricings and four confirmations-of-no-change, in one file.

## `chainid_opcode.json`

Two assertions in one body. The opcode does not exist before Phoenix, so every upgrade below it
writes no storage — and from Phoenix onward it returns **this chain's identifier, not
Ethereum's**.

| upgrade | slot zero |
|---|---|
| Frontier … Agharta | nothing written — the opcode is not there |
| Phoenix … Spiral | **61** |

**This one can never be borrowed, and the reason is worth stating precisely.** Ethereum's
equivalent upgrade activates the same EIP set, so the rule sets are identical — and filled there,
the identical body returns **1**. A fixture asserting one value is wrong for the other chain at
otherwise identical rules. The divergence is not in the rules; it is in what the chain *is*.

**An empty expectation is an assertion here, not a gap.** The five upgrades that write nothing are
asserting the opcode's absence. That is only legible because the fixture spans the schedule: at a
single upgrade, "no storage" and "we did not test this" look the same.

## Opcode availability across the schedule

Seven fixtures, one per opcode, each asserting where it becomes available. Every boundary was
confirmed against the production client rather than assumed from a specification:

| opcode | available from |
|---|---|
| `DELEGATECALL` | Homestead |
| `RETURNDATASIZE`, `STATICCALL` | Atlantis |
| `SHL`, `EXTCODEHASH` | Agharta |
| `SELFBALANCE`, `CHAINID` | Phoenix |
| `PUSH0` | Spiral |

### The probe stores a marker, not the opcode's result

Run the opcode, discard what it produced, store a non-zero value. A written slot means the opcode
executed; an empty one means it did not exist there.

**Storing the result instead does not work, and fails silently.** `RETURNDATASIZE` and
`SELFBALANCE` both return zero in this setup, and storing zero into an already-zero slot cannot be
distinguished from never storing at all — so the first version of these fixtures reported those
opcodes as unavailable at *every* upgrade, including ones where they plainly work. The probe
looked correct and measured nothing.

That is worth stating because it is the same failure this directory already documents twice: an
instrument that cannot report a negative is not measuring. Here the negative and the positive were
the same value.

## Two things the sequence surfaced that a single-upgrade fixture would not

**The signature scheme changes mid-schedule.** This chain adopts replay protection at Die Hard,
so every upgrade before it *rejects* a protected transaction and requires a legacy signature. The
fixture records which form each upgrade was filled with. A filler that signs one way for the whole
schedule silently produces nothing at the early upgrades — which is what happened first, and it
looked like an empty result rather than an error.

**Opcode availability and rule pricing are different axes.** An earlier body used `PUSH0`, which
does not exist before Spiral, so pre-Spiral upgrades failed for a reason that had nothing to do
with the rule under test. The body here uses `PUSH1 0x00` so it is valid everywhere and the only
thing varying is the price.

## The schedule in full, in order

Every entry, so the sequence reads completely and no absence is silent. **"Not filled" is a
classification here, never an omission.**

| # | upgrade | in a state fixture |
|---|---|---|
| 1 | Frontier | **filled** |
| 2 | Frontier Thawing | — the client models this as *unenforced*: scheduled and named, changing neither rule nor state |
| 3 | Homestead | **filled** |
| 4 | DAO Wars | — aborted; never activated on any chain |
| 5 | DAO Fork | — Ethereum's irregular state change, declined by this chain |
| 6 | Gas Reprice | **filled** |
| 7 | Die Hard | — no fork name in the generator |
| 8 | Gotham | — no fork name in the generator |
| 9 | Defuse Difficulty Bomb | — no fork name in the generator |
| 10 | Atlantis | **filled** |
| 11 | Agharta | **filled** |
| 12 | Phoenix | **filled** |
| 13 | MESS default on | — chain-selection policy, not a state-transition rule |
| 14 | Thanos | — no fork name in the generator |
| 15 | Magneto | **filled** |
| 16 | Mystique | **filled** |
| 17 | MESS default off | — chain-selection policy, not a state-transition rule |
| 18 | Spiral | **filled** |

Nine filled, nine classified. The nine that are not filled fall into **three distinct reasons**,
and collapsing them would misrepresent the coverage:

- **Nothing to assert** (2, 4, 5, 13, 17) — no state-transition rule changes, so a state fixture
  has no expectation to carry. Frontier Thawing is the clearest case: the client's own schedule
  marks it unenforced. Covering the MESS transitions needs chain-selection tests; covering the
  declined fork needs a block-level one.
- **No fork name** (7, 8, 9, 14) — the rules exist and are implemented, but the generator cannot
  be told to select them. A tooling gap, and the one that is worth closing.
- **Aborted** (4) — never happened anywhere.

## Upgrades with no expectation, and the two different reasons

Absence in a `post` map is never a claim that the case does not apply. Each fixture records which
of these it hit:

- **No fork name in the generator's table** — Die Hard, Gotham, Defuse Difficulty Bomb, Thanos.
  Their rules cannot be selected, so nothing can be filled. A tooling gap; see the parent
  directory.
- **Not a state-transition rule** — Frontier Thawing, DAO Wars, the DAO fork this chain declined,
  and both MESS transitions. A state fixture has nothing to assert about them; they need
  block-level or chain-selection tests.

Nine of the schedule's eighteen entries are fillable today. The other nine are accounted for by
one of those two reasons, and neither is "we did not get to it".
