# `networks/ethereumclassic/mainnet/` — coverage, in schedule order

Built **sequentially**, upgrade by upgrade, rather than by picking at gaps. A suite assembled from
whichever upgrades looked interesting is a suite with holes nobody has enumerated.

Each upgrade answers three questions, in order:

1. **What rules does it carry?** Read from the production client, never from a rendered spec.
2. **What existing corpus covers those rules**, and under whose chain configuration?
3. **What is left over?** That, and only that, is authored here.

## Where the sequence currently stands

The client certifies Ethereum at Frontier, Homestead and Tangerine Whistle, and Ethereum Classic
at **Gas Reprice** — by running Ethereum's Tangerine Whistle fixtures under Ethereum Classic's
rules, which is sound because that upgrade is EIP-150 on both chains.

**Gas Reprice is the last upgrade an Ethereum corpus can cover unaided.** Everything after it
needs this directory.

## The sequence

| upgrade | relation to Ethereum | coverage |
|---|---|---|
| Frontier … Gas Reprice | identical rules | **reference** — Ethereum corpora, run under this chain's configuration |
| **Die Hard** | **a combination no Ethereum fork ever had** | **author** — see below |
| Gotham | Ethereum Classic's own monetary policy | **author** — no Ethereum counterpart exists |
| Defuse Difficulty Bomb | Ethereum Classic removed the bomb | **author** — Ethereum only ever delayed it |
| Atlantis | Byzantium minus one EIP | **author the difference**; reference the rest |
| Agharta | Constantinople + Petersburg | reference — confirm the archive label first |
| Phoenix | Istanbul, equivalent | **reference** — archived label is sound |
| Magneto | Berlin, equivalent | **reference** — archived label is sound |
| Mystique | London minus three EIPs | **author the difference**; reference the rest |
| Spiral | Shanghai minus two EIPs | **author** — no archived coverage exists at all |

Read the rule content for each from the production client. **Heights are deliberately absent
here**; a copied activation point goes stale silently while continuing to read as authoritative.

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
