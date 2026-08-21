# `proposals/` — tests scoped to a single proposal

**Fukuii's own work.** One directory per proposal namespace, matching the client's
`chainspec/proposals/` and its `ProposalId`, which admits `Eip(n)` and `Ecip(n)`.

```
proposals/
  eip/
  ecip/
```

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
