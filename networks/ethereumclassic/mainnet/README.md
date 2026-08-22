# `networks/ethereumclassic/mainnet/` — coverage, in schedule order

Built **sequentially**, upgrade by upgrade, rather than by picking at gaps. A suite assembled from
whichever upgrades looked interesting is a suite with holes nobody has enumerated.

**Every fixture here is authored.** Nothing is borrowed, relabelled, or referenced as coverage.

## Why from scratch, when corpora already exist

Because a borrowed fixture is correct only by audit, and an authored one is correct by
construction.

The material available to lean on has known, measured problems. The archived Ethereum Classic
labels were produced by a text substitution that did not change what the fixtures test, so some
of them assert rules this chain does not run — including fixtures relabelled from Ethereum's
proof-of-stake transition, which assert that a zero-difficulty block is valid. Ethereum's own
corpora carry Ethereum's chain configuration and Ethereum's upgrade boundaries.

Auditing that material into trustworthiness is more work than authoring, and it leaves a suite
whose correctness rests on the audit having been exhaustive. Authoring leaves a suite where each
fixture states which upgrade it targets, under this chain's configuration, at the rule set that
upgrade actually carries.

**The existing corpora remain essential — as input, not as coverage.** They say what is worth
testing, they supply cases decades of adversarial work already found, and they serve as
comparison when an authored fixture and a borrowed one disagree. That is a different role from
being the suite.

## Each upgrade, in order

1. **What rules does it carry?** Read from the production client, never from a rendered spec.
2. **What has upstream already thought to test at the equivalent rules?** Mine the corpora for
   cases — the shapes, the edges, the attacks.
3. **Author the fixtures**, scoped to this upgrade, under this chain's configuration.
4. **Cross-check against the oracles**, and treat any disagreement as a finding.

## The generator gap, and it is the prerequisite for all of this

A fixture's post-state root has to be computed by executing the transaction, so every authored
fixture depends on a generator that can be told **which upgrade's rules to apply**.

core-geth's `evm t8n` is that generator, and it resolves an upgrade by name against a table in its
`tests` package. Measured 2026-08-21, that table carries **six** Ethereum Classic entries —
Atlantis, Agharta, Phoenix, Magneto, Mystique, Spiral. Every one of them is Atlantis or later.

**So the tooling gap lands exactly on the coverage gap.** Spiral can be generated today. Die Hard
cannot be generated at all, because there is no name to ask for — and Die Hard is precisely the
upgrade whose rule combination has never existed on Ethereum.

### What is missing, and which of it matters

| absent name | why it matters, or does not |
|---|---|
| **Die Hard** | **essential.** Two EIPs live, two not — plus this chain's own difficulty-pause rule, which activates at the same upgrade and has no Ethereum counterpart at all |
| Gotham | this chain's monetary policy — block rewards, so it reaches block-level fixtures rather than state ones |
| Defuse Difficulty Bomb | difficulty, not EVM state — needed for difficulty fixtures |
| Thanos | an epoch-length change to the mining algorithm; state tests do not exercise it |
| Gas Reprice | the same EIP as Ethereum's equivalent, which is already in the table |

### The six entries that exist are upstream's, and the missing ones are derivable

Verified 2026-08-21: the Ethereum Classic fork entries are **byte-identical between the
production client and this project's overlay**. None were added here. So generating a fixture at
one of them uses an oracle that is independent in both senses — upstream's configuration,
executed by upstream's implementation.

**Adding the missing entries does not give that up**, because they are derivation rather than
invention. The production client's own mainnet configuration already states which rules are live
at each height, and the existing entries show the encoding: a rule live at that upgrade is set to
zero, a rule not yet reached is left unset. An entry for the missing upgrade is that configuration
read at its own activation height and written in the table's own form.

What remains ours is the packaging — a name in a test table. The configuration is upstream's
reading of its own chain, and the implementation executing it is upstream's too.

**This is the modernization opportunity in the overlay.** Small, self-contained, and it unlocks
generation for the whole schedule rather than its last six upgrades. Until it lands, sequential
authoring stops where the table stops.

That work belongs in the client overlay, not here. What belongs here is the record that this
suite's sequence depends on it.

## Die Hard is the first upgrade that cannot be borrowed

Ethereum took EIP-155, EIP-160, EIP-161 and EIP-170 **together**, in one upgrade. Ethereum
Classic took the first two at Die Hard and the second two millions of blocks later, at Atlantis.

So between those two upgrades this chain runs a rule set that has never existed on Ethereum, and
**no upstream fixture can express it** — verified: the modern corpus offers 34 Spurious Dragon
fixtures and the legacy archive 579 at the equivalent label, and every one of them assumes all
four EIPs are live.

### What that makes the work

Not "write fixtures for Die Hard" — most of the Spurious Dragon corpus exercises only the first
two EIPs and is perfectly valid here. The work is:

- **Establish which of the borrowed fixtures are applicable** by running them under this upgrade's
  rules and classifying every divergence.
- **A divergence is a finding, not a failure.** It is either a fixture depending on an EIP this
  upgrade does not have — which needs an authored counterpart — or a client defect. Deciding
  which is the point.
- **Author the residue**: the behavior that is only correct when the first two EIPs are live and
  the second two are not.

That classification is the deliverable for this upgrade, and it comes before any fixture is
written.
