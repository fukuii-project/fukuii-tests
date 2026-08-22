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
blocks/       block-level fixtures         rewards and uncles
forkid/       fork-identifier vectors
```

**Type first, because the harness resolves by type.** A corpus is declared to the client as a
directory, an upgrade label and a chain id: it reads *one directory* and pulls *one upgrade's*
expectation from every fixture in it. A difficulty fixture and a state fixture are read by
different code paths and cannot share a directory.

**No upgrade appears in a path.** The upgrade dimension lives in each fixture's `post` map, which
is keyed by label — a directory named for an upgrade would compete with that map and eventually
disagree with it. To cover twelve upgrades the harness reads one directory twelve times.

This repository briefly had `schedule/` and `spiral/` directories and both were mistakes:
`schedule/` named a property every state fixture has, and `spiral/` named an upgrade. They are
recorded here because they were invented while authoring and never written down, which is exactly
how they drifted.

## Coverage

Twelve of the schedule's eighteen entries are addressable by the state generator and all twelve
are filled. Of the remaining six, four change no state-transition rule, one was aborted, and one —
the mining-epoch change — has no fork name and would not be exercised by a state fixture anyway.

`state/README.md` carries the per-upgrade detail and the full ordered schedule.

**Two upgrades can never be reached from `state/`.** This chain's era emission is a block reward,
and the bomb removal is difficulty behavior. A perfect state suite still leaves both at zero,
which is why `difficulty/` and `blocks/` are not optional extras.

## Adding a fixture

Read `../../../FIXTURE-FORMAT.md` first. Expected values must come from an implementation other
than the one under certification, and which one has to be recorded in the fixture.

Prefer a body valid at every upgrade. A probe built from a late opcode can only ever be asserted
from that upgrade onward — a fixture measuring the coinbase's access cost was removed from here
for exactly that reason, superseded by the same test written to span the schedule.
