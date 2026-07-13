# fundprint-methodology

The authoritative, versioned methodology behind **Fundprint**, a free public
dataset tracing private-equity and other institutional-financial ownership of US
applied behavior analysis (ABA) and autism therapy clinics.

- **Public dashboard:** https://whofundsmytherapist.com
- **This repo defines:** what counts as a clinic, an owner, and a PE-backed
  relation; how claims are scored; the confidence floors and validation gates
  that decide what gets published.
- **Current methodology version:** `2026.07-no-threshold-v1`

If you want to understand how a number on the dashboard was produced, or whether
you can trust it, this is the repository that answers that.

## What the dataset currently says

As of `2026.07-no-threshold-v1`: **private equity owns 1,398 of the 1,559 autism
therapy clinics Fundprint traces, across 43 states.** The remainder are held by a
pension fund, a family office and two search funds, which are labelled as such and
never folded into the private-equity figure.

Measured against the federal provider registry, which lists 17,567 ABA providers
running 21,172 locations: Fundprint can name the owner of 915 of those locations
(4.3%), of which 754 (3.6%) are private-equity held. That national figure is small
because the profession is overwhelmingly independent, with 15,133 providers running
a single location. Concentration is local, and there it is much higher: private
equity holds 17.3% of Minnesota's ABA locations and 14.1% of Colorado's.

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

> Doke, A. (2026). *Fundprint Methodology*, version 2026.07-no-threshold-v1.
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
