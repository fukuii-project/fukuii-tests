# fukuii-tests — contributor guide

Test fixtures and conformance vectors for Fukuii. Part of the [Fukuii project](https://github.com/fukuii-project).

## Layout

**Target layout — none of it exists yet.** The repository currently holds only its
license, wiring, and this guide. Treat the entries below as where things go when they
land, not as directories you can read today, and check before assuming a path is there.

- `etc/` — Ethereum Classic conformance corpus.
- `olympia/` — Fukuii fork vectors, one directory per ECIP in the Olympia suite.
- `hive/` — Fukuii client configs and simulators.
- `preserved/` — retained pre-Merge fixtures.
- Upstream suites (`ethereum/tests`, `hive`) are to be pinned submodules. There is no
  `.gitmodules` yet, so nothing is pinned.

### Do not hardcode the Olympia suite's membership

Not in this file, not in a script, not in a directory listing treated as canonical.
The specs are under active revision and the set moves in **both** directions: an ECIP
can be cited before it is authored, and can exist in the authoring copy before it is
published. A list written down here reads as settled and decays with nobody editing it
— which is worse than an obviously missing one, because nothing about it looks unchecked.

Read the membership from the ECIP specs at the moment you need it. Where an authoring
working copy of the specs is available it is authoritative and the published rendering
is not; a stale published spec answers every question you ask it, confidently and
wrongly. Contributors with that working copy configured will find the path and branch
in their local, uncommitted `CLAUDE.local.md`.

The corollary: an ECIP number is not necessarily public. Do not introduce one into a
tracked file in this repo without checking that the spec is published.

## Fork schedule vectors: verify against a client, never against the rendered spec

The `etc/` corpus encodes ETC's fork activation schedule, and the first build target is
**everything through Spiral (block 19,250,000), the current mainnet tip**.

**The published rendering of ECIP-1066 contains wrong activation blocks.** This is not a
hypothetical: two entries in the public version are incorrect, one by an order of
magnitude. A vector set generated from the ECIPs website or its GitHub mirror inherits
both errors, and every downstream conformance run then agrees with itself while
disagreeing with mainnet.

So an activation block gets into a fixture only after it is confirmed against the
**production** client's chain configuration, which is what mainnet actually runs. Where
they disagree, the client wins and the spec text needs a fix.

Two cautions that cost time if missed:

- **There is more than one core-geth checkout, and they are not interchangeable.** The
  Olympia development fork carries future fork blocks that are not live on mainnet.
  Reading it for a *current* schedule silently yields the wrong answer. Confirm which
  one you have open before quoting a number from it.
- **EIP numbering differs between the spec and the client for the alt_bn128
  precompiles** — the spec's EIP-196 / EIP-197 are the client's EIP-213 / EIP-212. Same
  forks, same activation block, different number. Not a discrepancy; do not "fix" it.

## Working here

- Public repo — never commit secrets; scrub fixtures of any real keys or tokens.
- Commits follow [Conventional Commits](https://www.conventionalcommits.org/) — see the org
  [Contributing guide](https://github.com/fukuii-project/.github/blob/main/CONTRIBUTING.md).
- Vendored corpora keep their upstream licenses; record attribution in `NOTICE` when adding them.

## Branching

Settled once, so it is not re-decided per commit:

- **Wiring, config, and docs go straight to `main`.** Trivially revertible, and CI reports
  status without blocking.
- **Fixture work branches first** — anything under `etc/`, `olympia/`, `hive/`, or
  `preserved/`. A bad vector misleads conformance runs, so it gets an isolated history
  before it lands.
- **Branches are local and never pushed.** Fast-forward onto `main` when the work is done;
  `main` is the only branch that goes to the remote.
- **Pushing is confirmed each time**, separately from where the commit landed.
