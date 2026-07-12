# Changelog

All notable changes to the Fundprint methodology are recorded here. Each
methodology version is a frozen point that one or more dataset releases were
produced under. Older versions are retained so that older releases stay
interpretable.

The version format is `YYYY.MM-<label>`.

## Unreleased

Correction to the clinic definition and the counts that follow from it. No change
to confidence floors, so the methodology version is unchanged.

- **A clinic is now explicitly a physical service location, not a billing
  registration** (section 2). A chain may hold several NPIs at one address, and
  Action Behavior Centers holds six at a single Broomfield, Colorado suite. The
  clinic linker previously de-duplicated registry records by NPI, which cannot
  see this, so one center was published as many clinics. Sites are now identified
  by owner, street address, and ZIP.
- **Tracked clinics fall from 904 to 723** as a result. The error was uneven
  across owners, so the ranking changes: Action Behavior Centers drops from 184
  clinics to 71 and Charlesbank from second-largest owner to fifth. KKR remains
  first. No clinic was removed from the record; 181 duplicate rows were marked
  superseded and the underlying registrations remain published as claims.
- **Ghost clinics are now addressed and measured** (section 9). The provider
  registry reports existence-ever, not existence-now: nothing deactivates an NPI
  when a clinic closes, and every registry-sourced clinic in the dataset reports
  an active status, including records last certified in 2008. Fundprint now
  records how long each registration has gone untouched, and discloses that 89 of
  462 registry-sourced clinics (19%) rest on a record not updated in six or more
  years. Seven clinics whose registration was provably dead, because a chain under
  a different parent firm now registers at the same street address with a two-to-
  seven year staleness gap, were quarantined out of publication. Tracked clinics
  fall from 723 to 716.
- **One residual counting limit is disclosed** (section 9): eleven addresses
  where one owner's merged brands are both registered at a single site.

## 2026.06-floors-v0

The initial published methodology. Governs dataset release `2026.07-beta` and
later, until superseded.

Establishes:

- The scope and unit of analysis: the ownership chain `clinic -> owner entity ->
  parent financial firm`, for US ABA and autism therapy clinics.
- The owner-type taxonomy: `private_equity`, `pension_fund`, `family_office`,
  `other`. Institutional owners that are not private equity are labeled
  honestly rather than folded into "PE."
- The five-layer pipeline (Acquire, Store, Resolve, Validate, Publish) and the
  shape / trust / provenance contract between layers.
- The resolution model: a matcher proposes candidates, a verifier disposes, a
  deterministic walk assembles the chain. Chain confidence is the minimum along
  the chain.
- The confidence-method tags: `human_verified`, `exact_match`, `llm_inferred`,
  `fuzzy_high`.
- The confidence floors: clinic existence `0.85`, clinic-to-owner `0.80`,
  owner-to-parent `0.85`, acquisition date `0.75`.
- The three trust levels (`unverified`, `verified`, `human_anchored`) and the
  95% hand-validation gate, with stratified sampling and an append-only audit
  record.
- The quarantine rules for contradicted, flagged, or challenged claims.
- The provenance requirement: every published claim traces to a content-hashed
  snapshot of a public source; historical ownership is recorded as dated events,
  not as current ownership.
- The corrections and takedown path, and the four pinned versions
  (`schema_version`, `resolver_version`, `methodology_version`,
  `dataset_version`).

Notes on the first release produced under this version (`2026.07-beta`):

- The release is fully deterministic. Clinic-to-owner links are high-confidence
  name matches (`fuzzy_high`); owner-to-parent links are `exact_match` against
  named primary sources. No `llm_inferred` claims are present.
- Clinic existence is drawn from two sources: the NPPES provider registry
  (primary) and, as a supplement, tracked owners' own public location directories
  (`owner_location_directory`). The directory source exists because NPPES
  enumerates only the NPIs a chain registers and so undercounts chains that run
  many centers under a few NPIs. Directory centers are self-reported, are
  de-duplicated against NPPES by owner, city, and state, and are read from
  machine-readable schema.org data rather than scraped from rendered pages.
- The owner set grew over the release as more chains were sourced and verified.
  The current-release figures in Section 10 track the live dataset. This release
  is the first to exercise every owner-type label: `private_equity`,
  `pension_fund`, `family_office`, and `other`. The `other` label is used for a
  private investment firm that provides expansion capital but is not a
  traditional buyout fund, pension, or family office, keeping the owner-type
  claim honest rather than folding it into "PE."
