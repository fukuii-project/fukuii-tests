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
| `archive/` | preserved copies of dying or deleted upstream material | **a submodule, and frozen** — see below |
| `upstream/` | live upstreams, pinned as submodules, fetched | tracked |
| `proposals/` | our tests for a single EIP or ECIP, network-agnostic | **authored** |
| `networks/` | our tests scoped to a network or an upgrade | **authored** |

`archive/` and `upstream/` sort by **source organization** — `archive/etclabscore/tests`,
`upstream/ethereum/tests` — because two upstreams both publish a repository called `tests` and a
path should say which one it is.

`proposals/` and `networks/` follow the client's own two axes, `chainspec/proposals/` and
`chainspec/networks/<family>/<Network>`.

### Two families, and they are NOT covered to the same depth

`networks/` holds two families and deliberately unequal amounts of work:

| family | upstream corpus | our standing | what we author |
|---|---|---|---|
| `ethereumclassic/` | unmaintained since 2023, deprecating | lead client maintainer | a complete suite |
| `ethereum/` | alive and maintained | a consumer of rules decided elsewhere | only what upstream structurally cannot hold |

**Do not "fill in" the Ethereum family by porting Ethereum Classic's fixtures across.** Ethereum's
own corpus is pinned under `upstream/` and is the authority for the EVM; re-authoring it here would
duplicate a far larger corpus, go stale as it moves, and carry no authority. The test for adding an
Ethereum-family fixture is one question — *could the upstream corpus express this?* — and the
measured answer for what it cannot is in `networks/ethereum/README.md`.

**The reverse also holds.** Ethereum Classic's coverage is complete because nobody upstream will
ever test ECIP-1017's emission, ECIP-1041's bomb removal, ECIP-1099's epoch change or ECIP-1100's
reorg defense. Those have no Ethereum counterpart and no upstream home.

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

### `archive/` is a SUBMODULE, and not initialised by default

It points at **`fukuii-project/archive-reference-material`** — reference material this project
archives, not an archive of this project's own material.

**That split exists for the consumer, and the reason is structural rather than a matter of
current size.** What a client needs to run this suite is `networks/`, `proposals/` and the format
specification — bounded by the schedule it covers. The archive is whole upstream repositories with
their history, so it is **orders of magnitude larger and grows every time anything is archived**,
while the fixtures do not. Carried in one history, that gap widens on its own and a Java or C#
implementer who wanted the fixtures pays for all of it.

**No size is quoted here on purpose.** Any figure is wrong the next time something is archived.
Measure it when you need it:

```bash
git ls-tree -r -l HEAD | awk '{s+=$4} END{print s/1048576 " MB"}'   # in either repo
```

```bash
git clone …/fukuii-tests            # fixtures and docs. This is what a consumer wants.
git submodule update --init archive # the archived material, when you actually need it.
```

**Do not describe the archive by listing what is in it, here or anywhere else.** It began as a
client lineage plus test corpora and has since taken in tooling and the personal-account work of
departed core developers; any sentence naming its contents is wrong at the next addition and
reads as settled while it is wrong. This is the same rule this file already applies to the
Olympia suite's membership and to clone sizes, and it decays the same silent way. **The authority
is `archive/PROVENANCE.md`, which carries one entry per vendored tree** — read it at the moment
you need it.

**A pin is not a copy, which is why the submodule points at a repository we own.** A gitlink is
twenty bytes of SHA and nothing else — the parent repository cannot even resolve it. Pinning a
third party means that if their repository is deleted the content is simply gone: GitHub does keep
a public repository's forks alive when it is deleted, but the URL in `.gitmodules` is dead either
way, and two of the corpora archived here have **zero forks**. Pointing at our own repository
moves that risk onto us, where we can answer for it.

### Archiving a corpus is not becoming its durable home

**These are two different jobs and reading them as one produces a false custody claim**, so the
distinction is stated here rather than inferred from the directory's existence.

| | what it means | where it lives |
|---|---|---|
| **archiving** | a frozen mirror of what an upstream published, kept so a copy can be compared against the original | `archive/`, this repository's submodule |
| **hosting** | being the maintained, durable home a corpus is fetched from and updated in | not this repository |

`archive/` does the first. Its whole value is that it is **frozen** — the section below exists to
keep it that way — and a frozen mirror is by construction not a living home for anything.

**Which project holds the second role is recorded where that decision was made, and this file does
not restate it.** That is the same rule this repository applies to specifications: a fact copied
out of its source goes stale silently, and a custody claim that has gone stale is worse than a
missing one because it names an org that may no longer be responsible. Read it from the client's
own corpus documentation at the moment you need it; do not cache it here, and do not infer it from
what `archive/` happens to contain.

**The inference to avoid specifically:** this repository archiving `<org>/tests` does **not** mean
it succeeded whatever effort previously undertook to host that corpus. Both can be true, either
can be true, and the directory listing cannot tell them apart.

### The archive is frozen, and corrections land in the suite

**Nothing under `archive/` is edited** — not to fix a label, not to satisfy a linter, not to
reorganize. Parts of it are wrong and stay wrong; the defects are recorded in
`archive/PROVENANCE.md`.

The reason is not reverence. A corrected mirror cannot be compared against what upstream
published, and that comparison is the only thing that makes a copy of a dead corpus worth
holding. Fixing it in place spends the artifact to save a rename.

**Editing an existing entry and appending a new one are different acts, and only the first is
frozen.** That is the archive's own rule, and restating it loosely here as "nothing is ever
written" would forbid the one write archiving something new requires. Recording a new corpus in
`PROVENANCE.md` or `NOTICE` is the documented path; revising a line already there is not. The
archive's `AGENTS.md` carries the command that proves a change was append-only — **run it there,
in that repository, rather than reasoning about it here.**

**Check the freeze by tree hash, not by diff.** `git subtree` relocates a corpus under a prefix,
so `git diff <ref>..HEAD -- <path>` reports every file as added and looks alarming while proving
nothing. Compare `git rev-parse '<ref>^{tree}'` against
`git rev-parse 'HEAD:<org>/<repo>'` **inside the archive submodule**; two identical hashes is the
whole proof. Note the path no longer carries an `archive/` prefix — that prefix was the mount
point, and inside the archive repository the organisation is the top level.

**Compare at `<org>/<repo>`, never at `<org>`, and this is a false-alarm trap rather than a
nicety.** An organisation's tree hash *necessarily* moves when that organisation gains an entry,
so an org-level comparison reports a frozen corpus as CHANGED and looks like a freeze violation
while proving nothing. Descend one level. **Calibrate the run too:** a sweep in which nothing
reports CHANGED cannot distinguish a held freeze from a broken comparison, so check it against
files you expect to differ — `AGENTS.md`, `PROVENANCE.md`, `NOTICE` — in the same pass.

**Bumping the gitlink is invisible to plain `git status` and `git diff`**, because `.gitmodules`
sets `ignore = all` on this submodule. That is deliberate and it hides the one thing you are
trying to confirm. Verify a bump by its own instruments instead:

```bash
git ls-files --stage archive                              # the staged SHA
git diff --cached --ignore-submodules=none --name-only    # must print exactly: archive
```

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

`networks/ethereumclassic/` encodes Ethereum Classic's upgrade schedule. **Everything through
Spiral, the current mainnet tip, is covered** — that was the first build target and it is met; every
activation in the production client's mainnet configuration is now asserted by a fixture. Mordor is
covered against its own configuration. The rule below governs anything added to either.

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
- **Fixture work branches first** — anything under `proposals/` or `networks/`. Archive work
  happens in `archive-reference-material` and lands here only as a submodule bump.
  A bad vector misleads conformance runs, so it gets an isolated history before it lands.
- **Branches are local and never pushed.** Fast-forward onto `main` when the work is done;
  `main` is the only branch that goes to the remote.
- **Pushing is confirmed each time**, separately from where the commit landed.
