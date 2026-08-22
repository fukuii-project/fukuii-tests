# `networks/ethereumclassic/` — the Ethereum Classic family

Upgrade-scoped tests for Ethereum Classic, and the assessment of which archived material can be
trusted for each upgrade.

**Family level, so it is shared by every network below it.** A rule set is the same on mainnet
and on Mordor; only the height at which it takes effect differs. Anything keyed to a height
belongs under `mainnet/` or `mordor/`.

## Two authorities, and they answer different questions

| question | authority |
|---|---|
| what does the network actually run? | **core-geth**, the production client |
| what is it called, and how is it structured? | **fukuii-cli**, `chainspec` |

This split is not a temporary state. core-geth is what mainnet runs today, so it settles rule
content and activation. `fukuii-cli` settles vocabulary and shape — `Upgrade`, `UpgradeRules`,
`Activation`, and the entry names.

**Where fukuii-cli has not yet declared an upgrade, that is not evidence the upgrade does not
exist.** Its schedule is being built up in order, so read rule content from core-geth and take
only the naming from the client.

## The archive's Ethereum Classic labels are not uniformly trustworthy

`../../archive/etclabscore/tests` carries labels produced by a text substitution over Ethereum
fixtures. **Renaming a label does not change the EIP set inside a fixture.** Measured 2026-08-21
against the frozen archive:

| label | fixtures | derived from | trustworthy? |
|---|---:|---|---|
| `ETC_Phoenix` | 7,469 | Istanbul | **yes** — equivalent rule set |
| `ETC_Magneto` | 5,301 | Berlin | **yes** — the production client calls it equivalent |
| `ETC_Agharta` | 99 | Constantinople+Petersburg | probably — confirm before relying on it |
| `ETC_Atlantis` | 90 | Byzantium | **no** — Ethereum Classic lacks an EIP that Byzantium carries |
| `ETC_Mystique` | 5,496 | London | **no** — Ethereum Classic takes a strict subset of London |
| `ETC_Spiral` | 1 | — | **effectively absent** — one filler, no generated fixtures |

### The sharpest defect, and why it is not a labelling nit

The substitution also rewrote Ethereum's **proof-of-stake** transition onto an Ethereum Classic
upgrade. 344 fixtures in the archive still carry the pre-substitution label, and the rewritten
ones sit under an Ethereum Classic name while their blocks carry proof-of-stake header values.

**Those fixtures assert that a block with zero difficulty is valid.** On a proof-of-work network
it is not. Running them as Ethereum Classic coverage does not merely test the wrong thing; it
asserts the opposite of the rule.

### Where the divergence bites depends on the tier

| tier | consequence |
|---|---|
| state tests — transaction-level | usually sound; no block reward or header validity is applied |
| blockchain tests — block-level | **unsound where the rule sets differ**, because the block itself is validated |

## What follows for this directory

- Sound labels can be **referenced** from the archive. They do not need reproducing.
- Overclaiming labels need **our own coverage**, written here, for the upgrades whose rule sets
  actually diverge.
- `ETC_Spiral` has no archived coverage at all, and Spiral is the current mainnet tip. That is the
  largest gap and the first thing worth authoring.

**Read rule sets and activation points from the client. Do not restate them here** — a copied
value goes stale silently while continuing to read as authoritative.
