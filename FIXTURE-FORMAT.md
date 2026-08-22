# Fixture format — what the harness reads

Everything in `proposals/` and `networks/` is data read by the client's certification harness.
This is the shape it parses. **Read the harness itself before authoring** — it is under active
development, and this file describes it as of 2026-08-21.

## State fixtures

One JSON file holds one or more named tests:

```jsonc
{
  "<test name>": {
    "config":      { "chainid": "0x3d" },     // 61 for Ethereum Classic mainnet
    "env": {
      "currentCoinbase":   "0x…",
      "currentNumber":     "0x…",
      "currentTimestamp":  "0x…",
      "currentDifficulty": "0x…",             // NON-ZERO — see below
      "currentGasLimit":   "0x…"
    },
    "pre":         { "0x<address>": { "balance": "0x…", "nonce": "0x…",
                                      "code": "0x…", "storage": {} } },
    "transaction": { "nonce": "0x…", "sender": "0x…", "to": "0x…",
                     "gasPrice": "0x…",        // or maxFeePerGas for a dynamic-fee tx
                     "gasLimit": ["0x…"], "value": ["0x…"], "data": ["0x…"] },
    "post": {
      "<UpgradeLabel>": [
        { "indexes": { "data": 0, "gas": 0, "value": 0 },
          "hash":    "0x…",                    // post-state trie root
          "logs":    "0x…",                    // logs bloom/root
          "txbytes": "0x…",
          "state":   { },
          "expectException": "TransactionException.…"   // only for a refusal case
        }
      ]
    }
  }
}
```

`gasLimit`, `value` and `data` are **arrays**, and `indexes` selects one of each — that is how a
single fixture expands into a matrix of cases.

`post` is keyed by upgrade label, and the harness is told which label to read a corpus under. The
same file can therefore carry expectations for several upgrades.

## Refusals are named, not implied

A case that must be rejected carries `expectException` with a `TransactionException.…` name. The
harness maps those names onto the refusals this build can produce.

**A name it does not recognize is not ignored** — it survives verbatim and the case diverges,
reporting which rule was unmatched. That is deliberate: it prevents a case passing because a
refusal for some *other* reason happened to leave the state root where the fixture expected it.

Use the vocabulary the harness already maps. Inventing a name produces a test that cannot pass.

## Two things that are Ethereum Classic's and not Ethereum's

**`chainid` is 61 on mainnet**, not 1. A fixture omitting it inherits a default that is wrong here,
and the failure is a signature mismatch that looks like something else.

**`currentDifficulty` is non-zero and must stay that way.** Ethereum Classic is proof-of-work.
Post-Merge Ethereum fixtures carry `currentDifficulty: 0x00`, and the archived corpus contains
some that were relabelled onto an Ethereum Classic upgrade without that being corrected — see
`networks/ethereumclassic/README.md`. **Never copy a zero-difficulty fixture forward.**

## You cannot hand-write a fixture, and you must not generate one from Fukuii

`hash` is the root of the state trie *after* the transaction executes. It cannot be reasoned out;
it has to be computed by running the transaction.

**That makes the oracle the whole question.** Generating expected values by running the
implementation under test produces a fixture that asserts only that the implementation agrees
with itself — it will pass forever, including for every bug it already has.

So an authored fixture's post-state must come from **an independent implementation**: a production
Ethereum Classic client, or a filling tool that is not the client being certified. Where a
divergence appears between two independent implementations, that is a finding to resolve before
publishing a fixture, not a number to pick from.

**A fixture whose oracle is unrecorded is not usable.** Say which implementation produced the
expected values, and at which version.

## The oracles, and what each one settles

Measured 2026-08-21. **Re-verify before relying on any of this** — these are readings of active
codebases.

| client | standing | settles |
|---|---|---|
| **core-geth** | **production. This is what Ethereum Classic runs.** | the answer. Where Fukuii disagrees, Fukuii is wrong until shown otherwise |
| **besu-etc** | independent, second implementation | corroboration — a second opinion on the same fixture |
| **go-ethereum-pow** | Ethereum's last proof-of-work release, v1.10.26, frozen 2022-11-03 | the proof-of-work-era EVM the two chains share |

**The point of a second implementation is disagreement.** Two clients agreeing raises confidence;
two disagreeing is a finding to resolve *before* a fixture is published, never a number to choose
between.

### besu-etc is weaker than core-geth, but not in the ways it is usually assumed

Checked rather than assumed, because the received wisdom about it is mostly wrong:

- **It has proof-of-work mining** — `PoWMiningCoordinator`, `PoWMinerExecutor`, in main source.
- **Its Ethereum Classic upgrade schedule is complete and correct**, through Spiral, with
  activation heights matching core-geth exactly and the DAO-rejection block set.

Where it is genuinely weak:

- **It does not implement MESS at all.** No artificial-finality code exists, so it cannot be an
  oracle for that rule — there is nothing to ask.
- **Its Ethereum Classic consensus classes are almost entirely untested.** Four of its five carry
  no test coverage, so agreement from it is weaker evidence than agreement from a client whose
  behavior is exercised.
- **It has never run Ethereum Classic-labelled fixtures**, consuming only Ethereum's corpus.

So treat it as a real second opinion with a known blind spot, not as a co-equal authority and not
as a client to dismiss.

### go-ethereum-pow is for the shared era only

It is **Ethereum's** client, carrying no Ethereum Classic configuration, and its history includes
post-merge code. It is useful precisely where the two chains share rules — the proof-of-work-era
EVM — and says nothing about anything Ethereum Classic did differently.

### Recording the oracle is part of the fixture

Which implementation produced the expected values, and at which version. A fixture whose oracle is
unknown cannot be re-derived, and cannot be re-checked when a client changes.

## Naming

Data-tree conventions, not Scala's. Directory and file names follow what the corpora already
use — lowercase, suite- and upgrade-shaped. There is no Scala in this repository to name.
