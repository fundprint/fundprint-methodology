# Changelog

All notable changes to the Fundprint methodology are recorded here. Each
methodology version is a frozen point that one or more dataset releases were
produced under. Older versions are retained so that older releases stay
interpretable.

The version format is `YYYY.MM-<label>`.

## Unreleased

### Owner discovery: 11 new owners, and one correction to a widely repeated claim

Ranking every ABA organization in the national registry by size and researching
the owners of the largest untracked chains took the dataset from 1,325 to **1,558
clinics**, and financial owners' share of chain-run ABA clinics from 24.5% to
**28.4%**.

Newly tracked: Autism Learning Partners (FFL Partners), InBloom Autism Services
(Elysium Management, a family office), Center for Social Dynamics and Behavior
Change Institute (Goldman Sachs Alternatives), Behavior Care Specialists (Pharos
Capital Group), Mission Autism Clinics (Zenyth Partners), Kind Behavioral Health
(Trilogy Search Partners), plus Behavioral Innovations' full 130-center roster,
which the registry had reduced to four.

**A correction.** Autism Learning Partners was previously left out because trade
chatter had it sold to H.I.G. Capital and the current owner looked ambiguous. That
chatter does not survive contact with a primary source: FFL Partners' own
portfolio page still lists ALP among its current holdings, separately from that
page's labelled exits, and H.I.G.'s portfolio does not list it at all. Fundprint
follows the primary source.

**What was deliberately NOT added is as important.** A minority investment is not
ownership: the methodology requires a controlling interest, so My Favorite
Therapists (5th Century Partners), Able Kids Services (MKH Capital) and Forta
(Insight Partners) are excluded, because in each case the founders retain control.
Several large chains turned out not to be institutionally owned at all, and are
therefore correctly absent: ABA Centers of America, Golden Steps ABA, Verbal
Beginnings, Applied ABC, Bierman, Stride, Intercare, Soar Health, Cortica. Two
chains with confirmed private-equity owners are still unpublished because their
only sources cannot be fetched and content-hashed, or because the ownership chain
needed three inferential hops.

### The registry ceiling was an artifact of the API

Fundprint previously sourced clinics from the NPPES *API*, which caps any query
at 1,200 records and returns only an NPI's **primary** practice location. A chain
operating fifty centers under a handful of NPIs therefore appeared as a handful
of clinics, and this was believed to be a hard ceiling on coverage. It was not.

CMS publishes the whole registry monthly, free and without a data-use agreement,
including a **Practice Location Reference File** listing every non-primary
practice location for every NPI. That is where a chain's other centers actually
live. Ingesting it takes tracked clinics from 668 to **967**, with the gains
concentrated exactly where the API was blindest: Action Behavior Centers 71 to
212, Hopebridge 101 to 142, Acorn Health 44 to 90.

The bulk file also carries NPI deactivation dates the API does not expose, so a
dead registration is skipped rather than published as a clinic (2,408 were
skipped on the first run).

One guard was required and is worth recording. `owner_entity` holds every company
scraped from a private-equity firm's portfolio page, not only the autism ones:
KKR's portfolio yields MyEyeDr., Heartland Dental and Del Taco. Against the
taxonomy-filtered API those names never matched anything. Against the full
registry they match thousands of optometry and dental locations, and a first pass
attributed roughly 5,000 non-ABA locations to PE firms as "clinics". Only owners
explicitly flagged as ABA providers may now capture a clinic.


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
- **Non-clinic locations are no longer published as clinics** (section 2). Two
  kinds were: in-home providers, which operate no centers at all (Key Autism
  Services registers one office suite per state across fourteen states; Butterfly
  Effects registers at apartments), and two corporate headquarters registered as
  practice locations. In-home owners now contribute zero clinics, but their
  ownership stays published with an explicit label, because the ownership is true
  and sourced and deleting it would understate private-equity presence in ABA.
  Tracked clinics fall from 716 to 668, and the owner list drops to 10 with
  clinics plus two in-home owners and one former owner.
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
