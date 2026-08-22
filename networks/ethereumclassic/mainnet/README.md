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
