# `components/` — the pieces a chain is assembled from

**Fukuii's own work, and the half of this corpus that is not about any particular chain.**

```
components/
  proposals/              one proposal's rule delta      eip/  ecip/
  consensus-algorithms/   one consensus mechanism        poa/  pow/
```

## The word is the client's

`chainspec` models a proposal as a **delta on the rule set**, and builds an upgrade by adopting
several:

```scala
final case class Component(id: ProposalId, delta: UpgradeRules => UpgradeRules)
def adopting(added: Component*): UpgradeRules
```

That is the relationship this directory names. **`../networks/` is the real implementation of chain
families; `components/` is what those implementations are assembled from.** A network says *"I run
Clique, and I activate EIP-155 at block 3,000,000"*; the components are the Clique rules and the
EIP-155 delta, tested on their own.

## The test, and it is the only thing you need to remember

> **A component is a pure function of its own inputs.
> A network is anything keyed to a height, a chain id, or a schedule.**

Clique's in-turn difficulty rule is the same rule on every Clique chain — a component. QBFT's
proposer selection is a function of a validator set, a parent proposer and a round — a component.
*When Ethereum Classic activated Etchash* is a fact about Ethereum Classic — a network fixture.

## Why a state fixture can never be a component, however proposal-shaped its subject

This is the part that surprises people, and it is a property of state roots rather than a filing
convention.

`networks/ethereumclassic/mainnet/state/gas/sload_cost.json` has EIP-2929 as its subject. What it
**asserts** is a state root at each of twelve `ETC_*` labels, at chain id 61. A state root is the
hash of a trie produced by executing under one specific rule composition — the same EIP-2929 delta
on Ethereum at Berlin yields a *different* root. **There is no network-agnostic state root.**

So that fixture answers *"where in this chain's schedule does the behavior change?"*, which is a
network question, even though its subject is a proposal. It is filed by what it asserts, which is
the rule.

**A component-level test of the same proposal would assert something else**: the gas numbers
themselves, the rule outcome, with no chain id and no schedule. That shape does not exist in this
corpus yet, which is why `proposals/` currently holds no fixtures. **That is the test above being
honest, not a gap somebody forgot to fill.**

## Each component suite is self-contained

A fixture here carries everything needed to run it, including **the configuration it needs**, in
its own `_info` or a `chainConfig` block — see `consensus-algorithms/poa/clique/`, which states the
header field count, which forks are active at block zero, and the terminal total difficulty.

**In the fixture, never in a sidecar file.** A config that can be separated from the vectors it
describes will be, and then the pair drifts. Where a component needs no configuration at all —
QBFT proposer selection needs none — there is no block, and its absence is information.

## Adding something here

Read `../AGENTS.md` for the three layers and `../FIXTURE-FORMAT.md` for the schema every file must
satisfy. Then apply the test at the top: if the expectation moves when you change the chain id or
the activation height, it is not a component and belongs in `../networks/`.
