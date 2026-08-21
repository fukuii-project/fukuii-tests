# fukuii-tests — contributor guide

Test fixtures and conformance vectors for Fukuii. Part of the [Fukuii project](https://github.com/fukuii-project).

## Layout

- `etc/` — Ethereum Classic conformance corpus.
- `olympia/` — Fukuii fork vectors (ECIP-1111 / 1112 / 1121 / 1122).
- `hive/` — Fukuii client configs and simulators.
- `preserved/` — retained pre-Merge fixtures.
- Upstream suites (`ethereum/tests`, `hive`) are pinned submodules.

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
