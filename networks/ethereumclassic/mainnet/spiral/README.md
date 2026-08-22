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

## Adding to this directory

Read `../../../../FIXTURE-FORMAT.md` first. The expected values must come from an implementation
other than the one under certification, and which one has to be recorded in the fixture.
