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
