# `proposals/` — tests scoped to a single proposal

**Fukuii's own work.** One directory per proposal namespace, matching the client's
`chainspec/proposals/` and its `ProposalId`, which admits `Eip(n)` and `Ecip(n)` — and one
namespace that deliberately does not, for a specification series that is neither. See *A third
namespace* below before adding a directory here.

```
proposals/
  eip/
    eip-225/            Clique proof-of-authority
      consensus/
  ecip/
  eea/
    qbft-v1/            QBFT, the EEA's own consensus specification
      consensus/
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

## A third namespace, and it is ahead of the client

`eea/` is not an oversight in the two-series list above. **QBFT is neither an EIP nor an ECIP.**
EIP-650 was IBFT 1.0 and was never merged: measured against the vendored `ethereum/EIPs`, there is
no `eip-650.md` at all and **zero** files mention QBFT, with `eip-155` present as the control. The
QBFT specification is published by the Enterprise Ethereum Alliance — *QBFT Blockchain Consensus
Protocol Specification v1* — and cited from the EEA Client Specification's CONS-092.

So the series is the EEA's, and the directory says so.

**This is ahead of the client, deliberately and visibly.** `ProposalId` admits `Eip(n)` and
`Ecip(n)` and has no case for a third series — read at the moment this landed, not assumed. But its
own documentation says the set is open: *"A network that authors its own series adds a case here…
a case added later makes every exhaustive match over this type report where it has to be
extended."* Adding that case is the client's call and is not made here; what is recorded here is
that a fixture exists whose proposal has no `ProposalId` yet.

The alternative — filing QBFT under `eip/` because that namespace already exists — would state
something false about where the specification comes from, and would do it in a path, which is the
one place a reader never re-checks.

**`qbft-v1` follows the publisher's own path**, `entethalliance.org/specs/qbft/v1/`, by exactly the
rule that makes `eip-225` follow `ethereum/EIPs`' own filename. The version is part of the name
because the specification carries one and the EEA's "latest release" currently resolves to it; a v2
would be a sibling rather than an edit.

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
