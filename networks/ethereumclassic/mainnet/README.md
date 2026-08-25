# `networks/ethereumclassic/mainnet/`

Fixtures for Ethereum Classic mainnet, authored by this project.

## Organized by test type, never by upgrade

```
state/        state-transition fixtures    post keyed by upgrade
  opcodes/      availability, and what an opcode returns
  gas/          repricing
  storage/      reads, writes, refunds
  accounts/     creation, clearing, code limits
difficulty/   difficulty fixtures          a different generator reads these
blocks/       block-level fixtures         emission, ommer credits, required headers,
                                           receipt encoding
forkid/       fork-identifier vectors
chainselection/  reorg-defense vectors     which of two chains a node prefers
pow/          proof-of-work epoch vectors  which dataset a seal is verified against
```

**Type first, because the harness resolves by type.** A corpus is declared to the client as a
directory, an upgrade label and a chain id: it reads *one directory* and pulls *one upgrade's*
expectation from every fixture in it. A difficulty fixture and a state fixture are read by
different code paths and cannot share a directory.

**No upgrade appears in a path.** The upgrade dimension lives in each fixture's `post` map, which
is keyed by label — a directory named for an upgrade would compete with that map and eventually
disagree with it. The harness reads one directory once per upgrade, however many there are.

This repository briefly had `schedule/` and `spiral/` directories and both were mistakes:
`schedule/` named a property every state fixture has, and `spiral/` named an upgrade. They are
recorded here because they were invented while authoring and never written down, which is exactly
how they drifted.

## Coverage

Twelve of the schedule's eighteen entries are addressable by the state generator and all twelve
are filled. Of the remaining six, four change no state-transition rule, one was aborted, and one —
the mining-epoch change — has no fork name and would not be exercised by a state fixture anyway.

`state/README.md` carries the per-upgrade detail and the full ordered schedule.

**Four upgrades can never be reached from `state/`, not two.** This chain's era emission is a block
reward and the bomb removal is difficulty behavior -- and beyond those, the reorg-defense pair is
chain-selection policy and Thanos is a proof-of-work rule. A perfect state suite leaves all four at
zero, which is why the other four directories are not optional extras.

**Every activation in the production client's mainnet configuration is covered by an assertion**,
and that claim is derived from the configuration field by field rather than from the EIP-shaped
subset of it. Two were found exactly that way: `ECIP1099FBlock`, which no `EIP...Block` grep can
see, and `RequireBlockHashes`, which is not an activation block at all.

The last to close was EIP-658, and it closed for a different reason worth separating from those
two. It was never *missed* — the rule ledger named it, identified its observable, and recorded that
a state fixture could not carry it. What was missing was a place to put it. `blocks/` is that
place, and the fixture landing there is what turns the ledger's last row from a classification into
an assertion.

## Adding a fixture

Read `../../../FIXTURE-FORMAT.md` first. Expected values must come from an implementation other
than the one under certification, and which one has to be recorded in the fixture.

Prefer a body valid at every upgrade. A probe built from a late opcode can only ever be asserted
from that upgrade onward — a fixture measuring the coinbase's access cost was removed from here
for exactly that reason, superseded by the same test written to span the schedule.
