# Changelog

All notable changes to the Fundprint methodology are recorded here. Each
methodology version is a frozen point that one or more dataset releases were
produced under. Older versions are retained so that older releases stay
interpretable.

The version format is `YYYY.MM-<label>`.

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
