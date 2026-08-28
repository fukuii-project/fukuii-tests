# Fixture format — what a harness has to read

Everything in `proposals/` and `networks/` is **data**. This file is the contract: every shape,
field by field, with its type, its units and what a reader must do when a field is absent.

**It is written for someone implementing a harness in a language nobody here has used.** Several
clients across several languages are meant to run these fixtures, and one more should need nothing
but this file. Where a rule below is subtle, the reason it is subtle is stated, because a
convention whose reason is unrecorded is a convention that gets "simplified" back out.

**No client reads this directory yet.** The readers cited below consume the vendored Ethereum
corpora, in the same formats, and are described here because they are the only working readers of
any of these shapes that exist. Treat them as the reference implementation of a format, not as a
consumer of this suite.

Verified against the readers and the corpus on **2026-08-26**, which is a dated reading of the
corpus rather than a promise about it.

**Counts are deliberately absent from this file, and that is a correction rather than a style.**
Every hardcoded number here decayed: it said six shapes while a seventh directory already existed,
and four `blocks/` schemas while there were eight. A count reads as precise and goes stale with
nobody editing it, which is the same failure this repository documents for the archive's contents
and for a specification suite's membership. Where you want a number, derive it — the commands are
below.

---

## Read this first: the shapes are not equally settled

Taking them as equally hardened is the mistake to avoid. Two axes matter and they are independent:
whether the format **pre-exists this repository**, and whether anything **reads it yet**.

### The inventory is the tree; this file is the contract

**This file cannot tell you what exists, and twice it has claimed to and been wrong.** Derive the
inventory from the corpus, then read the contract for each thing you find:

```bash
# every authored fixture, wherever the roots are. This names NO directory on purpose:
# submodule contents are gitlinks and are never listed, and a fixture is recognised by
# carrying `_info` rather than by where it sits -- so a new root needs no edit here.
fixtures() {
  { git ls-files '*.json'; git ls-files --others --exclude-standard '*.json'; } | sort -u |
  while read -r f; do
    jq -e 'type=="object" and (to_entries|any(.value|type=="object" and has("_info")))' \
       "$f" >/dev/null 2>&1 && echo "$f"
  done
}

fixtures | cut -d/ -f1 | sort | uniq -c          # the roots actually in use
fixtures | while read -r f; do                   # every schema, against the file carrying it
  jq -r --arg f "$f" 'keys[] | "\(.)\t\($f)"' "$f"
done | sort
```

**That predicate is calibrated, and re-calibrate it if you change it.** Run against this corpus it
selects every fixture and rejects exactly `.claude/settings.json` and `.mcp.json` — a filter that
cannot reject anything is not filtering.

Both were run against this corpus when this file was last revised, so a reader who gets no output
is looking at a broken invocation rather than an empty tree.

**It lists untracked-but-not-ignored files too, and that is a correction rather than the original
design.** The first version used `git ls-files` alone, on the reasoning that the corpus is what is
committed. True of the corpus, wrong for the check: an audit exists to catch an undocumented schema
**before** it lands, and a tracked-only enumerator is blind at exactly that moment. Recorded because
it fired on 2026-08-27 — a new fixture was written, the audit reported zero gaps, and it was
reporting on a tree that did not contain it. `--exclude-standard` keeps `.gitignore` in force, so
scratch trees stay out.

**A clean audit still means nothing if the invocation is broken**, which is the older caution and it
stands: both commands above were run against this corpus when this file was last revised, so a
reader who gets no output is looking at a broken invocation rather than an empty tree.

And this file can audit itself. **Only where the outer key selects a schema**, which is a property
this file declares rather than one the tree reveals: in `forkid/` and `difficulty/` every file
shares one schema and the key merely names an instance, so auditing those would report every
fixture as an undocumented schema. The declaration is one marker line, kept next to the check so it
cannot drift out of sight:

<!-- schema-selecting-paths: blocks/ consensus-algorithms/ /consensus/ components/proposals/ -->

```bash
sel=$(grep -oP '(?<=schema-selecting-paths: ).*(?= -->)' FIXTURE-FORMAT.md | head -1)
documented=$(grep -oE '^### .*' FIXTURE-FORMAT.md | grep -oE '`[A-Za-z0-9_]+`' | tr -d '`' | sort -u)
carried=$(fixtures | grep -F -f <(printf '%s\n' $sel) | xargs -I{} jq -r 'keys[]' {} | sort -u)
for k in $carried; do
  printf '%s\n' "$documented" | grep -qx "$k" && continue
  hit=0; for d in $documented; do case "$k" in "$d"*) hit=1;; esac; done   # per-network variant
  [ "$hit" = 1 ] || echo "UNDOCUMENTED SCHEMA: $k"
done
```

**Pick the token carefully: it is a path fragment, not a type name.** `/forkid/` looks like the
right token for the fork-identifier shapes and is wrong — it also matches the SIX network-level
forkid files, whose outer keys are instance names rather than schemas, and the audit then reports
every one of them as undocumented. `components/proposals/` matches only the component layer, where
the key really does select the schema. Measured when the token was first added, not reasoned.

**Adding a root costs one token in that marker, not an edit to the command** — which is the whole
point, because the previous version hardcoded `*/blocks/*.json` and `*/consensus/*.json` inside the
command and silently stopped covering a schema the moment a root appeared. One did, on 2026-08-27.

It printed nothing when this file was last revised. **Calibrate it before trusting a clean run** —
delete a `###` heading's backticks and confirm it reports that schema, because a check that cannot
fail is not a check.

**A type directory that appears in the first command and nowhere in this file is a gap in THIS
FILE**, not an undocumented fixture — close it here rather than working around it in a reader.

### Two levels, and conflating them is the recurring error

| level | what it is | what a harness does with it |
|---|---|---|
| **type directory** | `state/`, `blocks/`, `consensus/`, … | selects which reader runs |
| **outer key** | the single top-level key inside each file | names that file's contents, and in some directories also selects the schema |

**The outer key is not a synonym for the schema.** In `forkid/` and `difficulty/` every file shares
one schema and the key merely distinguishes instances — one per network. In `blocks/` and
`consensus/` the key genuinely selects the schema, and files sitting side by side share nothing
beyond `_info`. **Never assume a type directory is homogeneous**, and dispatch on the schema you
identified rather than on how many keys you happened to see.

### Which formats pre-exist, and which this repository publishes alone

| shape | directory | format | a reader exists |
|---|---|---|---|
| state | `state/` | **pre-existing** — geth's GeneralStateTest, shared with the Ethereum corpora | yes |
| difficulty | `difficulty/` | **pre-existing** — shared with `DifficultyTests` | yes |
| proof of work | `pow/` | **pre-existing, extended here** | yes |
| block-level | `blocks/` | **specified here** | **per-schema, not per-directory** |
| fork identifier | `forkid/` | **specified here** | not yet |
| chain selection | `chainselection/` | **specified here** | not yet |
| consensus mechanism | `components/consensus-algorithms/<class>/<algo>/`, and `consensus/` under a network | **specified here** | not yet |

**Where a format pre-exists**, get it wrong and your harness disagrees with a corpus of tens of
thousands of published files. Fix your reader.

**Where this repository is the only publisher**, no upstream corpus carries the format, because no
upstream corpus tests these rules — Ethereum has no era emission, no declined fork to exclude
from a checksum, and no reorg-defense rule. If one is awkward in your language, report it rather
than work around it: nothing has hardened these yet.

**"Has a reader" is per-schema wherever the outer key selects one.** A consumer reading the
emission vectors has done nothing toward reading the receipt-encoding ones, though both live in
`blocks/`. Establish which schemas your reader covers before treating a directory as supported.

> **Do not name a specific file in the client here.** An earlier version of this table cited three
> reader filenames and **all three had ceased to exist** — the client is under active development
> and its test tree moves. Name the module and let the reader search it; a stale filename is worse
> than none, because it reads as precise. This is the same rule this repository applies to
> specifications: cite, never restate.

**Do not infer a shape by pattern-matching a neighbouring file.** They nest differently and none is
guessable from another. Confirm a new one by round-tripping a fixture through a reader with a
deliberately corrupted value, and check that the reader *rejects* it — a reader that cannot report
a negative is not reading.

---

## The reading model

A corpus is declared to a client as a triple:

```
(directory, upgrade label, chain id)
```

The client reads **one directory**, and out of every fixture in it pulls the expectation for
**one upgrade**. Covering a schedule therefore means reading the same directory once per upgrade,
under one label each time — however many upgrades that schedule currently has. Read the count off
the labels in a `post` map rather than from any prose; it grows when the chain activates an
upgrade.

Three consequences, all of which shape the layout:

- **The upgrade dimension lives inside a fixture, never in a path.** No directory is named for an
  upgrade. See `networks/README.md`.
- **The chain id comes from the declaration, not from the file.** State fixtures carry
  `config.chainid` as documentation and for harnesses that want it, and `fukuii-cli` takes the
  value from the corpus declaration instead. **Both must agree.** A harness that reads neither
  will admit a transaction signed for another network.
- **Which rules are in force is the schedule's answer, not the corpus's.** A corpus is resolved at
  a network and a height, and whatever that network's schedule holds there is what runs. Naming a
  rule composition directly would certify the composition and say nothing about the activation.

### The upgrade labels

**Read the labels off the corpus, not off this file.** The set grows whenever the chain activates
an upgrade, and a list written here decays with nobody editing it:

```bash
find networks proposals -name '*.json' \
  -exec jq -r '..|objects|select(has("post"))|.post|keys[]' {} \; | sort -u
```

**A label belongs to a family, and the two families do not share a vocabulary.** Ethereum Classic
fixtures use this suite's own `ETC_`-prefixed labels in schedule order; Ethereum-family fixtures
use upstream's own names, which carry no prefix at all. The command above returns both mixed
together, which is the honest answer — a reader resolving a label has to know which family's
schedule it belongs to, and the path says so.

Further labels name entries in a schedule that carry **no state expectation anywhere in this
suite**. Derive those the same way, by diffing the schedule your client resolves against the
command above rather than trusting a list.

Each shape records why a label carries nothing, in its own way — a difficulty fixture keys them
in an `_info.unfillable` map, a state fixture names them in the value of an `_info.unfilled-*`
key. Either way the reason is written down; see "What a MISSING expectation means" below.

**The labels are this suite's, not the production client's.** `ETC_Frontier` is what that client
calls `Frontier`. A label is carried through a reader unmapped and resolved by whoever runs the
case; a label that resolves to no rule set is a **divergence to report**, never a case to skip
quietly. A case that disappears between the corpus and the report is coverage nobody can audit.

---

## Conventions every shape shares

### Quantities are strings, flags are JSON booleans

**Every numeric value is a JSON string.** Not a JSON number — an unsigned 256-bit quantity does
not survive an IEEE-754 double, and several fixtures deliberately state values that overflow a
64-bit integer because overflow is what they test.

A quantity string is **hex when it starts `0x` or `0X`, decimal otherwise**, and a reader must
accept both in every position:

| spelling | value |
|---|---|
| `"0x0f4240"` | 1000000 |
| `"1000000"` | 1000000 |
| `"0x"` | 0 |
| `""` | 0 |

Which spelling a file uses is consistent *within* a shape and differs *between* them — state
fixtures are hex throughout, difficulty, block-level and chain-selection fixtures are decimal
throughout, and fork-identifier fixtures are hex for hashes and decimal for block numbers. That is
a property of the corpora each shape came from and **not** something a reader may rely on. Parse
both, everywhere.

Reject a quantity wider than 256 bits rather than truncating or wrapping it. Wrapping produces a
well-formed world the fixture never described, with nothing reported.

**Booleans are real JSON booleans** (`true` / `false`), never `"true"`. Three shapes use them:
`state`'s `replay-protected`, `blocks`'s `uncleRewardDependsOnDistance` and `matchesComputed`, and
`chainselection`'s `active` and `rejected`.

**One field is JSON integers rather than strings**, and only one: `post[].indexes.{data,gas,value}`.
They are array subscripts, they are small, and the upstream corpus writes them that way.

### Byte strings

`0x`-prefixed, and an **odd digit count is left-padded rather than rejected** — a fixture writes a
storage value and a byte string in the same spelling, and only one of the two must be whole bytes.
An address is 20 bytes exactly; a hash is 32 bytes exactly; anything else of either length is a
malformed fixture, not a short one.

### `_info` is metadata and carries no cases

Every fixture has one. A reader **skips it** wherever it appears, at whatever level. Its keys are
free-form prose, and a reader must not depend on any of them.

What it is *for* is the part that matters: it is where a fixture records the things a bare
expectation cannot say — which implementation produced the values and at what version, what a
control is controlling, why an upgrade carries nothing, what a state root is standing in for. Four
keys recur and should be present on anything authored here:

| key | says |
|---|---|
| `comment` | what the fixture asserts and how to read it |
| `oracle` | which implementation produced the expected values, or which specification they were derived from |
| `oracle-version` | at what version — a fixture whose oracle cannot be re-run cannot be re-derived |
| `generated` | the date, `YYYY-MM-DD` |

**A fixture whose oracle is unrecorded is not usable.** It cannot be re-derived and cannot be
re-checked when a client changes.

### Duplicate keys are an error, and "duplicate" means *after* parsing

`"0x0"` and `"0x00"` are the same storage slot, and `"0xAB…"` and `"0xab…"` are the same address.
A file naming one twice must be **rejected**, not silently resolved to whichever came last. The
two spellings look distinct in the text and collide in the map, which is the shape a reader
silently loses data on.

---

## state fixtures (`state/`)

An existing format. One file holds one or more named cases:

```jsonc
{
  "<case name>": {
    "_info":  { },                              // skipped
    "config": { "chainid": "0x3d" },            // 61, Ethereum Classic mainnet
    "env": {
      "currentCoinbase":   "0x2adc25665018aa1fe0e6bc666dac8fc2697ff9ba",
      "currentNumber":     "0x1263d54",
      "currentTimestamp":  "0x65bd0a80",
      "currentDifficulty": "0x0f4240",          // NON-ZERO -- see below
      "currentGasLimit":   "0x7a1200"
    },
    "pre": {
      "0x<20-byte address>": {
        "balance": "0x…", "nonce": "0x…", "code": "0x…", "storage": { "0x…": "0x…" }
      }
    },
    "transaction": {
      "nonce":    "0x00",
      "sender":   "0x<20-byte address>",
      "to":       "0x<20-byte address>",        // "" or "0x" means CREATION
      "gasPrice": "0x3b9aca00",                 // or maxFeePerGas, for a fee-market transaction
      "gasLimit": ["0x186a0"],                  // ARRAYS -- see "the indexes matrix"
      "value":    ["0x00"],
      "data":     ["0x", "0x01"],
      "accessLists": [null, [ { "address": "0x…", "storageKeys": ["0x…"] } ]]
    },
    "post": {
      "<UpgradeLabel>": [
        {
          "indexes": { "data": 0, "gas": 0, "value": 0 },   // JSON integers
          "hash":    "0x<32-byte post-state root>",
          "logs":    "0x<32-byte hash of the RLP-encoded logs>",
          "txbytes": "0x<the signed transaction>",
          "state":   { "0x<address>": { } },
          "expectException": "TransactionException.…",
          "replay-protected": false
        }
      ]
    }
  }
}
```

### Required, optional, and what a reader does with each

| field | required | notes |
|---|---|---|
| `env.currentCoinbase` | **yes** | 20-byte address |
| `env.currentNumber` | **yes** | block height the transaction executes at |
| `env.currentTimestamp` | **yes** | seconds |
| `env.currentDifficulty` | **yes** | **must be non-zero**, see below |
| `env.currentGasLimit` | **yes** | the whole block's limit; a state fixture is one transaction against an otherwise empty block, so the transaction may ask for all of it |
| `pre.<address>.balance` `.nonce` `.code` | **yes** | `code` may be `"0x"` |
| `pre.<address>.storage` | no | absent means empty. **A slot holding `"0x00"` is written as a zero, not skipped** — the fixture said so, and letting its omission stand in for it agrees with an implementation for the wrong reason |
| `transaction.nonce` `.sender` `.to` | **yes** | `to` empty or `"0x"` is a contract creation |
| `transaction.gasPrice` | for a legacy transaction | a fee-market transaction states `maxFeePerGas` instead; charge it zero rather than failing to read the file, because such a transaction is invalid at these forks whatever it states |
| `transaction.gasLimit` `.value` `.data` | **yes** | arrays, indexed by `indexes`; a bare string is accepted as a one-element array |
| `transaction.accessLists` | no | see "typed transactions" |
| `post.<label>[].indexes` | **yes** | all three of `data`, `gas`, `value` |
| `post.<label>[].hash` | **yes** | the post-state trie root |
| `post.<label>[].logs` | no | when present, the keccak of the RLP-encoded log list |
| `post.<label>[].txbytes` | no | the signed transaction. **Where present it is the authority**; see below |
| `post.<label>[].state` | no | a partial expected post-state, checked account by account over the addresses and slots it names. The root covers everything else |
| `post.<label>[].expectException` | no | see "two vocabularies" |
| `post.<label>[].replay-protected` | no | documentation of how the case was signed; **no reader acts on it** |

### `currentDifficulty` is non-zero and must stay that way

Ethereum Classic is proof-of-work. Post-Merge Ethereum fixtures carry `"currentDifficulty":
"0x00"`, and the archived corpus holds some that were relabelled onto an Ethereum Classic upgrade
without that being corrected — see `networks/ethereumclassic/README.md`. **Never copy a
zero-difficulty fixture forward.**

### `chainid` is 61, not 1

A fixture omitting it inherits a default that is wrong here, and the failure surfaces as a
signature mismatch that looks like something else entirely.

### The `indexes` matrix — one body, many transactions

`gasLimit`, `value` and `data` are arrays. A `post` entry's `indexes` triple selects one entry of
each, so **one fixture expands into a matrix of runs**, and the same file can hold a case and its
control side by side rather than in a second file that has to be read alongside it.

```
indexes: {"data": 2, "gas": 0, "value": 0}
   ->  data[2], gasLimit[0], value[0]
```

Rules a reader must enforce:

- All three subscripts are **required**. A missing one is a malformed fixture, not a default of
  zero — a fixture that meant zero says zero.
- A subscript past the end of its array is a malformed fixture.
- **Not every combination need be present.** The `post` list carries exactly the combinations the
  fixture makes a claim about, and a combination that is absent is not being asserted. Do not
  enumerate the cross-product and expect an entry for each.
- Give each run a name that includes its subscripts — `<case>[d2g0v0]` — or a failure report
  cannot say which payload diverged.

**Every index of a case runs against the same `pre` state**, independently. They are alternative
transactions against one world, never a sequence.

### Typed transactions and access lists

`accessLists` is an array **parallel to `data`**: entry *i* belongs to `data[i]`. A `null` entry
means *that index is a legacy transaction*.

```jsonc
"data":        ["0x",  "0x",  "0x"],
"accessLists": [null,  [],    [ { "address": "0x…3000", "storageKeys": [] } ]]
//              legacy  typed, typed, listing one address
//                      empty list
```

**Determine a transaction's type per index, and prefer `txbytes` where it exists.** The envelope
byte settles it with no convention at all: a `txbytes` beginning `0x01` is an EIP-2930 access-list
transaction, `0x02` is a fee-market one, and a first byte of `0xc0` or above is a legacy RLP list.
Fall back to the `accessLists` entry only when `txbytes` is absent.

> **Known divergence, and it is the reader's, not the fixture's.** `fukuii-cli`'s reader decides
> the type from the *presence of the `accessLists` key on the transaction object*, so every index
> of such a fixture is classified as typed — including a `null` one. Before Magneto that refuses
> the legacy control for its format, at a point where the fixture expects it to execute. Nothing
> in that repository carries an access list into execution either, so the pre-warming a listed
> address buys is not modelled at all. Both are consumer gaps rather than fixture defects, and
> they are recorded here because a new harness should not copy the first one.

### Two exception vocabularies, and neither is complete

A case that must be **refused** carries `expectException` instead of describing a result. Two
vocabularies are in circulation and no single name is understood by everything:

| vocabulary | example | where it comes from |
|---|---|---|
| **modern** | `TransactionException.TYPE_NOT_SUPPORTED` | the current spec-tests enum, `TransactionException` / `BlockException` |
| **legacy `TR_`** | `TR_TypeNotSupportedBlob` | retesteth-era. Rare — the whole archived tree holds 24 occurrences of exactly one such name |

**Several names alternate, bar-separated**, and that is the corpus's own convention, verified in
the vendored tree and in the fill side's test data:

```json
"expectException": "TransactionException.INSUFFICIENT_ACCOUNT_FUNDS|TransactionException.INTRINSIC_GAS_TOO_LOW"
```

The reader's contract:

1. Split on `|`, trim, keep the set **verbatim and unmapped**.
2. The runner intersects that set with the refusals *this build can produce*.
3. A non-empty intersection matching the actual refusal is a pass.
4. **An empty intersection is a divergence that names the unmatched rule** — never a skip, and
   never a pass. The point is to stop a case passing because a refusal for some *other* reason
   happened to leave the state root where the fixture expected it.

That contract cuts both ways, and this suite has been on the wrong end of it: a fixture here
stated `TR_TypeNotSupported`, a name that exists in no corpus and no client — the archived label
has a `Blob` suffix — so all twelve of its cases diverged on every run, and the divergence read
exactly like the client refusing for the wrong rule. **When you author a refusal, state a name
some consumer maps, and state more than one when the vocabularies do not overlap.** Never invent
one.

Refusal names in use in this suite today:

```
TransactionException.INVALID_SIGNATURE_VRS
TransactionException.TYPE_NOT_SUPPORTED | TransactionException.TYPE_1_TX_PRE_FORK
```

### A refused transaction and a state root

A refusal leaves the pre-state exactly as it was, so **the root cannot tell one refusal from
another** and the reason has to be compared separately, in both directions. Checking merely that
*some* refusal occurred is satisfied by any of them, including one for a rule the case is not
about.

### `txbytes` is the authority on the sender

No transaction carries a sender — the specification has no such field and derives one. Where a
fixture publishes `txbytes`, **those bytes decide**, and the `sender` field is a convenience. A
signature this fork refuses refuses the transaction rather than letting it run as whichever
account the file happens to name; a published signature that does not decode establishes nothing
and the case is unreadable rather than passing.

`txbytes` sits on the **post entry** rather than beside the transaction because each combination
of the arrays is signed separately and so has bytes of its own.

**This chain adopts replay protection at Die Hard**, so every upgrade before it requires a legacy
signature and refuses a protected one. Getting that wrong presents as an *empty result*, not an
error — which looks exactly like an opcode being unavailable.

---

## difficulty fixtures (`difficulty/`)

An existing format, and it nests differently from every other shape: an **outer name**, then
**fork labels**, then **case names**, then six scalars.

```jsonc
{
  "difficultyUncleAdjustment": {                 // outer name -- one per file here
    "_info": {
      "comment": "…",
      "unfillable": { "ETC_Thanos": "no fork name in the generator", … }
    },
    "ETC_Frontier": {                            // fork label
      "interval_1s_uncles_0": {                  // case name
        "parentTimestamp":    "1950000",
        "parentDifficulty":   "13107200",
        "parentUncles":       "0",
        "currentTimestamp":   "1950001",
        "currentBlockNumber": "150000",
        "currentDifficulty":  "13113600"         // THE EXPECTATION
      }
    },
    "ETC_Homestead": { … }
  }
}
```

**All six fields are required** and all six are quantities. Five are inputs; `currentDifficulty`
is the expected output.

`parentUncles` is a **flag, not a hash and not a count**: every case in every corpus writes `0` or
`1`, and it is read as a quantity and tested against zero. Do not parse it as a list or compare it
to a digest.

### The outer name exists for a mechanical reason

A reader loads the file as a map and hands each top-level **value** to its per-fork loop.
Flattening that level would make every field of every case look like a fork label. There is one
outer name per file here; a reader must accept several.

### This shape is read for *all* its forks at once

The opposite of a state fixture, and the difference belongs to the corpora rather than to taste. A
state fixture states expectations for many forks over one input, so asking it about one fork is
the whole question. A difficulty file states cases under each fork it covers and a caller naming
one could only either agree with the file or silently read nothing. **So a case naming a fork the
runner does not know is visible as a case** — counted and reported — rather than as an empty read.

`_info.unfillable` is where a file records the labels it deliberately carries no cases for, one
entry per label, with the reason. A reader skips it with the rest of `_info`; a human reads it to
tell a gap from an omission.

---

## fork-identifier vectors (`forkid/`)

**Specified here.** EIP-2124: the pair a node announces at a given head, so that two chains
sharing a genesis can refuse each other before they reach the block they disagree about.

```jsonc
{
  "forkIdentifierMainnet": {
    "_info": { … },
    "genesisHash": "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
    "forkBlocks": ["1150000", "2500000", … ],     // ASCENDING, and EXACTLY what the checksum covers
    "declinedForkBlock": "1920000",               // documented, and DELIBERATELY NOT in forkBlocks
    "vectors": [
      {
        "head":      "1150000",
        "forkHash":  "0x97c2c34c",                // CRC-32, 4 bytes, lower-case, zero-padded
        "forkNext":  "2500000",                   // the next fork block, or "0" past the last
        "forkIdRlp": "0xc98497c2c34c832625a0"     // RLP([forkHash, forkNext])
      }
    ]
  }
}
```

### How a harness checks a vector

### Use YOUR genesis hash, not the one in the file

**`genesisHash` here is a cross-check, not an input to trust.** Seed the checksum from the genesis
hash *your* client computes, then compare the resulting `forkHash` against this file. Doing it the
other way — taking our value and accumulating fork blocks onto it — makes every vector pass no
matter what genesis your client actually built.

That distinction is the whole test for one class of defect. A wrong genesis alloc, a wrong initial
difficulty, a wrong `extraData` — none of them is visible in a fork identifier computed from a
supplied hash, and **all of them change the hash a real node derives**. Seed from your own and the
error surfaces at the very first vector, where `head` is 0 and no fork block has been folded in yet.

**If they disagree, stop rather than continuing.** A genesis mismatch is not a fork-identifier bug
and will not be fixed by anything in this file; every later vector is then noise. Compare our
`genesisHash` against yours directly and report *that*.

The same holds for `forkBlocks`: read them from your own chain configuration and check them against
this file. A client that takes both the genesis and the schedule from the fixture is testing our
arithmetic against itself.


```
crc = CRC32(genesisHash)                      # 32 raw bytes, not the hex text
for f in forkBlocks:
    if f <= head: crc = CRC32(uint64_be(f), crc)
    else:         forkNext = f; break
else:             forkNext = 0
forkHash  = crc as 4 big-endian bytes
forkIdRlp = RLP([ forkHash_as_4_bytes, forkNext_as_minimal_big_endian_bytes ])
```

`forkNext` of zero encodes as the **empty byte string** in the RLP, not as `0x00`.

`forkIdRlp` is redundant with the other two fields and is carried on purpose: it is where a
harness with a working checksum and a broken encoder finds out, and the wire format is the thing a
peer actually sees.

### `declinedForkBlock` is the whole point of this shape

This chain refused Ethereum's DAO fork. That refusal is not a state-transition rule and no state
fixture can reach it — but it is observable, because **the identifier is a checksum over exactly
the blocks in `forkBlocks`**, and adding the declined block would produce a hash no node on this
network computes.

So the number is recorded, in its own field, and **must never join `forkBlocks`**. Documenting the
rejection and excluding it from the checksum are the same position stated at two layers, not a
tension.

The vectors at `1919999`, `1920000` and `1920001` exist to assert that **nothing happens there**.
The identifier is unchanged across all three. That is a claim a client cannot make in its
configuration file: the production client expresses the rejection as a commented-out line, which
is invisible to every tool.

### Why this shape carries no upgrade key

An identifier is a checksum over the *whole* schedule evaluated at a head, so there is no
per-upgrade expectation to carry; the vector list is indexed by head. It is also the only shape
here that can assert something about an upgrade that changes no state-transition rule at all — a
fork block participates in the checksum whether or not a transaction can observe it. One upgrade
in this schedule is covered by nothing else in the suite for exactly that reason.

---

## block-level vectors (`blocks/`)

**Specified here.** Block-level rules are functions of height, with no per-upgrade expectation to
carry, so this shape is indexed by block number rather than keyed by upgrade.

> **`blocks/` holds several schemas and the outer key is what says which.** A type directory here
> groups files by the **reader code path** they need, and each file's single outer key names its
> own schema; block-level files sitting side by side share no fields beyond `_info`. A harness
> dispatches on that outer key and must not assume the directory is homogeneous.
>
> **The sections below describe the schemas; the corpus says which exist.** This paragraph counted
> them at four and there were eight by the time anyone checked — run the second command under
> "The inventory is the tree" and read `blocks` off it. A schema in that output with no section
> here is a gap in this file. Note also that one schema can appear under more than one outer key,
> once per network (`eraEmissionSchedule` and `eraEmissionScheduleMordor` are one reader path),
> so the key count is an upper bound on the number of reader paths, never the number itself.

### `eraEmissionSchedule` — what a block pays

```jsonc
{
  "eraEmissionSchedule": {
    "_info": { … },
    "eraLength":         "5000000",
    "activationBlock":   "5000000",
    "maxOmmersPerBlock": "2",
    "ancestorWindow":    "7",
    "validOmmerDistances": ["1","2","3","4","5","6"],
    "vectors": [
      {
        "block": "1", "era": "0",
        "winnerBaseReward":       "5000000000000000000",     // wei
        "includerBonusPerUncle":  "156250000000000000",
        "uncleMinerRewardByDistance": { "1": "…", "2": "…", …, "6": "…" },
        "uncleRewardDependsOnDistance": true,                // JSON boolean
        "totalIssuance": {
          "noUncles":                "…",
          "oneUncleByDistance":      { "1": "…", … },
          "twoUnclesBothAtDistance": { "1": "…", … }
        },
        "maxIssuance": "…",
        "maxIssuanceOverBaseReward": "2.812500"              // DECIMAL TEXT, see below
      }
    ],
    "observedMainnet": { … }
  }
}
```

**Every wei value is an exact integer as a decimal string.** The single exception is
`maxIssuanceOverBaseReward`, which is a **ratio rendered to six decimal places as text** — it is
there for a human reading the file, and a harness should recompute it rather than parse it. The
same is true of `requiredMultipleOfLocalTd` in the chain-selection shape.

`era` is **zero-based**, and the boundary is the trap: **the reduction lands at the boundary block
plus one, never at the boundary.** Block 5,000,000 still pays era 0; block 5,000,001 begins era 1.
A widely used block explorer gets this wrong on four blocks, which is recorded in
`_info` with the archive-state evidence.

`validOmmerDistances` and `maxOmmersPerBlock` are GHOST validity, not emission, and they are here
because **reward and validation have to agree or a client mints currency no other client mints**.
The rule this shape covers changes ommer *accounting* and lowers the block reward as a
consequence, rather than the reverse.

### `observedMainnet` — the chain's own record, above every client

A sub-object recording readings taken from archive state, not computed by anything:

```jsonc
"observedMainnet": {
  "method": "Miner balance delta across a single block, read from archive state",
  "source": "an archive node serving eth_getBalance at historical heights",
  "emptyBlockBaselines": {
    "note": "Zero transactions and zero ommers, so the delta IS the static reward",
    "blocks": [ { "block": "4900002", "era": "0",
                  "observedDeltaWei": "5000000000000000000", "matchesComputed": true } ]
  },
  "eraBoundaries": { … },
  "ommerPayments": { … },
  "explorerDiscrepancy": { … }
}
```

`balance(miner, N) - balance(miner, N-1)` at an archive node, on a block with **zero transactions
and zero ommers**, *is* the static reward: no fee component to subtract and nothing else moving.
Two RPC calls, and it outranks every client including the production one.

Two disciplines this sub-object exists to enforce:

- **Resolve identifiers rather than inferring them.** An ommer's reward implies its distance under
  the era-0 formula, so reading the distance off the reward and then declaring the reward correct
  proves nothing. Every `ommerPayments` entry must carry an `ommerHeight` resolved from the
  ommer's own hash.
- **A block explorer is an implementation, not an observation.** It computes rewards from its own
  configuration. A disagreement with one is a cheap signal that something needs checking, and
  never the chain.

---

### `requiredBlockHeaders` — what a header at a height must be

```jsonc
{
  "requiredBlockHeaders": {
    "_info": { … },
    "requiredHashes": [ { "block": "1920000", "hash": "0x9436…", "role": "…", "note": "…" } ],
    "vectors": [ { "block": "1920000", "hash": "0x…", "accepted": true,
                   "hashTakenFrom": "…", "note": "…" } ],
    "declinedForkExtraData": {
      "marker":     "0x64616f2d686172642d666f726b",
      "markerText": "dao-hard-fork",
      "range":      { "from": "1920000", "to": "1920009" },
      "mustCarryMarker": false,
      "observed":   [ { "block": "1920000", "extraData": "0x…",
                        "carriesMarker": false, "miner": "0x…" } ]
    }
  }
}
```

`requiredHashes` is the rule: a block at one of these heights **must** hash to exactly that value,
and a chain presenting anything else there is refused however much difficulty it carries.

`vectors` exercises it, and reading them needs one rule that is easy to get backwards: a vector at
a height with **no** requirement carries `accepted: true`. Those exist so a harness cannot pass the
file by refusing everything — a check that refuses every block also refuses every wrong one.
Counterexamples are **real hashes of real neighbouring blocks presented at the wrong height**,
never fabricated values, because a client validating only the *shape* of a hash passes a fabricated
one for the wrong reason.

`declinedForkExtraData` is an **observation, not a rule this suite invents**. It records what the
chain actually carries across Ethereum's DAO block range, and `mustCarryMarker: false` is the claim
a client has to satisfy: a client that ported Ethereum's DAO header validator unchanged requires
the marker, rejects all ten of these blocks, and cannot sync past 1,920,000 — a failure that
presents as a corrupt database rather than as a misconfigured fork.

### `ommerPaymentVectors` — what a block shape credits, and to whom

```jsonc
{
  "ommerPaymentVectors": {
    "_info": { … },
    "eraLength": "5000000", "maxOmmersPerBlock": "2",
    "validOmmerDistances": ["1","2","3","4","5","6"],
    "vectors": [
      { "name":      "era_boundary_first_block_of_era_1",
        "grounding": "observed",                         // or "computed"
        "block":     { "number": "5000001", "coinbase": "0x…" },
        "ommers":    [ { "number": "5000000", "distance": "1", "coinbase": "0x…" } ],
        "expectedCredits": { "0x<address>": "<wei>" },   // KEYED BY ADDRESS, VALUES ARE TOTALS
        "derivedIntermediates": { "era": "1", "winnerReward": "…",
                                  "includerBonusPerOmmer": "…", "ommerRewards": ["…"] },
        "observed":  { "transactionsInBlock": "6", "includerCreditIsExact": false, "note": "…" },
        "note":      "…" }
    ]
  }
}
```

A harness builds the block, applies the reward rules, and compares the balance delta of every
address in `expectedCredits`.

**`expectedCredits` is keyed by address and its values are TOTALS.** An address receiving two
payments in one block appears **once**, holding their sum. That is not a convenience of the file
format — the client credits with an add rather than a write, and an ommer's miner may also be the
including block's miner. One vector exists solely to separate those readings, and both wrong
answers it distinguishes appear as legitimate values elsewhere in the file.

`grounding` says whether a vector is a real mainnet block (`observed`) or a constructed shape
(`computed`, with obviously synthetic addresses). For an observed vector,
**`observed.includerCreditIsExact` says which side of the block to trust**: transaction fees land
in the includer's balance and nowhere else, so an ommer miner's credit is exact whether or not the
block carried transactions, while the includer's is exact only at zero.

`derivedIntermediates` is what a correct harness computes on the way — published so a failing
consumer lands on the step. A harness that reads those fields instead of deriving them asserts
nothing.

**This file is payments; admissibility is `eraEmissionSchedule`'s.** Whether an ommer may be
included at all — at most two, within seven ancestors, not already included — is validation rather
than emission. The two have to agree or a client mints currency no other client mints, which is why
both are stated rather than one inferred from the other.

### `ommerValidityRules` — which ommers a block may include at all

```jsonc
{
  "ommerValidityRules": {
    "_info": { },
    "maxOmmersPerBlock":   "2",
    "ancestorWindow":      "7",
    "validOmmerDistances": ["1","2","3","4","5","6"],
    "vectors": [
      { "name": "…", "blockNumber": "6000000",
        "ommers": [ { "number": "5999999", "distance": "1", "id": "A" } ],
        "valid": true,
        "rejectionClass": "tooManyUncles",
        "note": "…" }
    ]
  }
}
```

**Admissibility, not payment.** What an included ommer *earns* is `ommerPaymentVectors`'s; whether it
may be included at all is here. The two have to agree or a client mints currency no other client
mints, which is why both are stated rather than one inferred from the other.

`valid` is a real JSON boolean. **`rejectionClass` appears only where `valid` is false**, and names
which rule the shape violates — too many ommers, a duplicate, an ancestor presented as an ommer, a
distance outside the window. **Compare the class, not merely that the block was refused**: a client
that rejects every ommer-bearing block passes a file checked only for refusal.

`id` labels an ommer within its vector so a duplicate can be expressed at all; it is not a hash and
a reader derives nothing from it.

### `daoIrregularStateChange` — a scripted mutation, and the one thing here that is not a rule

```jsonc
{
  "daoIrregularStateChange": {
    "_info": { },
    "block":       "1920000",
    "beneficiary": "0x…",
    "headerMarker": { "marker": "0x…", "markerText": "dao-hard-fork",
                      "range": { "from": "1920000", "to": "1920009" },
                      "mustCarryMarker": true },
    "drainedAccounts": [ "0x…", "0x…" ]
  }
}
```

**No opcode implements this, no gas is charged for it and no schedule of EVM rules can express
it.** At one height, on one network, the balance of every account in `drainedAccounts` moves in
full to `beneficiary`. A harness applies it as a scripted state mutation at `block` and compares
balances afterwards.

**`beneficiary` is deliberately not in `drainedAccounts`**, and a reader that merges the two lists
drains the destination.

**`headerMarker` is a SEPARATE rule from the state change** and is stated separately: across the
range, a header must carry the marker. Note the polarity — `mustCarryMarker: true` here, against
`false` in Ethereum Classic's `requiredBlockHeaders.declinedForkExtraData`, which asserts the
mirror image on the chain that refused this fork. The two files are a matched pair and reading
either alone gets the sign wrong half the time.

**There is no list of drained accounts to shorten.** Its length is the assertion: a client that
applies the mutation to a subset produces a state root nothing else computes, and the file exists
because no `state/` fixture can reach a change that no transaction causes.

### `receiptStatusEncoding` — what a receipt's first field is

```jsonc
{
  "receiptStatusEncoding": {
    "_info": { … },
    "activationBlock":  "8772000",
    "activationUpgrade": "ETC_Atlantis",
    "emptyTrieRoot":    "0x56e81f17…",
    "vectors": [
      { "name":                 "boundary_post_one_successful_transaction",
        "blockNumber":          "8772000",
        "receiptCarriesStatus": true,              // the rule flag, stated not inferred
        "receipts": [ { "firstFieldKind":    "status",      // or "postStateRoot"
                        "firstField":        "1",           // or "0x<32 bytes>"
                        "succeeded":         true,
                        "cumulativeGasUsed": "21000",
                        "logCount":          "0",
                        "logsBloom":         "0x…" } ],
        "encodedReceipts": [ "0x…" ],              // rlp of each receipt, in order
        "receiptsRoot":    "0x…",
        "note":            "…" }
    ],
    "observedMainnet": { … }
  }
}
```

A receipt encodes as `rlp([postStateOrStatus, cumulativeGasUsed, logsBloom, logs])` on **both**
sides of the activation — only the first field changes, and everything derived from it. Before
Atlantis it is a 32-byte intermediate post-state root, so a receipt cannot say whether its
transaction succeeded; from Atlantis it is `1` or `0`.

The `receiptsRoot` is a Merkle-Patricia root over `rlp(index) → encoded receipt`.

**Vectors come in PAIRS** — the same execution at `8771999` and at `8772000`. Only the pair shows
that nothing but the encoding moved: same gas, same logs, same outcome, different root. A client
writing the wrong form produces a header no other node accepts.

**`receiptCarriesStatus` is published rather than left implicit** so a harness can check its own
schedule resolution against the fixture instead of inferring the rule from the answer it is trying
to verify.

**One vector must NOT move**: a block with no transactions has the empty-trie root either side of
the boundary. A harness whose root changes there is deriving it from the rule rather than from the
receipts.

## chain-selection vectors (`chainselection/`)

**Specified here.** Whether a proposed reorganization is accepted. The only rule in the schedule
that decides between two competing chains rather than computing something about one.

Several vector lists, and **each is a different claim** — a harness may implement them
independently and should report them separately. Read which exist off the file rather than off
this sentence; the block below shows the ones it carried when this was written:

```jsonc
{
  "messArtificialFinality": {
    "_info": { … },
    "activationBlock":   "11380000",
    "deactivationBlock": "19250000",
    "curveDenominator":  "128",
    "curveXCap":         "25132",
    "curveAmplitude":    "15",
    "curveHeight":       "3840",

    "windowVectors":   [ { "block": "11380000", "active": true } ],

    "curveVectors":    [ { "timeDeltaSeconds": "12566", "curveNumerator": "2048",
                           "requiredMultipleOfLocalTd": "16.000000" } ],

    "decisionVectors": [ { "commonAncestorTimestamp": "1000000",
                           "localHeadTimestamp":      "1012566",
                           "proposedHeadTimestamp":   "1012566",
                           "localSubchainTd":         "1000",
                           "proposedSubchainTd":      "16000",
                           "rejected":                false,
                           "timeDeltaSeconds":        "12566",
                           "curveNumerator":          "2048",
                           "note":                    "…" } ],

    "subchainVectors": [ { "name": "…",
                           "commonAncestor":  { "number": "…", "timestamp": "…",
                                                "totalDifficulty": "…" },
                           "localSegment":    [ { "number": "…", "timestamp": "…",
                                                  "difficulty": "…" } ],
                           "proposedSegment": [ … ],
                           "rejected": false,
                           "derivedIntermediates": { "timeDeltaSeconds": "…",
                                                     "curveNumerator": "…",
                                                     "localSubchainTd": "…",
                                                     "proposedSubchainTd": "…" },
                           "note": "…" } ]
  }
}
```

**`windowVectors`** — is the rule in force at this height? The window is **inclusive at the start
and exclusive at the end**: the bundled default configuration enables it from `activationBlock` up
to but *not including* `deactivationBlock`. Both edges are covered because either could be off by one
without the other
showing it.

**`curveVectors`** — the polynomial alone, with no header attached to its input:

```
xcap   = 25132                       # floor(8000 * pi)
ampl   = 15
height = CURVE_FUNCTION_DENOMINATOR * (ampl * 2)      # 3840
x      = min(time_delta, xcap)
numerator = CURVE_FUNCTION_DENOMINATOR + (3*x**2 - 2*x**3 // xcap) * height // xcap**2
```

**The integer divisions and their order are part of the rule**, not an implementation detail: they
truncate, and floating point does not reproduce them.

**`decisionVectors`** — the comparison, which is **strict**:

```
rejected  ==  proposed_subchain_td * CURVE_FUNCTION_DENOMINATOR
                  <  numerator(time_delta) * local_subchain_td
```

so a proposal matching the threshold **exactly** is accepted. Every threshold in the file is
pinned by a pair — the exact match and the value one below it — because a non-strict comparison
passes every test that does not sit on the boundary.

> ### The curve's `x` is the LOCAL head's age, and this is the trap
>
> `x = local_head.timestamp - common_ancestor.timestamp`. **Not the proposed head's.**
>
> The comment sitting directly above the production client's own implementation says
> `proposed.Time - commonAncestor.Time`, and the line beneath it reads `current.Time -
> commonAncestor.Time`. ECIP-1100 reproduces that stale comment verbatim in its non-normative
> listing of the same function, while its own **normative** pseudocode says `current.Time`. The
> client's error message prints one quantity under two labels, `current.span` and `proposed.span`.
>
> All four implementations read at their call sites on 2026-08-24 take the local head.
>
> This is why `decisionVectors` carries **all three timestamps** rather than one derived delta: a
> harness has to choose. Two vectors are marked `DISCRIMINATOR` in their `note`, and for those the
> two choices produce **opposite verdicts** — in both directions, so that pinning it one way
> cannot be satisfied by loosening the comparison instead of by reading the right header.

**`subchainVectors`** — that a client *derives* the three scalars correctly from two competing
segments: summing difficulty rather than counting blocks, measuring from the common ancestor
rather than from genesis, taking the age off the local head.

A segment entry carries a **number, a timestamp and a difficulty and nothing else**. No parent
hash, no state root, no nonce, no transactions. That is deliberate and it is what makes these
portable: the decision reads exactly those fields, so supplying more invites a harness to validate
the chain instead of the rule, and supplying real blocks would make every consumer mine
proof-of-work to run a test.

`derivedIntermediates` is what a correct harness **computes**. It is published so a failing
consumer can see which step went wrong. **A harness that reads those fields instead of deriving
them asserts nothing.**

### What no vector in this shape reaches

Presenting a client with two real competing chains and watching it decline to reorganize. That
needs a chain-level runner. See "What is not covered" below.

---

### This shape is the one POLICY in the suite, not a consensus rule

**Every other shape here asserts something a chain either does or is invalid. This one does not,
and a harness that treats it the same way will fail correctly-configured nodes.**

ECBP-1100 is a local chain-selection defense. A node running with it switched off produces a
perfectly valid chain — it simply accepts reorganizations a defended node would refuse. **Nothing
in a block header records whether its producer had MESS on**, which is the structural reason this
can never be a consensus rule.

Two consequences for a reader:

- **`activationBlock` and `deactivationBlock` are the window in which the client's *bundled
  configuration* turns it on.** They are not activations in the sense every other height in this
  suite is. In the reference client all three of `--ecbp1100`, `--override.ecbp1100.deactivate` and
  `--ecbp1100.nodisable` override them, and the client additionally has auto-shutoff mechanisms of
  its own — so "inside the window" is not a prediction about any particular node.
- **The gate has two layers and both must hold.** The configuration window must be open *and* the
  runtime toggle must be on. The reference client documents the runtime layer as sitting *below*
  the configuration: enabling it at runtime while the config has not activated it is a no-op, and
  the enabled-status is tracked separately and described as "agnostic of feature activation by
  chain configuration".

**Assert the decision given the rule is in force; never assert that a node must refuse a
reorganization.** The curve, the comparison and the subchain derivation are wrong whenever they run
if they are wrong at all, and that is the useful thing to certify. `windowVectors`' `active` field
means *the bundled default has it enabled at this height* — read as "a conformant node refuses
reorgs here", it will fail a node that is merely configured differently.

## proof-of-work epoch vectors (`pow/`)

The published `PoWTests` tier is a **flat map from case name to case**, with no fork level and no
state anywhere in it: there is no fork to resolve, because a seal's algorithm belongs to the engine
and no fork selects one. `_info` is skipped as everywhere else.

```jsonc
{
  "<case name>": {
    "header":      "f901f3a0…",      // the whole sealed header as RLP
    "nonce":       "4242424242424242",
    "mixHash":     "58f759ed…",
    "seed":        "00000000…",
    "result":      "dd47fd2d…",
    "header_hash": "2a8de2ad…",      // the seal hash: the header minus its last two elements
    "cache_hash":  "35ded12e…",
    "cache_size":  16776896,          // A JSON NUMBER, not a string
    "full_size":   1073739904         // likewise
  }
}
```

Two things about that tier catch a reader out, and both are properties of the published data:

- **Its hex carries no `0x` prefix**, alone among every tier here. A reader tolerant of both is
  fine; one that requires the prefix skips every case.
- **`cache_size` and `full_size` are JSON numbers** where every other field in the file is a hex
  string. A reader reaching for the string path gets a decode failure per case, which counts as a
  skip and therefore as neither agreement nor divergence.

The intermediates are why the tier is worth more than its case count: a case states the seed, the
cache size, the dataset size and the seal hash beside the answer, so a divergence lands on the step
that caused it rather than on the digest at the end.

### And the tier covers epoch 0, and nothing else

Two cases, both Ethereum's, both at the first epoch — `cache_size` 16776896 and `full_size`
1073739904 in both. It confirms the base of the schedule and says nothing whatever about any epoch
above the first, and therefore nothing about **ECIP-1099**, which is this chain's only change to
proof-of-work and the one activation in its mainnet configuration that no other fixture here
reaches.

### The extension: `pow/etchash_epoch_schedule.json`

A sealed header cannot be authored, because it requires a mined nonce. The **epoch-derived
quantities** can be, and they are exactly what ECIP-1099 changes — each is a pure function of the
block number, verifiable by hand:

```jsonc
{
  "etchashEpochSchedule": {
    "_info": { … },
    "activationBlock":      "11700000",
    "epochLengthBefore":    "30000",
    "epochLengthFrom":      "60000",
    "seedIterationDivisor": "30000",
    "vectors": [
      { "block":            "11700000",
        "epochLength":      "60000",
        "epoch":            "195",
        "epochStartBlock":  "11700001",
        "seedHash":         "0x…",
        "seedIterations":   "390",
        "cacheSizeBytes":   "…",
        "datasetSizeBytes": "…",
        "note":             "…" }
    ]
  }
}
```

Field names follow the published tier where it has one: `seedHash` is its `seed`, `cacheSizeBytes`
its `cache_size`, `datasetSizeBytes` its `full_size`. Everything is a **decimal string** here
rather than a JSON number, for consistency with every other shape this repository authors.

```
epochLength(n)   = 60000 if n >= 11_700_000 else 30000
epoch(n)         = n // epochLength(n)
epochStart(e, L) = e * L + 1
cacheSize(e)     = 2**24 + 2**17 * e - 64   reduced by 128 while (size // 64)  is not prime
datasetSize(e)   = 2**30 + 2**23 * e - 128  reduced by 256 while (size // 128) is not prime
```

**At the activation the epoch length doubles, so the epoch number HALVES** — 389 to 195 — and the
dataset requirement falls back to what it was at roughly half the height. That is the entire point
of the proposal, and a client that keeps the old length computes epoch 390 there and asks for a
dataset nobody else is using. It does not diverge subtly: it rejects every block after the
activation.

> ### The seed's iteration count does NOT use the epoch length
>
> ```
> seed(e, L) = keccak256 applied  (epochStart(e, L) // 30000)  times to 32 zero bytes
> ```
>
> The divisor is the **default** epoch length, 30000, whatever length is actually in force — and it
> is not the epoch number either. Across the activation that gives **389** iterations for epoch 389
> and **390** for epoch 195, so the two seeds differ and no epoch reuses another's.
>
> An implementation that divided by the length in force would compute 195 iterations for the first
> post-activation epoch and **collide with a seed already used at block 5,850,000**. `seedIterations`
> is published beside each seed so a consumer failing this lands on the step, not on the digest.

## proposal-algorithm vectors (`components/proposals/<series>/<proposal>/`)

**Specified here.** A proposal's rule tested as a **pure function of supplied inputs** — no chain
id, no upgrade labels, no state root. The type directory beneath the proposal is the same one a
network uses, so a harness resolves by type before it resolves by subject.

**This is the component layer, and it inverts a rule the network layer states.** Read
`components/README.md` before using one.

### `eip2124ForkIdentifierAlgorithm` — the checksum, with the chain supplied rather than derived

```jsonc
{
  "eip2124ForkIdentifierAlgorithm": {
    "_info": { },
    "algorithm": {
      "checksum":   "CRC32-IEEE",
      "seed":       "the 32 raw bytes of the genesis hash",
      "fold":       "for each fork block <= head, ascending: crc = CRC32(uint64_be(fork), crc)",
      "forkHash":   "the resulting checksum as 4 big-endian bytes",
      "forkNext":   "the first fork block > head, else 0",
      "forkIdRlp":  "RLP([forkHash as 4 bytes, forkNext as a minimal big-endian integer])"
    },
    "vectors": [
      { "name": "…", "comment": "…",
        "genesisHash": "0x…", "forkBlocks": ["100","200"], "head": "100",
        "forkHash": "0x…", "forkNext": "200", "forkIdRlp": "0x…",
        "tags": [ "fold-boundary" ] }
    ]
  }
}
```

> ### `genesisHash` is an INPUT here, and that is the opposite of the network shape
>
> `forkid/` under a network says: **use YOUR genesis hash, never the one in the file**, because
> seeding from a supplied hash makes every vector pass whatever genesis your client actually built.
> That is right there and wrong here. This shape's subject is the checksum arithmetic, so the chain
> is supplied on purpose.
>
> **It therefore cannot detect a wrong genesis, a wrong alloc or a wrong schedule, and does not
> substitute for the network fixture.** Run both: this one says your arithmetic is right, that one
> says your chain is.

**All three outputs are asserted**, because a client can have the checksum right and the encoder
wrong. `forkNext` of zero is the **empty byte string** in the RLP, never a zero byte.

**`forkBlocks` is taken in the order given and is not re-sorted by the reader.** A fixture supplies
it ascending; a client that sorts defensively passes anyway, and one that folds out of order does
not.

---

## consensus-mechanism vectors (`consensus-algorithms/`, and `consensus/` under a network)

**Specified here.** How a chain decides *who may produce a block, and which block wins* — rules
that are not state transitions and that no `state/`, `blocks/` or `difficulty/` fixture can reach.

**These schemas live in two places, and the split is the subject rather than the shape.** A rule
that belongs to a **mechanism** holds wherever that mechanism runs and sits under
`components/consensus-algorithms/<class>/<algo>/` — Clique's difficulty rule is the same rule on
every Clique chain. A rule keyed to **one network's own constants** sits under
`networks/<family>/<network>/consensus/` — when a particular chain stopped doing proof of work is a
fact about that chain. Read `components/README.md` for the test.

> **The mechanism schemas moved on 2026-08-27**, from `proposals/eip/eip-225/consensus/` and
> `proposals/eea/qbft-v1/consensus/`, and again under `components/` the same day. Clique had been filed as an improvement proposal because it
> has an EIP number, which no other mechanism does; a path is not a citation, and EIP-225 is
> recorded in that fixture's `_info` where it belongs.

**Like `blocks/`, this directory is heterogeneous and the outer key selects the schema.** The
schemas share no fields beyond `_info`, and a reader that handles one has done nothing toward
another. Derive which exist with the second command under "The inventory is the tree"; the
sections below are the contract for each.

> ### A quantity here may be a JSON number, and that is a divergence to tolerate
>
> "Conventions every shape shares" requires every quantity to be a JSON **string**. The Clique
> schemas below carry `epoch`, `period`, `extraVanityBytes` and `extraSealBytes` as JSON
> **numbers**; the QBFT schema carries every quantity as a string, per the convention. **Accept
> both wherever this directory states a quantity.** The divergence is recorded rather than
> smoothed over because a reader that only accepts one form fails on half the directory, and the
> failure surfaces as a parse error rather than as a disagreement.

### `cliqueSealDifficulty` — the in-turn / out-of-turn difficulty rule, as accept/reject pairs

```jsonc
{
  "cliqueSealDifficulty": {
    "_info": { },
    "chainConfig": { "headerFieldCount": 16, "activatedAtBlockZero": [ "London", … ] },
    "diffInTurn": "0x2",
    "diffNoTurn": "0x1",
    "keyDerivation": "keccak256(\"fukuii-tests/eip-225/\" + label)",
    "vectors": [
      { "name": "…", "comment": "…",
        "epoch": 30000, "period": 1,
        "genesisSignerLabels": ["A","B","C"],
        "genesisSigners":      ["0x…","0x…","0x…"],
        "genesis":        { "…full header…", "hash": "0x…", "sealHash": "0x…", "rlp": "0x…" },
        "prefixHeaders":  [ { "…full header…" } ],
        "acceptedHeader": { "…full header…" },
        "rejectedHeader": { "…full header…" },
        "expectedError":  "…", "direction": "…" }
    ]
  }
}
```

**A vector is a PAIR and the accepted half is the load-bearing one.** A client that refuses every
block scores perfectly against rejection vectors alone. The two headers differ in `difficulty` and
in everything downstream of it — `extraData`, because the seal covers a different header, and
therefore the hashes and the RLP. That cascade is expected and is not a second variable.

**Import the two halves onto separate chains.** They are siblings at the same height, not a
sequence: build the genesis from `genesisSigners`, confirm `genesis.hash`, replay `prefixHeaders`
if present, then present `acceptedHeader` to one chain and `rejectedHeader` to another.

**`chainConfig` is not derivable from the vectors and must be read first.** These headers carry a
specific field count, and a client configured outside that window fails at RLP decode — which
reads as a broken fixture rather than as a misconfigured harness.

### `cliqueSignerVoting` — the authorized signer set, derived from a header chain

```jsonc
{
  "cliqueSignerVoting": {
    "_info": { },
    "chainConfig": { },
    "keyDerivation": "…",
    "extraVanityBytes": 32, "extraSealBytes": 65,
    "nonceAuthVote": "0xffffffffffffffff", "nonceDropVote": "0x0000000000000000",
    "vectors": [
      { "name": "…", "comment": "…", "epoch": 30000, "period": 1,
        "genesisSignerLabels": [ "A" ], "genesisSigners": [ "0x…" ],
        "genesis": { "…full header…" },
        "headers": [ { "…full header…" } ],
        "expectedSignerLabels": [ "A", "B" ],
        "expectedSigners":      [ "0x…", "0x…" ],
        "expectedFailure": "" }
    ]
  }
}
```

A harness seeds a chain from `genesis`, imports `headers` in order, and compares the signer set it
derives against `expectedSigners`.

**`expectedFailure` is an assertion when non-empty and says nothing when empty.** A non-empty value
names the condition the chain must be refused for — an unauthorized signer, a signer that signed
too recently. **Compare the reason, not merely that some refusal happened**, for the same reason a
refused state transaction needs its reason compared: any refusal leaves the same observable state,
including a refusal for a rule the vector is not about.

**The labels exist so a failure report can name a signer.** `expectedSignerLabels` is parallel to
`expectedSigners` and carries no information a reader needs; it is there so a human reading a
divergence sees `B` rather than a second 20-byte address.

### `ecip1010DifficultyBombPause` — the bomb's reference point, frozen then resumed

One vector list. `pauseBlock` and `continueBlock` are per-vector **inputs**, so the same file
certifies any chain adopting the rule.

```jsonc
{
  "ecip1010DifficultyBombPause": {
    "_info": { }, "constants": { "expDiffPeriod": "100000",
                                 "bombTermIsZeroUnlessPeriodCountExceeds": "1" },
    "explosionReferenceVectors": [
      { "name": "…", "blockNumber": "4000000",
        "pauseBlock": "3000000", "continueBlock": "5000000",
        "explosionReference": "3000000",     // ASSERTED -- the client's own output
        "periodCount": "30", "bombTerm": "268435456" }   // DERIVED, see below
    ]
  }
}
```

The rule: below the pause the reference is the block itself; from the pause until the continue
block it is frozen at the pause block; from the continue block on it is `blockNumber - (continue -
pause)`.

**The resume is continuous, and that is the assertion.** At the last block of the window the
reference is the pause block; at the continue block it is the same value again. The bomb picks up
where it stopped rather than catching up, and the two vectors either side of that boundary are what
separate a correct resume from a client that lets it jump.

**`explosionReference` is asserted; `periodCount` and `bombTerm` are derived.** The reference is the
client's own output. The other two are computed from the published `2**((reference // 100000) - 2)`
by the generator and cross-checked against a separate transcription, so they are two agreeing
readings of a formula rather than a third execution of it. Publish-and-derive, exactly as
`derivedIntermediates` elsewhere: a harness that reads them instead of computing them asserts
nothing.

> **A boundary mutation here is inert, and it is worth knowing why before reporting it as a gap.**
> Changing the window test from `<` to `<=` produces no wrong answer at any height, because the two
> branches coincide at the continue block by exactly the continuity property above. A
> changed-and-compiling patch that catches nothing usually means a coverage gap; here it meant an
> empty patch.

### `ecip1017EmissionSchedule` — Ethereum Classic's monetary policy as pure functions

**Four vector lists, four separate claims**, with `eraLength` and `baseReward` as per-vector
**inputs** rather than constants.

```jsonc
{
  "ecip1017EmissionSchedule": {
    "_info": { }, "constants": { "disinflationRateQuotient": "4", "disinflationRateDivisor": "5",
                                 "ommerDistanceDivisorEraZero": "8",
                                 "ommerRewardDivisorFromEraOne": "32", … },
    "eraVectors":           [ { "name": "…", "block": "5000000", "eraLength": "5000000",
                                "era": "0" } ],
    "winnerRewardVectors":  [ { "name": "…", "era": "21", "baseReward": "5000000000000000000",
                                "winnerReward": "46116860184273879" } ],
    "uncleRewardVectors":   [ { "name": "…", "era": "0", "blockNumber": "…", "uncleNumber": "…",
                                "distance": "…", "baseReward": "…", "uncleReward": "…" } ],
    "includerBonusVectors": [ { "name": "…", "era": "…", "uncleCount": "1", "baseReward": "…",
                                "includerBonus": "…" } ]
  }
}
```

**An era begins at the boundary block PLUS ONE.** Block 5,000,000 still pays era 0 at an era length
of 5,000,000; era 1 starts at 5,000,001. A widely used block explorer gets this wrong, which the
network-scoped emission fixture records against archive state.

**The reward is one division, not a repeated step.** `winnerReward = baseReward * 4**era / 5**era`,
truncating **once** at the end. Applying a 4/5 step per era truncates every time; the two agree
through era 20 and first differ at era 21, by a single wei. Both eras are present, and the *pair* is
the assertion.

**The ommer rule changes shape at era 1.** Era 0 pays `(8 - distance)/8` of the base reward, so
distance matters; from era 1 it pays `winnerReward/32` and distance stops mattering — which is why
two era-1 vectors at different distances carry the *same* value. The includer's bonus is
`winnerReward/32` per ommer, derived from the **era-adjusted** reward, so it decays too.

### `etchashEpochFunctions` — ECIP-1099's epoch arithmetic as pure functions

**Four vector lists, and they are four separate claims.** A client may implement any three
correctly and the fourth wrong, so a harness should report them separately.

```jsonc
{
  "etchashEpochFunctions": {
    "_info": { },
    "constants": { "cacheInitBytes": "16777216", "epochLengthDefault": "30000",
                   "epochLengthEcip1099": "60000", "bundledTableCoversEpochsBelow": "2048",
                   "seedIterationDivisor": "30000", … },

    "epochLengthVectors": [ { "name": "…", "block": "11700000",
                              "activationBlock": "11700000",     // null means never adopted
                              "epochLength": "60000" } ],

    "epochVectors":       [ { "name": "…", "block": "…", "epochLength": "…",
                              "epoch": "…", "epochStartBlock": "…" } ],

    "sizeVectors":        [ { "name": "…", "epoch": "2048",
                              "servedFromBundledTable": false,   // JSON boolean
                              "cacheSizeBytes": "…",   "datasetSizeBytes": "…",
                              "cacheSizeFromFormula": "…", "datasetSizeFromFormula": "…" } ],

    "seedVectors":        [ { "name": "…", "epoch": "195", "epochLength": "60000",
                              "seedIterations": "390", "seedHash": "0x…" } ]
  }
}
```

**`activationBlock` is an INPUT, and that is what makes this a component.** A network fixture pins
the height one chain actually chose; here it is supplied per vector, so the same file certifies any
chain adopting the rule. `null` means the chain never adopts it, and the length stays at the default
however high the block.

**The size vectors carry each quantity twice on purpose.** A client may serve sizes from a bundled
table below `bundledTableCoversEpochsBelow` and compute them above it. `cacheSizeBytes` is what it
must produce; `cacheSizeFromFormula` is the same quantity computed. They are equal at every vector
here **including across the 2047/2048 seam**, so a table transcription error shows up as a
disagreement between the two columns rather than only above the seam. Measured: a defect in the
formula alone leaves the table intact, and asserting one column would have caught it at three
vectors out of eight instead of all eight.

> #### The seed divisor is the trap, and it is not the epoch length
>
> `seedIterations = epochStartBlock / 30000` — the **default** length, whatever length is in
> force, and not the epoch number. So a post-activation epoch takes the seed its legacy twin would
> have had: `seed_epoch_195_length_60000` and `seed_epoch_390_length_30000` carry the **same** hash,
> because both start at block 11,700,001. A client dividing by the length in force computes 195
> iterations there and silently reuses a seed from six million blocks earlier — generating a real
> dataset for the wrong epoch rather than failing.
>
> Epoch 0 is the other edge: its start block is 1, below the divisor, so the seed is **32 zero
> bytes** and not the hash of them.

### `qbftProposerSelection` — which validator proposes at a height and round

```jsonc
{
  "qbftProposerSelection": {
    "_info": { },
    "policy": "roundRobin",
    "ordering": "byteOrder",
    "offsetFromParentProposer": "1",
    "validatorSets": {
      "<set name>": {
        "comment": "…",
        "validators": [ "0x…", "0x…" ],
        "stringOrderForContrast": [ "0x…", "0x…" ]
      }
    },
    "vectors": [
      { "name": "…", "comment": "…",
        "validatorSet": "<set name>",
        "blockNumber": "100", "round": "0",
        "parentProposer":   "0x…",
        "expectedProposer": "0x…",
        "tags": [ "round-zero", "discriminates-string-ordering" ] }
    ]
  }
}
```

The rule is `validators[(index(parentProposer) + offsetFromParentProposer + round) mod size]`, with
`validators` in ascending order of the raw 20 address bytes.

**Sort the set yourself rather than trusting the file's order.** It is already in byte order; sort
anyway, and **if your sort disagrees with the file's order, that is the finding** — ordering is
precisely what one of these vectors' defect classes is about.

**`stringOrderForContrast` is not an alternative a client may choose.** It appears only where a set
was constructed so that byte order and EIP-55 checksummed-string order differ, and it records the
wrong ordering so a reader can see what the vector discriminates against.

**`blockNumber` is carried for realism and is not an input.** The rule reads the validator set, the
parent block's proposer and the round, and nothing else.

**Addresses are EIP-55 checksummed and comparison is case-insensitive**, per "Duplicate keys are an
error" above: two spellings are one address.

**`tags` are documentation, not selectors.** A harness runs every vector; the tags say what each
one separates, so a failure report can say which property broke.

### `terminalTotalDifficultyMainnet` and `mergeNetsplitBlockSepolia` — where proof-of-work ends

Two network-scoped schemas for the same question, differing because the two networks answer it
differently: one by accumulated difficulty, one by a block number and a timestamp.

```jsonc
{
  "terminalTotalDifficultyMainnet": {
    "_info": { },
    "terminalTotalDifficulty": "58750000000000000000000",
    "mergeNetsplitBlock": null,
    "vectors": [
      { "name": "…", "parentTotalDifficulty": "…", "totalDifficulty": "…",
        "isTerminalPoWBlock": true, "note": "…" }
    ]
  }
}

{
  "mergeNetsplitBlockSepolia": {
    "_info": { },
    "terminalTotalDifficulty": "17000000000000000",
    "mergeNetsplitBlock": "1735371",
    "shanghaiTime": "1677557088",
    "vectors": [
      { "name": "…", "blockNumber": "…", "timestamp": "…",
        "isPostMerge": true, "note": "…" }
    ]
  }
}
```

**`mergeNetsplitBlock` is `null` where the network has none**, and JSON `null` here means *this
network does not use that mechanism* — it is not a missing field and not zero. A reader that
coerces it to zero concludes every block is at or past the netsplit.

**`isTerminalPoWBlock` is a predicate over a PAIR of difficulties**, the parent's accumulated total
and this block's: a block is terminal when its parent is below the threshold and it is at or above
it. Both are stated because the rule cannot be evaluated from one.

---

## What a MISSING expectation means, per shape

**This is the question a harness gets wrong most expensively**, because every wrong answer is
green.

There are three distinct situations and they must not be collapsed:

### 1. An absent `post` label — a SKIP, and it must be reported

A state fixture with no section for the upgrade being read yields **no runs** for that case. It is
not a pass.

**Count it and name it.** A case that quietly disappears between the corpus and the report is
coverage nobody can audit — a harness that skipped everything would otherwise be
indistinguishable from one that found nothing wrong.

Why a label is absent is recorded in `_info`, and the reasons are genuinely different:

| `_info` key | means |
|---|---|
| `unfilled-no-fork-name` | the generator has no name for that upgrade's rules, so nothing could be filled. **A tooling gap, not a claim that the case does not apply** |
| `unfilled-not-a-state-rule` | the upgrade changes no state-transition rule. A state fixture has nothing to assert; the coverage lives in another shape |
| `unfilled-typed-before-die-hard` | the transaction cannot be *signed* under those rules, so there is no encoding to record. A limit of the format, not a consensus claim |
| `unfillable` (difficulty) | the same idea, as a map of label to reason |

### 2. An EMPTY result inside a present expectation — an ASSERTION

A `post` entry that **is** present and whose expected state writes nothing is a claim: *this
opcode does not exist here*, *this refund was not granted*. The state root pins it.

**This is only legible because a fixture spans the schedule.** At a single upgrade, "no storage
written" and "we did not test this" are the same file. Five upgrades writing nothing and seven
writing a marker is an assertion about where an opcode arrives.

Two consequences for anyone authoring:

- **Never store an opcode's own result to prove it exists.** Several return zero, and zero into an
  already-zero slot cannot be distinguished from never storing. The first availability batch here
  did exactly that and reported opcodes as unavailable at every upgrade including ones where they
  plainly work — it looked correct and measured nothing.
- **Write an unconditional marker to a second slot.** One slot answers *did execution reach here*
  and the other answers *what was the outcome*. A liveness marker on the outer frame does not
  vouch for what the frame called: a failed inner call can leave it set.

### 3. An absent optional field — NOT an assertion

`logs`, `state`, `txbytes` absent means *this fixture makes no claim about that*, and a reader
checks what is present. Only `hash` is mandatory, and it covers everything the others do not.

### And one that is not about expectations at all

**An unreadable file is an outcome, not a crash.** Record it as one counted skip naming the parse
error. A reader that throws aborts every test in the suite, and a runner that catches a throw and
calls it a skip lets a machine that breaks on every case report as green. **A throw is a
divergence; only "there was nothing here to compare" is a skip.**

---

## Worked example: reading one state case end to end

Abridged from `state/accounts/typed_transaction_access_list.json`, with the real leading bytes so
the shape is checkable against the file:

```jsonc
"transaction": {
  "nonce": "0x00", "sender": "0xa94f…f0b", "to": "0x…1000", "gasPrice": "0x3b9aca00",
  "gasLimit": ["0x186a0"], "value": ["0x00"],
  "data":        ["0x",  "0x",  "0x"],
  "accessLists": [null,  [],    [ { "address": "0x…3000", "storageKeys": [] } ]]
},
"post": {
  "ETC_Phoenix": [
    { "indexes": {"data":0,"gas":0,"value":0}, "hash": "0x8dadc06c…", "txbytes": "0xf866…" },
    { "indexes": {"data":1,"gas":0,"value":0}, "hash": "0x6f1c4a7d…", "txbytes": "0x01f8…",
      "expectException": "TransactionException.TYPE_NOT_SUPPORTED|TransactionException.TYPE_1_TX_PRE_FORK" },
    { "indexes": {"data":2,"gas":0,"value":0}, "hash": "0x6f1c4a7d…", "txbytes": "0x01f8…",
      "expectException": "TransactionException.TYPE_NOT_SUPPORTED|TransactionException.TYPE_1_TX_PRE_FORK" }
  ],
  "ETC_Magneto": [
    { "indexes": {"data":0,"gas":0,"value":0}, "hash": "0xf83e0bb2…", "txbytes": "0xf866…" },
    { "indexes": {"data":1,"gas":0,"value":0}, "hash": "0xf83e0bb2…", "txbytes": "0x01f8…" },
    { "indexes": {"data":2,"gas":0,"value":0}, "hash": "0x53fee151…", "txbytes": "0x01f8…" }
  ],
  "ETC_Thanos": …absent…
}
```

**Reading this corpus at `ETC_Phoenix`** produces three runs against the same `pre` state,
independently:

1. `case[d0g0v0]` — `data[0]`, and `accessLists[0]` is `null`. `txbytes` begins `0xf8`, an RLP
   list, so it is legacy. Seed `pre`, admit, settle, compare the resulting root to `0x8dadc06c…`.
2. `case[d1g0v0]` — `txbytes` begins `0x01`, so it is an EIP-2930 access-list transaction. Phoenix
   predates that format, so it is refused **for its format, before anything is spent on its
   signature**. Split the stated names on `|`; if either maps to a refusal this build can produce,
   and that is the refusal produced, the case passes.
3. `case[d2g0v0]` — the same, with a non-empty access list. Also refused.

**Note that runs 2 and 3 carry the same root, and that it is not run 1's.** A refusal leaves the
pre-state exactly as it was, so both refusals reach the identical root — which is precisely why
the *reason* has to be compared separately. A harness checking only the root cannot tell these two
cases apart, and cannot tell either of them from a third refusal for some unrelated rule.

**Reading the same corpus at `ETC_Magneto`** produces three runs, all executing, and the roots do
**not** simply split three ways:

- `d0` and `d1` reach the **same** root. The typed envelope is admitted there, and an *empty*
  access list warms nothing, so the legacy transaction and the empty-list typed one leave
  identical state. That equality is the assertion — it separates *the envelope being accepted*
  from *the list having an effect*.
- `d2` reaches a different one, because listing the address the body touches makes the first touch
  warm.

**Reading it at `ETC_Thanos`** produces **zero** runs and **one reported skip naming the case**.
Not a pass. `_info.unfilled-no-fork-name` says why.

## Oracles: where an expected value may come from

`hash` is the root of the state trie *after* the transaction executes. It cannot be reasoned out;
it has to be computed by running the transaction. **That makes the oracle the whole question.**

**Never generate expected values from the client under certification.** That fixture asserts only
that the implementation agrees with itself — it will pass forever, including for every bug it
already has.

| for | oracle | standing |
|---|---|---|
| anything with an observable balance or state change | **the chain itself**, read from archive state | above every client, including the production one |
| everything through Spiral | **core-geth production** | the answer. Where another client disagrees it is wrong until shown otherwise |
| corroboration | **parity-ethereum**, **besu-etc** | genuinely independent lineages — see below |
| the shared proof-of-work era | **go-ethereum-pow** (v1.10.26) | the EVM both chains ran, and nothing Ethereum Classic did differently |
| Frontier and Homestead | **the pre-DAO pair** | the strongest oracle here: before the split there is no such thing as Ethereum's implementation as against this chain's |
| Olympia | **none yet** | the overlays lag the redrafted specifications |

### Independence is a property of the LINEAGE, and it is checkable

Not of the name. Establish it mechanically, with `git rev-list --max-parents=0 HEAD`:

| client | root commit | language |
|---|---|---|
| core-geth | `5db3335dc` | Go |
| **multi-geth** | **`5db3335dc`** | Go — **the same root. One lineage under two names, not a second opinion.** |
| parity-ethereum / openethereum | `f7b618cec` | Rust |
| besu-etc | `7dfc2e408` | Java |

> #### That command has two failure modes, and both return a confident wrong answer
>
> **1. A repository has SEVERAL roots, and the first line is not the one anyone quotes.** Measured
> 2026-08-26: `ethereum/go-ethereum` and `Consensys/quorum` each print **six**, identically, and the
> first is `6cf6981ed0` ("init", 2014-12-10) — not `5db3335dc`, which sits **last** and is the root
> every independence claim in this suite names. A check that takes `| head -1` therefore disagrees
> with a check that greps for the known root, and both look right. **Compare the SET, or ask the
> direct question:**
>
> ```bash
> git -C <clone> rev-list --max-parents=0 HEAD | sort          # all of them
> git -C <clone> cat-file -t 5db3335dce766bd679c54ea44f6df08a7ff74762   # "commit" = shares that root
> ```
>
> **2. A SHARED root is a question, not an answer — open it.** `ethereum/aleth` (C++, 27 roots)
> and `ethereum/go-ethereum` (6 roots) **share one**, `68ccbefc9`. Read naively, that says the C++
> client is a geth lineage. Opened, the root holds **three files** — `.gitignore`, `ethereum.js`,
> `index.html` — the early JavaScript library's history, merged into both repositories and into
> neither client's implementation. aleth is a genuinely independent lineage.
>
> ```bash
> git -C <clone> ls-tree -r --name-only <shared-root>    # what does the shared history CONTAIN?
> ```
>
> **The two failure modes point opposite ways**, which is why both are stated: the first hides a
> shared lineage, the second invents one. A root commit is evidence about what two repositories
> once merged, never by itself evidence about what they implement.

**This project's own overlays are NEVER an independent oracle**, whatever they are named after.
This project's own Besu, Nethermind and Scala overlays were written from the production client and
the specification by the same maintainer as the client under certification, so their agreement
establishes that one reader read the specification the same way twice. Check before
citing one: a file present in a fork and absent from its upstream is ours. `git ls-tree -r
upstream/master | grep -c <file>` against a control that must be non-zero settles it in one
command — and the control is not optional, since a zero from a broken search looks identical to a
zero from a real absence.

That rule cost something to learn: this suite cited its own Nethermind overlay as a "second
independent implementation" of ECIP-1010 for one commit, while stating the correct rule two
directories over.

**Search identifiers AND concepts, because naming differs across lineages.** A token search for
`ecip1010` finds nothing in besu-etc; the rule lives in `ClassicDifficultyCalculators` under
Besu's own vocabulary. A zero from one token set is a search result, not a finding — the same
trap as `grep -i mess` matching "message", arriving from the opposite direction.

**The point of a second implementation is disagreement.** Two clients agreeing raises confidence;
two disagreeing is a finding to resolve *before* a fixture is published, never a number to choose
between.

**Agreement across a representational difference outranks agreement between two copies.** Two
clients that compute Homestead's adjustment by different expressions — one in signed arithmetic as
a single formula, the other branching because its unsigned type cannot hold the negative term —
agreeing is much stronger evidence than two clients sharing a lineage agreeing.

The difficulty bomb is the worked example, and it is the strongest oracle claim in this suite:
three lineages state ECIP-1010 three ways — a reference point frozen then reduced by the window's
length; a start/end **pair** whose resume delay is *derived* as `(continue - pause) / period`; and
three named calculators carrying no reference point at all. Because the second derives what the
first states, **a shared transcription of one constant cannot explain their agreement** — which is
exactly what two implementations of the same shape can never rule out.

**Match the oracle to the era.** A client that never ran a rule is not a witness to it. The two
Parity-lineage clients were frozen before this chain's reorg-defense rule activated; multi-geth
was frozen three months after it and never implemented it.

### One rule has no independent oracle at all, and that is permanent

**MESS is implemented by exactly one production client** — not "one we have", one that exists.
This project's own Besu, Nethermind and Scala implementations agree with it and **none of them is
independent**: all were written from that client and the specification. They confirm the
specification was read the same way four times. They cannot catch the specification being wrong.

The compensation available is arithmetic, not a second opinion: the curve is a closed-form
polynomial, so it can be recomputed from the specification and checked against the client, which
is what the fixture does. That catches an implementation drifting from the spec. Nothing available
catches the spec itself being wrong.

> **A tooling note that has cost time more than once:** `grep -i mess` matches "message" and
> returns hundreds of files in every client, which reads as broad support. **Search identifiers,
> never nicknames** — the proposal number, the internal rule name, `artificialFinality`.

### Olympia: three overlays, one specification

Olympia has no production client to match, so its oracles are this project's own overlays, each
forked from an unrelated upstream. They are independent as *implementations*; the specification's
independence varies by rule, and treating it as uniform is its own error:

| the rule is… | specification risk |
|---|---|
| an EIP adopted unchanged | **low** — implemented and audited across the ecosystem for years |
| an adaptation with precedent elsewhere | **low-to-moderate** — the pattern has been reviewed where it already runs |
| genuinely novel, no precedent anywhere | **this is where the caveat bites** — only our own review stands behind it |

So the question for an Olympia fixture is which row its rule sits in, and the ECIP is where that
is recorded. Reserve the strong caveat for genuinely novel rules, and say which ECIP and which
review the fixture leans on.

**As of 2026-08-24 the overlays are not a valid oracle at all.** The specifications were redrafted
after the overlays were written, so an Olympia fixture generated from one today would encode a
superseded draft — and it would look exactly like a correct fixture, because a state root computed
from the wrong rules is still a valid state root. **When the overlays align, any Olympia fixture
authored earlier has to be re-derived, not re-checked.**

This is why the current scope stops at Spiral. Before Olympia there is a client to be right
against; at Olympia there is not yet.

---

## Running `state/` against the reference client, and what that does NOT check

**Every fixture in `state/` is directly runnable by core-geth's own state-test runner.** The shape
in this document *is* geth's GeneralStateTest shape, so no conversion is needed:

```bash
evm statetest networks/ethereumclassic/mainnet/state/gas/sload_cost.json
```

**Corpus-wide result, 2026-08-25: 531 of 546 subtests pass.** This was the suite's first
second-opinion check of any kind — until then the roots had only ever been *produced*, never
re-verified against anything.

### Two adjustments a geth-family runner needs

**One is ours to state, one is the runner's limit.**

**1. Three labels are not in any core-geth fork table.** `ETC_Frontier`, `ETC_Homestead` and
`ETC_GasReprice` are this project's names for upgrades core-geth addresses generically, because
before Die Hard this chain and Ethereum ran the same rules:

| our label | what a geth-family runner calls it |
|---|---|
| `ETC_Frontier` | `Frontier` |
| `ETC_Homestead` | `Homestead` |
| `ETC_GasReprice` | `EIP150` |

They are equivalent here and not merely approximated: the divergences that would make the naming
matter — a chain identifier reaching the EVM, replay protection — do not exist at those heights.
`CHAINID` does not arrive until Phoenix.

**Three further labels go the other way and only ONE build has them.** `ETC_DieHard`, `ETC_Gotham`
and `ETC_DefuseDifficultyBomb` are absent from production core-geth's fork table entirely; the
modernized build's list is production's plus exactly those three. **So the three upgrades whose
expectations no second build can even address are the three that most look settled**, and every
fixture's `oracle-version` says so explicitly rather than leaving it to be re-derived.

### `config.chainid` is NOT read by the reference runner — you must apply it yourself

**A harness that ignores this certifies the wrong network and reports success.**

geth's state-test struct carried exactly `_info`, `env`, `pre`, `transaction`, `out` and `post`
when this was checked, and **no `config` field of any kind**. Re-read the struct rather than the
count if the behavior below ever changes. A fixture's `config.chainid` is therefore
silently discarded, and every test runs at whatever chain id the named fork's own configuration
carries: **61 for every `ETC_*` label.**

Measured 2026-08-25: a `CHAINID` fixture asserting **61** was handed `"config": {"chainid":
"0x3f"}` (63) and **passed**. Nothing warned.

**On mainnet this is harmless and that is exactly why it went unnoticed** — the `ETC_*` forks are
chain 61, the fixtures assert 61, and the field merely restates what the runner already does. It
becomes load-bearing the moment a fixture describes a different network on the same rule sets,
which is what `networks/ethereumclassic/mordor/` does.

**Consequences for a consumer:**

- **Read `config.chainid` and apply it.** It is the fixture's statement of which chain these
  expectations belong to, and no geth-family state-test runner will apply it for you.
- **A fixture whose `config.chainid` disagrees with the chain id you are certifying is not a
  fixture you can run.** Fail loudly rather than proceeding — a passing run against the wrong
  chain id is worse than no run.
- **Mordor's `state/` cannot be checked with `evm statetest` at all**, and its fixtures say so in
  their own `_info`. Use `evm t8n --state.chainid 63`, which does honor a chain id; that is how
  they were produced and how they round-trip.

**The tell that this bit you**: a chain-identifier fixture that passes on two different networks.
It should be impossible, and under this runner it is routine.

### What the reference runner cannot check

**All 15 non-passing subtests are every `expectException` case in the suite, and none of them is a
wrong root.** The cause is structural and worth knowing before anyone reports them as failures:

> geth's state-test runner builds the message it executes from the `transaction` object's `sender`
> field. It never validates a signature and never decodes an envelope. `txbytes` is used only to
> confirm the bytes decode and a sender is recoverable — with `LatestSigner`, which accepts every
> form at every fork.

So the runner cannot express *"this transaction is refused at this height"*, which is exactly what
those 15 assert — a replay-protected transaction before Die Hard, and a typed envelope before
Magneto. A real client refuses both. The runner never asks.

**This is the boundary of the check, not a caveat on it.** 531 roots are confirmed by an
implementation that is not the one under certification. The other 15 need a consumer that decodes
`txbytes` as the authority, which is what this document already requires of one.

### `txbytes` was malformed corpus-wide until 2026-08-25

Recorded because the defect is invisible by inspection and disabled every check above.

Every `txbytes` in `state/` held `rlp([tx])` — the transaction wrapped in a one-element RLP list —
where the contract is the transaction's own binary encoding. **It still began `0xf8`, so it still
passed the leading-byte type test this document describes**, and it was a well-formed RLP object.
It simply was not a transaction, and `UnmarshalBinary` refuses it.

The consequence was total: no client could consume any `txbytes` in the corpus, and the
second-opinion run above was impossible for as long as it stood. 531 entries were unwrapped; 15
held `0xc0`, an empty wrapper meaning *no transaction bytes*, and were removed rather than
rewritten — the field is optional and absent is the honest encoding, where `0xc0` decodes to an
empty list and reads as data.

**No expectation moved.** The fix touched `txbytes` and nothing else; every `hash` and `logs` value
is byte-identical across it, which is the check to run after any mechanical pass like this one.

**Where the wrapper came from, since knowing it prevents the next instance.** `evm t8n
--input.txs`, with a `.rlp` extension, takes a JSON string holding the RLP of a **list** of
transactions — `rlp([tx])`, exactly the shape every `txbytes` here carried. It was the generator's
INPUT format, copied into an OUTPUT field where a bare transaction is required. **One shape,
correct in one direction and wrong in the other**, with nothing in either artifact recording which
direction it was for.

**That tool fails silently on the opposite mistake too.** Passing a bare transaction where `t8n`
wants the list runs nothing at all and reports success. It is detectable only because every fixture
body here writes a **liveness marker** to `0x1f` — a slot whose value is known to be non-zero on
any run that executed. Without it, "the transaction did nothing" and "the rule produced zero" read
identically.

**The unwrap is not a plain header strip, and getting it wrong looks like it worked.** A legacy
transaction sits inside the wrapper as a nested *list*, a typed one as a byte *string*. Removing
the outer header alone yields the right answer for every legacy case and silently corrupts every
typed one — the type byte goes with the header. Decode the wrapper as RLP and take its single
element.

## Every `state/` fixture has been shown to fail a client that gets its rule wrong

A fixture that passes a correct client has demonstrated nothing on its own — it may be pinning a
value no rule depends on. The check that matters is the opposite one: **build a client with the
rule removed and confirm the fixture fails.**

**As of 2026-08-25, 39 of the 41 fixtures in `state/` have been scored that way** against
deliberately-broken builds of the reference client, one defect per rule. Each fixture's `_info`
records the defect it was scored against and the result, under `wrongBuildScores` where the score
is specific enough to be worth naming.

**The two exceptions are `replay_protection` and `typed_transaction_access_list`**, and they are
unprovable by this method rather than unproven: they assert that a transaction is *refused*, and
the reference runner never validates a signature or decodes an envelope (see "What the reference
runner cannot check"). A consumer whose harness treats `txbytes` as the authority — which this
document requires — can score them; that runner cannot.

**Two results are worth carrying forward as reading advice, because both look like coverage gaps
and are not:**

- **A fixture can pin a value's magnitude without pinning the value.** `sstore_clear_refund` does
  not detect a one-gas change to the refund it is named for, because a transaction's refund is
  capped at a fraction of gas used and this body's refund exceeds the cap. It detects a third-sized
  change at four upgrades. That is a property of capped refunds, not a weakness, and shrinking the
  body until the cap stops binding would trade a mainnet-realistic case for an unobservable one.
- **A rule with more than one activation site needs all of them broken.** SELFBALANCE is installed
  by a function two call sites reach; disabling one leaves the opcode available and the resulting
  "no fixture caught this" is an artifact of the patch, not a fact about the suite. Before trusting
  a no-catch result, confirm the modified client actually behaves differently.

## What is not covered by any shape here

Stated so that a harness author does not go looking, and so a reader does not mistake the suite's
coverage for the rule's:

- **A block-level runner.** `blocks/ommer_payment_vectors.json` now states which address receives
  what for a given block shape, so the DATA a runner needs exists — but nothing executes it. The
  transition tool cannot: its reward option is one flat value applied uniformly and never consults
  an era schedule.
- **A chain-selection runner.** `chainselection/` asserts the decision and the derivation of its
  inputs. Presenting a client with two real competing chains needs a runner.
- ~~**Receipts.**~~ **Now covered** by `blocks/receipt_status_encoding.json`. A `post` entry
  carries a state root, a logs hash and the signed transaction and none of those reaches a
  receipt — so this was never a state-fixture gap but a missing block-level shape, and it is the
  last activation in this chain's mainnet configuration to be closed.
- **A sealed header from this chain.** `pow/` asserts the epoch schedule that decides *which*
  dataset a seal is verified against. It does not assert that a client rejects a seal validated
  against the wrong one — that needs a mined nonce, which is a mining problem rather than an
  authoring one.
- **Access-list pre-warming, in `fukuii-cli` specifically.** The fixture asserts it; that consumer
  does not carry an access list into execution. A consumer gap, recorded above.
- **A running proof-of-authority chain.** `consensus/` asserts the rules a PoA client applies
  — which difficulty a seal must carry, which signer set a header chain derives, which validator
  proposes next. Nothing here stands up two nodes and watches them agree, and one region is
  asserted by nothing on purpose: where implementations disagree and the specification declines to
  rule, that fixture's `_info` says so rather than picking a value.

**This list is maintained by hand and is therefore the least trustworthy section in the file.**
An entry can be closed without anyone striking it through — receipts were, once. Treat it as a
starting point for what to check, never as a statement of what remains.

---

## Naming

Data-tree conventions, not Scala's. Directory and file names follow what the corpora already use —
lowercase, suite- and upgrade-shaped. **There is no Scala in this repository to name.**
