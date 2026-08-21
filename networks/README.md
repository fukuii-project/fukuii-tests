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
