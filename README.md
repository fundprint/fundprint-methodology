# fundprint-methodology

The authoritative, versioned methodology behind **Fundprint**, a free public
dataset tracing private-equity and other institutional-financial ownership of US
applied behavior analysis (ABA) and autism therapy clinics.

- **Public dashboard:** https://whofundsmytherapist.com
- **This repo defines:** what counts as a clinic, an owner, and a PE-backed
  relation; how claims are scored; the confidence floors and validation gates
  that decide what gets published.
- **Current methodology version:** `2026.07-statefile-v1`

If you want to understand how a number on the dashboard was produced, or whether
you can trust it, this is the repository that answers that.

## What the dataset currently says

As of `2026.07-statefile-v1`: **private equity owns 1,621 of the 1,755 autism
therapy clinics Fundprint traces, across 42 states.** The remainder are held by a
pension fund, a family office and two search funds, which are labelled as such and
never folded into the private-equity figure. Each clinic is graded on four separate
questions, not one: whether it is open, at its address, a centre or in-home care,
and who owns it. 1,573 are owner-verified centres, listed by the owner's own
directory today; the rest rest on the provider registry alone.

Measured against the federal provider registry, which lists 17,569 ABA providers
running 21,088 locations: Fundprint can name the owner of 682 of those locations
(3.2%), of which 580 (2.8%) are private-equity held. That national figure is small
because the profession is overwhelmingly independent, with 15,142 providers running
a single location. Concentration is local, and there it is much higher: private
equity holds 15.8% of Minnesota's ABA locations and 13.2% of New Mexico's.

Those shares are a **floor**. Fundprint removes closed centres from its own count,
because it reads its owners' directories and can see that they closed; it cannot
read the directories of the other 17,000 operators, so their closed centres stay in
the denominator. The bias runs against us, and it is disclosed rather than adjusted
away.

**And the count itself is a floor, for a second reason: we do not cover every
company.** Fundprint publishes **21 of 32** known private-equity-backed ABA
platforms. The denominator there is deliberately not ours; it is the appendix of
the Private Equity Stakeholder Project's April 2026 report, content-hashed like any
other source, because a list we drew ourselves would be one we could close by
declining to look further. Of the 11 platforms not covered, 8 are simply not
started and 3 are blocked on a documented obstacle, and between them they hold
**504 facilities** by PESP's own count. Every one is named, with the reason, in
[section 8d](./METHODOLOGY.md). The list runs both ways: PESP omits four platforms
published here, including Caravel Autism Health and its 79 clinics.

**Against the published estimate, the difference is 2.8x, and it is decomposed
rather than asserted.** The peer-reviewed count for the same country is 574
PE-owned sites as of 31 December 2024 (*JAMA Pediatrics*, January 2026, from
PitchBook acquisition records). Fundprint publishes 1,621. Both find private equity
in exactly 42 states, so the disagreement is depth, not footprint. Restricted to
what the federal registry can see, the two counts are **580 against 574**: two
censuses sharing no data source land six sites apart. The whole of the remainder is
centers that appear only in an operator's own published directory. And the
registry's blind spot is not a constant, running from 1.0x to unbounded operator by
operator, which is why no multiplier can correct a deal-based or registry-based
count.

A **second** published estimate settles what that means. The Private Equity
Stakeholder Project counts platforms rather than deals, and over the 16 platforms
both cover it reports 1,521 facilities against Fundprint's 1,640, a ratio of 1.08x,
with Fundprint *lower* on four of them and exact on two. That is a precision check,
not a national total: summed across all 32 in-scope platforms PESP implies 2,047,
against the peer-reviewed 574. The two outside sources differ from each other by more
than either differs from Fundprint, which sits between them. So the estimates sort by
what they count, not by how carefully: a census built on deals or registrations sees
a company frozen at the moment it was bought or registered, and one built on
platforms or directories sees it today. Both comparisons, with their limitations and
their partial independence named, are [section 8e](./METHODOLOGY.md). This is a
contribution rather than a rebuttal: the peer-reviewed letter states both of these
gaps itself.

**The state files pair the ownership footprint with what government auditors
found, and refuse the causal claim.** Federal and state audits of Medicaid ABA
spending in Colorado, Indiana, Maine and Wisconsin found $197.9 million in improper
payments, with a further $206.8 million flagged as potentially improper. **None of
those audits attributes a dollar to private equity**; they audit state programs and
name no company. Maine is published as the control case: the third-largest finding
of the four, and zero private-equity-owned clinics traced there. What the pairing
does support is narrower and needs no causal claim, and Wisconsin shows it best: of
the 48 private-equity-owned centres traced in that state, the federal provider
registry can see 2, so the people auditing the spending cannot see who owns the
providers. See [section 8f](./METHODOLOGY.md).

**There is no "chain" share.** An earlier release headlined private equity's share
of clinics run by operators with five or more locations. It was withdrawn, and the
reasoning is in the [changelog](./CHANGELOG.md): the cutoff was arbitrary, and the
group it measured against is one that private equity itself assembles, so its own
buying inflated the numerator and the denominator together. The figures that
survive require no cutoff at all.

## Read the methodology

The full methodology is a single versioned document:

> **[METHODOLOGY.md](./METHODOLOGY.md)**

It covers purpose and scope, definitions, the five-layer pipeline, resolution,
confidence methods and floors, the 95% hand-validation gate, provenance and
audit, limitations, the current release at a glance, corrections, and
versioning.

## Why a separate methodology repo

Fundprint is built as three repositories with one contract between them:

| Repository              | Role                                                      |
|-------------------------|-----------------------------------------------------------|
| `fundprint-methodology` | Defines the method. Frozen per release. (This repo.)      |
| `fundprint-data`        | Runs the pipeline against a methodology version and publishes the dataset. |
| `fundprint-dashboard`   | Renders the published snapshot at whofundsmytherapist.com. |

A dataset release names the methodology version it was produced under. That
version points at exactly the text in this repo at that tag, so an older release
stays interpretable even after the method evolves. Definitions are frozen here
first, before any data is validated against them.

## How to cite

If you reference Fundprint's method or dataset, please cite the methodology
version you relied on. See [CITATION.cff](./CITATION.cff), or use:

> Doke, A. (2026). *Fundprint Methodology*, version 2026.07-statefile-v1.
> https://github.com/fundprint/fundprint-methodology

For a specific dataset release, also name its `dataset_version` (for example,
`2026.07-beta`) and the dashboard URL.

## Corrections

Fundprint publishes claims about real organizations and commits to a correction
path. Corrections are made in the next dataset release with a changelog entry;
challenges move the disputed claim to quarantine until resolved with sources.
See the corrections section of the methodology for details.

**Contact:** atharva.doke737@gmail.com

## License

The methodology text in this repository is licensed under
[CC BY 4.0](./LICENSE). You may share and adapt it with attribution.

## Changelog

Version history is in [CHANGELOG.md](./CHANGELOG.md).
