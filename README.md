<div align="center">
  <img src="https://raw.githubusercontent.com/fukuii-project/fukuii-brand/HEAD/logo/fukuii-hex-logo.png" alt="fukuii-tests" width="280"/>
</div>

# fukuii-tests

Conformance fixtures for Ethereum Classic and Ethereum, as language-neutral JSON.

**Not test code.** Every fixture here is data. The runner lives in whatever client consumes it, in
whatever language that client is written in — which is the point: a Scala-specific test would be
Fukuii's alone, while a fixture can be run by any client on either chain.

Part of the [Fukuii project](https://github.com/fukuii-project).

---

## Start here: run one

Most fixtures under `networks/*/*/state/` are **directly runnable by go-ethereum's state-test
runner**, with no conversion — the shape is geth's own `GeneralStateTest` shape:

```bash
evm statetest networks/ethereumclassic/mainnet/state/gas/sload_cost.json
```

Two adjustments a geth-family runner needs, both documented in
[FIXTURE-FORMAT.md](FIXTURE-FORMAT.md):

- **Three Ethereum Classic labels have no entry in any core-geth fork table.** Map
  `ETC_Frontier` → `Frontier`, `ETC_Homestead` → `Homestead`, `ETC_GasReprice` → `EIP150`. They are
  equivalent: before Die Hard this chain and Ethereum ran the same rules.
- **`config.chainid` is ignored by that runner.** Its test struct has no `config` field at all, so
  every fixture executes at whatever chain id the named fork carries. Harmless on mainnet, and
  **wrong for Mordor** — see the warning in the format document before running anything under
  `networks/ethereumclassic/mordor/state/`.

**Everything passes except the cases that runner structurally cannot check.** No count is quoted
here on purpose — it is wrong the next time a fixture is added. Measure it:

```bash
for f in $(find networks -path '*/state/*.json'); do evm statetest "$f"; done
```

The only expected failures are the `expectException` cases, and none of them is a wrong expectation:
that runner builds its message from the transaction object's `sender` and never validates a
signature or decodes an envelope, so it cannot express *"this transaction is refused here"*. A
consumer whose harness treats `txbytes` as the authority — which the format document requires — can
check them.

## What is covered

| network | fixtures | posture |
|---|---|---|
| **Ethereum Classic mainnet** | 56 | a complete authored suite |
| **Mordor** (ETC testnet) | 8 | complete against its configuration |
| **Ethereum mainnet** | 3 | only what the upstream corpus cannot hold |
| **Sepolia, Holesky, Hoodi** | 4 | the same |

**The asymmetry is deliberate and is the first thing to understand about this repository.** Fukuii
is the lead maintainer of an Ethereum Classic client and the ETC upstream corpus is unmaintained, so
ETC gets a full suite authored here. Ethereum's own corpus is alive and vastly larger than anything
maintainable here, so it is **pinned** under `upstream/` and this repository authors only what that
corpus structurally cannot contain — fork identifiers, activation heights, the DAO event, the Merge.

The test for adding an Ethereum-family fixture is one question: *could the upstream corpus express
this?* If yes it belongs upstream. If no it belongs here. See
[`networks/ethereum/README.md`](networks/ethereum/README.md).

## Shapes

Seven, and they are not interchangeable — a reader for one will not read another.

| directory | holds | keyed by |
|---|---|---|
| `state/` | state-transition fixtures | upgrade label |
| `difficulty/` | difficulty adjustment and the bomb | upgrade label |
| `blocks/` | emission, ommer credits and validity, required headers, receipt encoding | block number |
| `forkid/` | fork identifiers | head, and on Ethereum also time |
| `chainselection/` | reorg-defense (MESS) vectors | — |
| `pow/` | proof-of-work epoch schedule | block number |
| `consensus/` | the Merge — accumulated difficulty, or a netsplit block | — |

**[FIXTURE-FORMAT.md](FIXTURE-FORMAT.md) is the contract.** Every field, its units, what a reader
does when one is absent, and which shapes have a reader today. Read it before writing a reader;
the table above is only for finding the right section.

## What makes a fixture here trustworthy

Two properties, both stated in each fixture's own `_info`:

**An independent oracle.** Expected values come from an implementation other than the one under
certification, named in the fixture. Where a value could be derived from a specification instead, it
was — and where two sources exist, both are recorded.

**A measured ability to fail.** A fixture that passes a correct client has demonstrated nothing; it
may be pinning a value no rule depends on. So fixtures here are scored against **deliberately broken
builds** of a reference client, and the score is recorded. Where a fixture cannot be scored that
way — a required block hash is configuration, not arithmetic — the fixture says so rather than
leaving the absence to look like an omission.

## Repository layout

| path | what | fetched by default |
|---|---|---|
| `networks/` | fixtures scoped to a network — **this project's work** | yes |
| `proposals/` | fixtures for a single EIP or ECIP, network-agnostic | yes |
| `upstream/` | live upstream corpora, pinned as submodules | no |
| `archive/` | preserved copies of dying or deleted upstream material, frozen | no |

**A plain clone gives you the fixtures and nothing else, which is the point:**

```bash
git clone https://github.com/fukuii-project/fukuii-tests.git   # fixtures and docs only
git submodule update --init upstream/ethereum/tests            # Ethereum's own corpus
git submodule update --init archive                            # the archived client lineage — large
```

No clone size is quoted, for the same reason `AGENTS.md` quotes none for the archive: the figure is
wrong the next time a fixture is added, and a stale one reads as current. Measure it:

```bash
git ls-tree -r -l HEAD | awk '{s+=$4} END{print s/1048576 " MB"}'
```

Nothing under `archive/` is ever edited. See [AGENTS.md](AGENTS.md) for why, and for how the freeze
is verified.

## Contributing

Read [AGENTS.md](AGENTS.md) first — it carries the rules that are easy to violate by accident, in
particular: **verify activation heights against a client, never against a rendered specification.**
The published rendering of ECIP-1066 contains two wrong activation blocks, one by an order of
magnitude, and a fixture generated from it inherits both.

## License

Apache-2.0 for Fukuii's own files. Vendored upstream corpora retain their original licenses. See
[`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

© 2025–present The Fukuii Authors · Chippr Robotics LLC · White B0x Inc.
