# `networks/` — tests scoped to a network

**Fukuii's own work.** Organized as the client organizes it: `networks/<family>/<network>`,
matching `chainspec`'s `networks/ethereumclassic/Mainnet`, `networks/ethereum/`, and so on.

```
networks/
  ethereumclassic/
    mainnet/
    mordor/
  ethereum/
```

## One directory per test type, and no upgrade in a path

Beneath a network, fixtures are grouped by **test type first**, then by subject:

```
networks/ethereumclassic/mainnet/
  state/            state-transition fixtures     post keyed by upgrade
    opcodes/          availability
    gas/              repricing
    storage/          reads, writes, refunds
    accounts/         creation, clearing, code limits
  difficulty/       difficulty fixtures           a different generator, a different reader
  blocks/           block-level fixtures          rewards, uncles
  forkid/           fork-identifier vectors
```

Type first because the harness resolves by type: a difficulty fixture and a state fixture are
read by different code paths and cannot share a directory. Subject beneath it because a flat
directory of a few hundred fixtures is unnavigable, and subject is the axis a reader searches by.

**No directory is ever named for an upgrade.** A corpus is declared to the client as
`(directory, upgrade label, chain id)`: the client reads one directory and pulls the expectation
for one upgrade out of every fixture in it, reading the same directory once per upgrade. The
upgrade dimension therefore lives inside each fixture, and a directory named for an upgrade
competes with that map and will eventually disagree with it.

Two upgrade-shaped directories did get created here while authoring, and were removed. They are
worth naming because the mistake is easy to repeat: one described *how* a fixture was scoped and
the other *which* upgrade it targeted, so a fixture could sit in either and the split decided
nothing. Recording the layout here is what stops that recurring -- it drifted precisely because
it lived only in the directory listing.

## Each type has its own file shape -- check it against the reader

The two types in use nest differently, and neither is guessable:

| type | shape |
|---|---|
| state | `{ config, env, pre, transaction, post: { <upgrade>: [ … ] } }` |
| difficulty | `{ <outerName>: { _info, <upgrade>: { <case>: { … } } } }` |
| fork identifier | `{ <outerName>: { _info, genesisHash, forkBlocks, vectors: [ … ] } }` |

**A fork-identifier file is not keyed by upgrade at all**, and that is a property of the subject
rather than an inconsistency. An identifier is a checksum over the *whole* schedule evaluated at a
head, so it has no per-upgrade expectation to carry; the vector list is indexed by head block. It
is also the one type here that can assert something about an upgrade changing no
state-transition rule, because a fork block participates in the checksum whether or not it
changes anything a transaction can observe.

A difficulty file carries an **outer name above the upgrade labels**, because the reader loads
the file as a map and hands each top-level *value* to its per-upgrade loop. Flattening that level
makes every field of every case look like an upgrade label. Confirm a new type's shape by
round-tripping a fixture through a client that reads it, with a deliberately wrong value proving
the reader rejects one -- not by pattern-matching an existing file.

## What is network-scoped, and what is not

The client composes rules in three layers, and a test belongs to the layer whose facts it
actually depends on:

| layer | asserts | lives in |
|---|---|---|
| proposal | one EIP or ECIP's rule delta, in isolation | `../proposals/` |
| upgrade | a composed rule set, named | this directory, at family level |
| activation | behavior at or across a specific height | this directory, under a network |

**Upgrade names are family-level; activation points are network-level.** Mordor runs the same
named upgrades as Ethereum Classic mainnet at entirely different heights. So a test asserting
*the rules in force at Agharta* holds for both networks, while a test asserting *what happens at
the Agharta block* is specific to one.

Getting this wrong is silent: a flat layout either pins family-level rule coverage to one
network's heights, or duplicates it per network and lets the copies drift.

## Not-yet-activated upgrades live here too

An upgrade that has been specified but has no activation point is still an entry in that
network's schedule — the client's `UpgradeSchedule` filters on whether an activation point is
defined, so unscheduled entries are a modelled case rather than an absence.

Coverage for one belongs under its network, marked as unactivated. **It does not get a parallel
tree** that has to be merged in later.

## The archive mapping belongs here

Which archived fixtures apply to which upgrade, and which of the archive's Ethereum Classic
labels are sound, is a per-upgrade question and is answered here.

`../archive/` is frozen and stays wrong where it is wrong. This directory is where the correct
answer is written — as our work, mapped, not by editing what was inherited.

## Two rules

**Read rule sets and activation points from the client, never from a rendered specification.** A
published specification can carry wrong activation points; the client is what the network runs.

**Do not restate an activation point in a file here.** Cite where to read it. A copied value goes
stale silently while continuing to read as authoritative.
