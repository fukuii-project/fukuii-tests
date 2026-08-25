# `proposals/` — tests scoped to a single proposal

**Fukuii's own work.** One directory per proposal namespace, matching the client's
`chainspec/proposals/` and its `ProposalId`, which admits `Eip(n)` and `Ecip(n)`.

```
proposals/
  eip/
  ecip/
```

## This directory is empty today, and the reason is structural

**No fixture has yet turned out to be proposal-scoped**, and that is worth stating rather than
leaving a reader to wonder whether something is missing.

Everything authored so far depends on a specific network. A state fixture carries a chain id and a
`post` map keyed by **one family's upgrade labels**, so `ETC_Phoenix` is not a thing Ethereum has;
a fork identifier is a checksum over one network's whole schedule; emission, the difficulty bomb and
the reorg defense are Ethereum Classic's alone. None of those is a delta on a rule set in isolation.

**A genuinely proposal-scoped fixture would have to carry no chain id and no network's label set** —
just a rule delta and its observable effect. That is expressible in the format and nothing here has
needed it yet. When one does, this is where it goes.

The alternative — putting network-agnostic material under whichever network happened to need it
first — is the failure this axis exists to prevent, so the directory stays and stays empty until
something genuinely belongs in it.

## Why proposals are their own axis

The client models a proposal as a **delta on the rule set** — `Component(id, delta: UpgradeRules
=> UpgradeRules)` — which upgrades then compose. A proposal's effect is therefore meaningful on
its own, independent of any upgrade that adopts it or any network that schedules it.

**So a proposal-scoped test is network-agnostic.** EIP-150's repricing behaves the same whether
it arrives on Ethereum Classic at Gas Reprice or on Ethereum at Tangerine Whistle. Testing it
here says so once.

Anything that depends on *which* upgrade adopted it, or *when* a network activated it, is not
proposal-scoped and belongs in `../networks/`.

## ECIPs are first-class here

`ProposalId` carries `Ecip` alongside `Eip` deliberately. Ethereum Classic's own proposals — its
monetary policy, its difficulty-bomb removal, its Etchash epoch change, its reorg defense — have
no Ethereum counterpart, and nothing upstream will ever test them.

**That coverage does not exist anywhere today.** It is tested only inside individual clients, in
different languages, with no shared vectors — which is much of why this repository exists.

## Naming

Follow the client: the directory is `eip/`, not `eips/`, because `ProposalId.Eip` is. The rule is
in `AGENTS.md` — read the client before naming anything we author.
