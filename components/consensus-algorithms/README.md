# `consensus-algorithms/` — tests scoped to a consensus mechanism

**Fukuii's own work.** One directory per mechanism, under the class of mechanism it belongs to.

```
consensus-algorithms/
  poa/     clique/  qbft/          proof of authority
  pow/                             proof of work
```

## What has to be true before a fixture belongs here

**The subject is the ALGORITHM, and the fixture holds wherever that algorithm runs.** No chain id,
no network's upgrade labels, and no dependence on which network happens to have deployed it. Clique's
in-turn difficulty rule is the same rule on every Clique chain; QBFT's proposer selection is the same
function on every QBFT chain.

**A fixture that depends on one network's constants is network-scoped and belongs in `../../networks/`.**
The two merge predicates under `../../networks/*/consensus/` are the worked contrast: *when* a particular
chain stopped doing proof of work is a fact about that chain, keyed to its own terminal difficulty
and netsplit block, and it does not generalize to another network running the same engine.

## Why this is its own root rather than a corner of `proposals/`

**Because a consensus algorithm is not an improvement proposal, and only one of these has ever
looked like one.** Clique has an EIP number; QBFT, IBFT, AuRa, Ethash and Etchash do not. Filing
Clique under `proposals/eip/eip-225/` was correct on its own terms and generalized badly — the
second mechanism to arrive had no number, and was given a namespace invented to hold it.

`../proposals/` is for a **proposal's rule delta**, and the client's `ProposalId` models numbered
improvement proposals. A mechanism is a different kind of subject, and this repository's own layout
rule is to split by **what a thing is**.

**The client agrees, and it was read rather than assumed:** its modules are `chainspec`,
`consensus`, `consensus-pow`, `evm`, `execution` and so on — consensus engines are their own axis
there too, beside `chainspec/` rather than inside it.

**A mechanism's proposal number, where it has one, is an `_info` fact.** `clique_seal_difficulty.json`
records EIP-225 in its metadata, which is where a citation belongs; a path is not a citation, and
encoding one in a directory name is what made the previous layout brittle.

## No test-type directory, and that is deliberate

Under `networks/` and `proposals/` the test type is a directory, because a harness resolves by type
before it resolves by subject. Here the subject *is* the mechanism and **the outer key of each file
selects the schema**, exactly as `FIXTURE-FORMAT.md` describes for a heterogeneous directory. A
`consensus/` level beneath `clique/` would name the thing twice and resolve nothing.

## Naming

Directory names are the mechanism's own, lowercase: `clique`, `qbft`, `ibft`, `aura`, `ethash`,
`etchash`. **Not versioned.** QBFT's specification carries a version and the algorithm does not;
which document governs a fixture is recorded in its `_info`, where it can be corrected without
moving a file.

Read `../README.md` for the test that separates a component from a network, `../../AGENTS.md`
before adding a directory here, and `../../FIXTURE-FORMAT.md` for the schema each file must
satisfy.
