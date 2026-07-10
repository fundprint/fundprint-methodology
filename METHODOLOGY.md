# Fundprint Methodology

**Methodology version:** `2026.06-floors-v0`
**Applies to dataset version:** `2026.07-beta` and later, until superseded.
**Maintained at:** https://github.com/fundprint/fundprint-methodology
**Public dashboard:** https://whofundsmytherapist.com

This document is the authoritative description of how Fundprint decides what to
publish. It is versioned: a dataset release names the methodology version it was
produced under, and that version points at exactly this text. If the method
changes, the version changes, and the previous text stays in the changelog so
older releases remain interpretable.

The guiding rule for everything below: **we publish a claim only when we can
defend it in front of a journalist, an academic, or a Senate staffer.** Every
design choice here follows from that rule.

---

## 1. Purpose and scope

### What Fundprint is

Fundprint is a free, public dataset tracing private-equity and other
institutional-financial ownership of applied behavior analysis (ABA) and autism
therapy clinics in the United States. It exists so that a parent, reporter, or
researcher can look up a clinic and see who ultimately owns it, with a public
source for every link in the chain.

The ownership of autism-therapy providers has consolidated rapidly. A January
2026 academic study documented private-equity firms acquiring more than 500
autism centers over the prior decade, using PitchBook and scattered public
records. Fundprint builds the public-data equivalent of that linkage: an open,
downloadable, source-cited artifact rather than a proprietary one.

### What Fundprint is not

- It is not a directory of every clinic. Coverage is bounded by what public
  records expose (see Section 9).
- It is not a quality rating. Fundprint says nothing about the care a clinic
  provides. Ownership is not a judgment of quality.
- It is not investment advice or a prediction. Fundprint reports ownership that
  is documented now. It does not guess at "likely PE-acquired" clinics.
- It is not real-time. The dataset is a versioned snapshot. When a fresher
  number is needed, a new snapshot is cut.

### Units of analysis

Three entities and the links between them:

- **Clinic:** a physical or organizational service location, identified where
  possible by a National Provider Identifier (NPI).
- **Owner entity:** the operating brand or legal entity that runs one or more
  clinics (for example, a named ABA company).
- **Parent financial firm:** the private-equity firm, pension fund, or family
  office that ultimately owns the owner entity.

A published record is a chain: `clinic -> owner entity -> parent financial
firm`, with a confidence score and a public source for each hop.

---

## 2. Definitions

**ABA / autism therapy clinic.** A provider whose service is applied behavior
analysis or related autism intervention. Identified through public provider
registries.

**Private equity (PE).** A firm that acquires companies primarily using pooled
investment capital with the intent of a later sale or recapitalization. This is
Fundprint's primary frame.

**Owner type (`firm_type`).** Not every institutional owner is private equity.
Fundprint labels each parent firm honestly rather than calling every financial
owner "PE." The taxonomy:

| Label             | Meaning                                                        |
|-------------------|----------------------------------------------------------------|
| `private_equity`  | A traditional private-equity fund or PE-backed platform.       |
| `pension_fund`    | A public or private pension investor (for example, a teachers' retirement plan). |
| `family_office`   | A single-family or closely-held private holding company.       |
| `other`           | Another institutional financial owner that fits none of the above. |

**PE-backed relation.** An owner entity is treated as backed by a parent firm
when a public source states that the parent acquired, controls, or holds a
controlling interest in the owner entity, and that relationship is the current
one. Historical ownership that has since ended is recorded separately as an
event, not as current ownership (see Section 8).

**Claim.** A single scored assertion with a method tag and at least one source.
Nothing is asserted as truth without being a claim first.

---

## 3. The five-layer pipeline

Fundprint's data pipeline is five layers, each with one job, one input shape,
one output shape, and one quality guarantee.

```
1. Acquire    external sources        -> raw, source-stamped snapshots
2. Store      snapshots               -> typed staging tables in Postgres
3. Resolve    staging rows            -> resolved entities and ownership chains
4. Validate   resolved entities       -> confidence-scored, auditable claims
5. Publish    validated rows          -> public dataset, dashboard, this repo
```

Two separations are deliberate and load-bearing:

- **Acquire versus Store.** Acquisition absorbs the instability of the outside
  world (sites change, APIs rate-limit, PDFs come in odd encodings). Storage
  promises typed, queryable data. A website redesign must not break a query.
- **Resolve versus Validate.** Resolution produces a best guess. Validation
  decides whether the best guess is good enough to publish. Keeping them
  separate lets resolution scale without silently loosening what gets published.

Each boundary between layers carries three properties: **shape** (what crosses),
**trust** (what is promised), and **provenance** (what is logged). A change that
cannot fill in all three is not ready to ship.

---

## 4. Resolution

Resolution turns staging rows into scored candidate links. It never asserts
truth; it proposes claims that Validation then accepts or holds.

The design is a three-stage pipeline: a broad, cheap matcher proposes candidate
links; a narrow, expensive verifier confirms them; a deterministic graph walk
assembles the chain to the ultimate parent firm.

- **Candidate (propose).** Name and locality matching produces the top few
  possible links. High recall, low precision. Its failures look like missed
  matches.
- **Verify (dispose).** A language model reads the candidate plus its supporting
  documents and returns a structured, scored claim with the exact snippets it
  relied on. High precision when calibrated. Its failures look like false
  matches.
- **Chain (assemble).** A deterministic walk links `clinic -> owner -> parent`.
  Its failures look like broken provenance (a hop that lost its source).

The split is rigid: **the matcher proposes, the verifier disposes.** Reversing
that would let the verifier invent links the matcher would never have surfaced,
which is the one failure mode the project cannot afford. A model claim with no
supporting snippet is rejected at the boundary: no snippet, no claim.

### Confidence propagation

A chain's confidence is the **minimum** confidence along the chain, never the
product or the average. If `clinic -> owner` scores 0.95 and `owner -> parent`
scores 0.60, the chain is 0.60. A weak link defines the chain. This is
conservative by design: it biases toward "we are unsure," and unsureness is
exactly what the validation floor gates.

### Idempotence

Resolution is idempotent for a fixed `(resolver_version, methodology_version,
input set)`: running it twice over the same inputs yields the same entities and
the same scored claims. Bumping `resolver_version` produces new rows that
supersede the old; the old rows are retained for audit. If yesterday's dataset
cannot be reproduced from yesterday's inputs, yesterday's claims cannot be
defended.

---

## 5. Confidence methods

Every claim carries a `confidence_method` tag recording how it was established.
The methods, from strongest to weakest evidence:

| Method            | How the link was established                                   |
|-------------------|----------------------------------------------------------------|
| `human_verified`  | A person read the sources and confirmed the link.              |
| `exact_match`     | An exact, normalized identity between a stated name and an entity (used for ownership links drawn from a named primary source). |
| `llm_inferred`    | A language model confirmed the link from supplied documents, with cited snippets. |
| `fuzzy_high`      | A normalized name match above a high threshold (for example, a brand-name prefix match tying a registry record to a known owner brand). |

The method tag travels with the claim into the published dataset, so any
consumer can filter by how a link was established. See Section 10 for the
method breakdown of the current release.

---

## 6. Confidence floors

The publishable view is a filter over validated claims subject to a per-type
confidence floor. A claim below its floor stays internal. It is never deleted;
it waits for better evidence or improved resolution.

The floors defined by methodology version `2026.06-floors-v0`:

| Claim type          | Floor  | What it governs                                    |
|---------------------|--------|----------------------------------------------------|
| Clinic existence    | `0.85` | The clinic is real and at the stated location.     |
| Clinic to owner     | `0.80` | The clinic is operated by the named owner entity.  |
| Owner to parent firm| `0.85` | The owner entity is owned by the named parent firm.|
| Acquisition date    | `0.75` | A dated ownership event, where "circa year" is acceptable. |

These are conservative thresholds. They are set in the methodology and mirrored
in the pipeline's implementation; changing a floor requires a methodology
version bump and a re-run of validation before the next release.

---

## 7. Validation and the 95% gate

Resolution proposes. Validation decides which claims earn the right to be
published. This is the layer most likely to be skipped under deadline pressure,
and skipping it is what would make the project hollow, so it is treated as
non-negotiable.

### Trust levels

Every claim sits at one of three levels:

| Level            | Meaning                                             | Where it can go              |
|------------------|-----------------------------------------------------|------------------------------|
| `unverified`     | A scored claim that has not been validated.         | Internal only.               |
| `verified`       | Passed the validation run's confidence floor.       | Dashboard and public dataset.|
| `human_anchored` | Additionally spot-checked by a human reviewer.      | Dashboard, dataset, citable. |

Trust moves upward freely and downward only with a recorded validation event.
Trust is never silently degraded.

### The 95% hand-validation gate

Before any scaling step (a jump in clinic count, a new state, or a new
`resolver_version`), a random sample of newly-resolved claims is hand-validated:

- The sample is drawn at random from new claims, stratified by
  `confidence_method` so that both name-match and model-inferred claims are
  represented.
- A reviewer reads each row's source documents and labels it agree, disagree,
  or unclear.
- The gate passes only if `agree / (agree + disagree)` is at least **0.95**.
- "Unclear" rows are counted separately as a clarity signal on the methodology,
  not as failures.
- A failed gate blocks publication of that batch. The fix goes to the pipeline,
  not to the sample.
- The sample, the labels, and the sources reviewed are committed as a dated,
  append-only record. They are never rewritten.

### Quarantine

Quarantine is the explicit "held out of publication" state. A claim is
quarantined when sources contradict each other and the resolver cannot pick a
winner, when a model extraction raised a flag, when a human reviewer labeled a
row unclear, when an external party has challenged the claim and the challenge
is unresolved, or when a correctly-identified claim is out of scope for the
dataset (for example, an owner we track whose named entity does not operate ABA
or autism clinics). Quarantined claims never appear in public exports, and each
quarantine is an append-only decision that records its reason. The quarantine
rate is itself a monitored metric: a rising rate signals that a source changed
shape or that resolution needs work.

---

## 8. Provenance and audit

**Every published claim traces to a captured public source.** For clinic
existence, that is a public provider-registry record, drawn from NPPES (the CMS
National Plan and Provider Enumeration System), which lists provider
organizations with a National Provider Identifier, name, and location. Clinics
are gathered two ways: a broad pull by provider taxonomy, and a targeted pull by
organization name for each tracked brand. The brand-targeted pull matters
because a single taxonomy query is capped by the registry, so it cannot surface
every location of a large chain; querying each tracked brand by name gives it
its own record budget and captures locations the broad pull misses. For an
ownership link, the source is a captured primary source: an acquisition
announcement or a reputable trade-press report that explicitly states the
ownership.

When a source is ingested, Fundprint fetches and stores the exact document as a
content-hashed snapshot, then links it to the claim. The claim records which
snapshot supports it. A row missing its source records does not publish,
regardless of confidence.

Historical ownership that has ended is not shown as current ownership. It is
recorded as a dated event (for example, an acquisition, a divestiture, or a
bankruptcy), each with its own source, and displayed on a timeline rather than
as a live ownership link. A firm that no longer owns any clinic in the dataset
is shown, if at all, only for that history, with a current clinic count of zero.

Every validation run produces an append-only audit record: a run identifier and
timestamp, the exact input claim identifiers and the `resolver_version` they
came from, the `methodology_version` used for the floors, the pass or fail
decision per claim with the deciding rule, and the hand-validation sample where
one was run. A future reader asking "how did you know this at the time?" must be
answerable from this record without reconstruction.

---

## 9. Limitations and known gaps

Fundprint states its limits plainly, because a number is only useful if its
boundaries are known.

- **Coverage is bounded by public records.** A clinic that does not appear in
  the public registries Fundprint ingests will not appear in the dataset, even
  if it exists and is PE-owned. "Clinics tracked" is coverage in the public
  record, not a census of every clinic an owner operates.
- **The current release is deterministic.** The `2026.07-beta` release links
  clinics to owners by high-confidence name matching and links owners to parent
  firms by exact match against named primary sources. The language-model verify
  stage described in Section 4 is part of the designed pipeline but was not
  exercised for this release. There are no `llm_inferred` claims in the current
  dataset. This keeps the release fully deterministic and reproducible.
- **Name-prefix matching can over-capture.** A brand-name prefix match can link
  a registry record that shares a leading name with an owner brand but is not
  actually part of it. It can also capture a correctly-identified owner that is
  out of scope: an early release attached six Geode Health records (a KKR-backed
  outpatient mental-health provider, not an ABA or autism provider) to KKR
  through a name match. Those records were identified and held out of
  publication as out of scope, and the linker is guarded against re-capturing
  them. Conversely, a spelling variant or an abbreviated legal name can cause a
  real clinic to be missed. Both directions of error are possible, and the
  confidence floor, the out-of-scope holds, and the hand-validation gate are the
  controls against them.
- **Ownership is a moving target.** Deals close and unwind continually. The
  dataset reflects what public sources documented as of its snapshot date. A
  divestiture the week after a snapshot will not be reflected until the next
  snapshot.
- **Non-controlling and indirect stakes.** Fundprint records ownership that a
  public source states as controlling or operating. Minority stakes, debt
  positions, and complex holding structures may be understated where public
  sources do not make them explicit.

---

## 10. The current release at a glance

Figures below describe dataset version `2026.07-beta`. The dataset and the
dashboard are the live source of truth; these numbers are a snapshot for
context.

- **Clinics tracked:** 570
- **Current owners with tracked clinics:** 7, plus one former owner shown for
  history only
- **States covered:** 29
- **Method breakdown:** every published clinic-to-owner link in this release is
  a high-confidence name match (`fuzzy_high`), and every owner-to-parent link is
  an `exact_match` against a named primary source. No `llm_inferred` claims are
  present in this release.

Current owners, by owner type and tracked clinic count:

| Parent firm                    | Owner type      | Clinics tracked |
|--------------------------------|-----------------|-----------------|
| Charlesbank                    | private equity  | 184             |
| Arsenal Capital Partners       | private equity  | 139             |
| KKR                            | private equity  | 138             |
| Ontario Teachers' Pension Plan | pension fund    | 46              |
| Moran Capital Partners         | family office   | 39              |
| Thomas H. Lee Partners         | private equity  | 20              |
| Tenex Capital Management       | private equity  | 4               |

Blackstone appears as a former owner (Center for Autism and Related Disorders,
lost in the 2023 bankruptcy) with a current tracked-clinic count of zero, shown
for its documented history only.

---

## 11. Corrections and takedowns

Fundprint publishes claims about real organizations, so it commits to a
correction path:

- **Corrections** are made in the next dataset release with a changelog entry.
  A published release is immutable; it is never edited in place. A consumer who
  pinned a version must get the same bytes later.
- **Challenges** from an external party (an owner, a journalist, an advisor)
  move the disputed claim to quarantine, where it is held out of public exports
  until the challenge is resolved with sources.
- **Contact for corrections:** atharva.doke737@gmail.com.

---

## 12. Versioning

Fundprint pins four versions together at every release:

- `schema_version`: the shape of the stored data.
- `resolver_version`: the resolution logic that produced the claims.
- `methodology_version`: the definitions and floors in this document.
- `dataset_version`: the released snapshot.

A methodology change (a new floor, a changed definition of a countable
relation) requires a `methodology_version` bump and a re-run of validation. The
release order is methodology first (freeze the definitions), then data (validate
against them and publish), then dashboard (pin the release and deploy). Out of
that order, a reader could see numbers that do not match the document they are
reading, which is the kind of inconsistency that this whole process exists to
prevent.

---

## Glossary

- **NPI:** National Provider Identifier, a public identifier for a healthcare
  provider or organization.
- **Claim:** a single scored, sourced assertion (the atomic unit of the
  dataset).
- **Confidence floor:** the minimum score a claim of a given type must meet to
  be published.
- **Quarantine:** the held state for a claim that is contradicted, flagged, or
  challenged.
- **Owner entity:** the operating brand or legal entity that runs clinics.
- **Parent firm:** the ultimate institutional financial owner of an owner
  entity.
- **Supersede:** to replace an older row with a newer one while retaining the
  old for audit.
</content>
