# Provenance — `upstream/`

Every entry is a pinned submodule of a **live, maintained** upstream. Nothing here is vendored.
This directory tracks what the Ethereum Foundation is doing; it does not preserve anything.

| entry | upstream | pin | upstream date |
|---|---|---|---|
| `ethereum/tests` | `https://github.com/ethereum/tests` | `c67e485ff8` | 2025-06-04 |
| `ethereum/legacytests` | `https://github.com/ethereum/legacytests` | `1f581b8ccd` | 2025-06-04 |
| `ethereum/devp2p` | `https://github.com/ethereum/devp2p` | `51dc101fdd` | 2026-07-13 |
| `ethereum/hive` | `https://github.com/ethereum/hive` | `dde4f59d04` | 2026-07-30 |
| `ethereum/execution-specs` | `https://github.com/ethereum/execution-specs` | `ccaaaba58c` | 2026-08-10 |
| `ethereum/execution-spec-tests` | `https://github.com/ethereum/execution-spec-tests` | `10eaa63d5d` | 2026-07-02 — **archived** |

Recorded 2026-08-21.

## Known: `ethereum/tests` is being reduced upstream

Measured 2026-08-21 — its HEAD carries only Prague, Cancun and 10 stray Istanbul network labels.
Every proof-of-work-era fork label is gone from the labelled corpus.

The purge is **not uniformly archived**. Some went to `legacytests`; some went nowhere:

| commit | what | archived? |
|---|---|---|
| `f4f76dc8a0` 2025-03-17 | archive transition tests to legacy | yes — `legacytests` |
| `78e58e1b6e` 2025-03-11 | Remove all EIPTests — 4,232 files | **no** |
| `613f53571c` / `2317dd5dff` 2025-05-14 | remove general state tests + fillers | yes — `legacytests` |
| `2db0128918` 2025-05-05 | remove ported to .py intrinsic tests | superseded upstream |

The unarchived material is extracted into `../archive/ethereum/`. The pins here remain the way to
re-derive and verify that extraction.

## `legacytests` health, measured

Not abandoned, **measured 2026-08-21 and stated as a reading of that date rather than a standing
claim**: 30 commits since 2021-10-06, activity rising (3/yr through 2023, 7 in 2024, 14 in 2025),
last commit the same day as `ethereum/tests` HEAD. Re-measure before relying on it. Its README describes it as legacy tests
for all clients to test against.

It holds 36,535 json across the proof-of-work-era forks — **more completely than any other
surviving source**. It carries zero `PoWTests` and zero `DifficultyTests`, which is why those are
extracted rather than relied upon here.

## The fixture release — recorded, not pinned, and not vendored

The modern fixture corpus is published as a **release artifact**, not a repository. There is no
commit to point a submodule at, so it cannot be pinned the way everything else here is.

| field | value |
|---|---|
| release | `tests-v20.0.1` |
| published by | `execution-spec-tests` — releases now come from `execution-specs` |
| size | 8.4 GB unpacked |
| proof-of-work-era subset | 483 MB (`state_tests` + `blockchain_tests`, Frontier through London) |

**Not vendored, deliberately.** Three reasons, in order of weight:

1. **It is regenerable.** The generator is pinned above at `ccaaaba58c`. A fixture release is
   output, and the input is preserved.
2. **`legacytests` already holds the classic corpus more completely** — see the near-miss
   recorded below.
3. **Most of it can never apply here.** Roughly 3.0 GB is Engine API tests, and Ethereum Classic
   has no consensus layer; the rest above London is post-Merge.

**What a release artifact does not get from any of this:** GitHub can remove release assets, and
no pin protects them. If that risk is ever judged to matter, the answer is to vendor the
proof-of-work-era subset — 483 MB, and fork is a path component (`state_tests/for_berlin`), so
the selection is clean and auditable.

## Purge review, 2026-08-21 — what each pin was checked for

Every pinned repository was checked for material it has removed that no other upstream holds.
**Recording the negative results so they are not re-derived.**

| repo | finding | action |
|---|---|---|
| `ethereum/tests` | `EIPTests` purged 2025-03-11, archived nowhere; `PoWTests` / `DifficultyTests` at HEAD with no archive | **extracted** to `../archive/ethereum/tests` |
| `ethereum/hive` | DAO-fork simulators removed 2020; devp2p `eth` and `discv4` removed 2022 | **extracted** to `../archive/ethereum/hive` |
| `ethereum/legacytests` | 59 paths ever deleted, **all** `Cancun/`; zero proof-of-work-era | nothing to extract |
| `ethereum/devp2p` | last content deletion 2020 and cosmetic; eth/62 through eth/72 all documented | nothing to extract |
| `ethereum/execution-specs` | live home of the migrated spec tests; carries more proof-of-work-era coverage than the archive | nothing to extract |
| `ethereum/execution-spec-tests` | archived 2026-07-02; fork tests migrated to `execution-specs`, static fillers held more completely by `legacytests` | nothing to extract |

**Re-run this review when bumping a pin.** A bump moves the window in which upstream may have
deleted something, and the pin itself is what makes the old state reachable to compare against.
