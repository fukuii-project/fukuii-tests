# `proposals/` — tests scoped to a single proposal

**Fukuii's own work.** One directory per proposal namespace, matching the client's
`chainspec/proposals/` and its `ProposalId`, which admits `Eip(n)` and `Ecip(n)`.

```
proposals/
  eip/
    eip-225/            Clique proof-of-authority
      consensus/
  ecip/
```

## What has to be true before a fixture belongs here

**No chain id, and no network's upgrade labels** — a rule delta and its observable effect, nothing
else. That test is why this directory stood empty while `networks/` filled up, and it is still the
test.

Everything authored before EIP-225 depends on a specific network. A state fixture carries a chain id
and a `post` map keyed by **one family's** upgrade labels, so `ETC_Phoenix` is not a thing Ethereum
has; a fork identifier is a checksum over one network's whole schedule; emission, the difficulty
bomb and the reorg defense are Ethereum Classic's alone. None of those is a delta on a rule set in
isolation.

**EIP-225 is the first that is.** Clique's authorized signer set is a pure function of the header
chain — no chain id reaches the rule, no upgrade schedules it, and the same vectors hold on every
network that runs Clique. A fixture asserting *a particular Clique network's* genesis signer set or
block period would not be: that is network-scoped and belongs in `../networks/`.

The alternative — putting network-agnostic material under whichever network happened to need it
first — is the failure this axis exists to prevent. Nothing lands here to fill the directory; it
lands here because it passed the test above.

## Naming beneath a namespace

`eip-225/` follows the convention the source corpus uses — `ethereum/EIPs` names its own files
`eip-225.md` — per the rule in `../AGENTS.md` that a data tree follows the corpora's conventions
rather than the client's type names. Beneath it, the **test type** is a directory exactly as under a
network, because a harness resolves by type before it resolves by subject.

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
