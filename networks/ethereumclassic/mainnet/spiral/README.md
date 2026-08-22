# Spiral

The current mainnet tip, and the upgrade with no inherited coverage at all — the archived corpus
carries a single filler under this name and no generated fixtures.

## Why Ethereum's Shanghai fixtures cannot be borrowed here

Spiral takes most of Shanghai and leaves two EIPs out, and both absences follow from the same
fact: **Ethereum Classic is proof-of-work and Ethereum is not.** One removes the beacon-chain
withdrawal operations; the other is the change that turned opcode `0x44` from `DIFFICULTY` into
`PREVRANDAO`.

That second one makes the two upgrades **mutually incompatible at the environment level**, not
merely different in behavior. Demonstrated with one binary, 2026-08-21:

| run | env demands | `0x44` returns |
|---|---|---|
| `ETC_Spiral` | non-zero difficulty, no base fee | the block difficulty |
| `Shanghai` | difficulty **must be zero**, base fee **required** | prevrandao |

Feeding this upgrade's environment to Shanghai is refused outright — *"post-merge difficulty must
be zero"* — and feeding Shanghai's to this upgrade drops the value the contract reads. **A
relabelled Shanghai fixture is not a weaker test here; it is an unrunnable one.**

## `difficulty_opcode.json`

Three bytes: `DIFFICULTY`, `PUSH0`, `SSTORE` — store the block difficulty at slot zero.

It exercises both halves of what makes this upgrade Ethereum Classic's:

- **`0x44` is `DIFFICULTY`**, because this chain never adopted the change that replaced it. The
  fixture asserts the environment's difficulty value lands in storage.
- **`PUSH0` is live**, because Spiral activates it. The same three bytes would be invalid before
  this upgrade.

The oracle, its version, and the generation date are recorded in the fixture's own `_info`.

## `warm_coinbase.json` and `cold_account_access_control.json`

EIP-3651 makes the coinbase account warm from the start of a transaction. Both fixtures store the
gas consumed by a `BALANCE` read, so the assertion is a number rather than a state shape.

Measured at fill time, the same bytecode across two adjacent upgrades:

| `BALANCE` on | Spiral | one upgrade earlier |
|---|---:|---:|
| the coinbase | **106** | 2606 |
| an unrelated address | 2607 | 2607 |

**The control is the point.** A cheaper coinbase read on its own could be a general repricing;
the unrelated address staying at 2607 is what attributes the 2,500-gas saving to this EIP
specifically. A fixture that only asserted the first row would pass for the wrong reason if
anything else about account access changed.

## What the failures proved, and they are worth keeping

While probing, the same bytecode was run one upgrade earlier and **wrote no storage at all**. That
is not a defect — `PUSH0` does not exist before this upgrade, so the code is invalid there.

So Spiral activation is confirmed from both directions: the opcode works here, and identical code
is rejected before. The pre-Spiral measurements above therefore use a `PUSH1 0x00` variant, so
that what is being compared is the account-access price and not the availability of an opcode.

## Adding to this directory

Read `../../../../FIXTURE-FORMAT.md` first. The expected values must come from an implementation
other than the one under certification, and which one has to be recorded in the fixture.
