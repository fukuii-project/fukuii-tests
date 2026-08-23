# fukuii-tests — contributor guide

Test fixtures and conformance vectors for Fukuii. Part of the [Fukuii project](https://github.com/fukuii-project).

## What this repository contains, and what runs it

**Fixtures, as language-neutral JSON. Not test code.**

The runner is Scala and lives in the client, not here — `fukuii-cli` has the certification harness
under its `chainspec` and `evm` test trees, built on ScalaTest. This repository supplies the data
it reads.

That split is deliberate and standard: every client consumes the same fixtures through its own
harness, in its own language. It is why a fixture is worth authoring at all — a Scala-specific
test would be Fukuii's alone, while a fixture can be run by any Ethereum Classic client.

### Naming follows the fixture convention, not Scala's

`proposals/` and `networks/` are **data trees**. Directory and file names follow the conventions
the corpora already use — lowercase, fork- and suite-shaped — not Scala's PascalCase. A
`FooSpec.scala` name here would be wrong; there is no Scala here to name.

The client-vocabulary rule below still applies to the *concepts* — upgrade, activation, proposal
— and to directory names we invent. It does not turn data files into type names.

**The schema is in [FIXTURE-FORMAT.md](FIXTURE-FORMAT.md)** — what the harness parses, the
Ethereum Classic specifics that differ from Ethereum's, and why an authored fixture's expected
values must come from an implementation other than the one being certified.

### How the client finds a corpus

`fukuii-cli` resolves a fixture root at test time, from an environment variable or a pointer file
in its own working tree, and expects corpora at organization-scoped paths beneath it.

**It does not point here yet, and it should not until there is a suite to point at.** Today it
reads the raw reference clones directly, which is the right bootstrap. Read the client for the
current variable name and the exact paths rather than restating them here — that contract is
under active development.

## Layout

Four roots, split by **what a thing is** rather than which network it concerns. Nothing is named
for a bucket that means different things in different places.

| directory | what it holds | posture |
|---|---|---|
| `archive/` | preserved copies of dying or deleted upstream material | **frozen — never edited** |
| `upstream/` | live upstreams, pinned as submodules, fetched | tracked |
| `proposals/` | our tests for a single EIP or ECIP, network-agnostic | **authored** |
| `networks/` | our tests scoped to a network or an upgrade | **authored** |

`archive/` and `upstream/` sort by **source organization** — `archive/etclabscore/tests`,
`upstream/ethereum/tests` — because two upstreams both publish a repository called `tests` and a
path should say which one it is.

`proposals/` and `networks/` follow the client's own two axes, `chainspec/proposals/` and
`chainspec/networks/<family>/<Network>`.

### The three layers, and which one a test belongs to

The client composes rules in three layers. A test belongs to the layer whose facts it actually
depends on, and putting it a layer too low duplicates it while a layer too high overclaims:

| layer | asserts | scope | lives in |
|---|---|---|---|
| proposal | one EIP or ECIP's rule delta, alone | network-agnostic | `proposals/` |
| upgrade | a named, composed rule set | family | `networks/<family>/` |
| activation | behavior at or across a height | network | `networks/<family>/<network>/` |

**Upgrade names are family-level; activation points are network-level.** The same named upgrades
land at entirely different heights on a testnet than on mainnet, so rule coverage is shared
across a family while anything keyed to a height is not.

**An upgrade with no activation point yet still lives under its network**, marked unactivated —
the client's `UpgradeSchedule` treats unscheduled entries as a modelled case. It does not get a
parallel tree that has to be merged later.

### A pin is readable, so read it

**The pins are fetched, and an unfetched pin is a reason to fetch it, not to skip the check.**
`git submodule update --init <path>` materializes the exact recorded tree; the SHA does not move,
so the working tree stays clean and nothing is re-decided.

**Never answer a question about a pin by reading a newer ref of the same upstream.** A branch
tracking upstream is a different commit that will answer confidently and sometimes differently,
which is the same failure this repository already documents for a rendered specification against
its authoring copy. It is also not durable: a commit no branch or tag reaches can be dropped by a
routine `git gc`, so a pin that resolves against a moving clone today may not tomorrow. The pin
is only guaranteed readable where it is held.

**A claim about the whole corpus has to include the pins**, which are the majority of it by file
count. Verify by reading the tree at the pinned SHA, and prefer comparing content over comparing
directory names — two corpora can carry identically-named directories holding different data, or
identical data under a name that suggests otherwise.

### Vendor, extract, or pin — decided by what would be lost

Not by habit. Ask what disappears if the upstream vanishes tomorrow:

- **Dying upstream → vendor it whole, with history, into `archive/`.**
- **Live upstream removing parts → extract only the unarchived material** into `archive/`, and
  pin the rest in `upstream/`.
- **Live upstream, intact → pin it in `upstream/`.** A copy would only drift.

Each site records which was chosen and why. **Do not "correct" one into another.**

### The archive is frozen, and corrections land in the suite

**Nothing under `archive/` is edited** — not to fix a label, not to satisfy a linter, not to
reorganize. Parts of it are wrong and stay wrong; the defects are recorded in
`archive/PROVENANCE.md`.

The reason is not reverence. A corrected mirror cannot be compared against what upstream
published, and that comparison is the only thing that makes a copy of a dead corpus worth
holding. Fixing it in place spends the artifact to save a rename.

**Check the freeze by tree hash, not by diff.** `git subtree` relocates a corpus under a prefix,
so `git diff <ref>..HEAD -- <path>` reports every file as added and looks alarming while proving
nothing. Compare `git rev-parse '<ref>^{tree}'` against
`git rev-parse 'HEAD:archive/<org>/<repo>'`; two identical hashes is the whole proof.

**Gaps and inherited mistakes are answered in `proposals/` and `networks/` instead** — our tests,
our names, mapped. A reader can then see both what was inherited and what this project asserts,
which is impossible when the two are the same edited files.

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

`networks/ethereumclassic/` encodes Ethereum Classic's upgrade schedule, and the first build
target is **everything through Spiral, the current mainnet tip**.

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
- **Fixture work branches first** — anything under `archive/`, `proposals/`, or `networks/`.
  A bad vector misleads conformance runs, so it gets an isolated history before it lands.
- **Branches are local and never pushed.** Fast-forward onto `main` when the work is done;
  `main` is the only branch that goes to the remote.
- **Pushing is confirmed each time**, separately from where the commit landed.
