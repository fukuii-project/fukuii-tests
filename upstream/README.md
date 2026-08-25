# `upstream/` — tracked, not preserved

**Every entry here is a pinned submodule.** That is the rule for this directory, and it follows
from one fact: the Ethereum Foundation is alive and building for the Ethereum network. Its
repositories are not disappearing, so there is nothing here to rescue, and a copy would only
drift from what upstream is actually doing.

Contents are **not fetched by default**. A normal clone records pins and downloads none of it.

| entry | upstream | pin |
|---|---|---|
| `ethereum/tests` | `ethereum/tests` | `c67e485ff8` |
| `ethereum/legacytests` | `ethereum/legacytests` | `1f581b8ccd` |
| `ethereum/devp2p` | `ethereum/devp2p` | `51dc101fdd` |
| `ethereum/hive` | `ethereum/hive` | `dde4f59d04` |
| `ethereum/execution-specs` | `ethereum/execution-specs` | `ccaaaba58c` |
| `ethereum/execution-spec-tests` | `ethereum/execution-spec-tests` | `10eaa63d5d` — **archived upstream** |

Entries are named `<org>/<repo>`, so a path says where it came from. Ethereum and Ethereum
Classic both publish a repository called `tests`, and a bare name cannot tell them apart.

## What a pin buys, and what it does not

**Buys:** the material as it stood at the pin date, surviving upstream moving on. A non-shallow
clone at a pin carries upstream's whole history — including files later purged from its default
branch. That is why pinning is enough for most of this.

**Does not buy:** survival of the repository being deleted, its history rewritten, or a clone made
shallow. And a pin cannot make purged material *discoverable* — nobody finds a deleted directory
unless they already know it existed and which commit holds it.

**Where those gaps bite, the material is extracted into `../archive/ethereum/`** — a deliberate
subset, not a second copy. See `../archive/ethereum/EXTRACTION.md`.

## Fetching

```sh
git submodule update --init upstream/ethereum/devp2p        # 732 KB
git submodule update --init upstream/ethereum/hive          # 15 MB
git submodule update --init upstream/ethereum/tests         # 281 MB
git submodule update --init upstream/ethereum/legacytests   # 2.6 GB
git submodule update --init upstream/ethereum/execution-spec-tests   # 46 MB
git submodule update --init upstream/ethereum/execution-specs        # 54 MB
```

Sizes are the working tree, measured; a fetch also downloads history. They are here so the
2.6 GB entry cannot be started by accident, not as figures to keep current.

## Bumping a pin

Deliberately, never reflexively. `hive`'s simulators retarget to whatever Ethereum currently
ships, so older wire and proof-of-work coverage ages out of its default branch while staying
reachable at the pin. Bumping without checking is how that coverage is lost.
