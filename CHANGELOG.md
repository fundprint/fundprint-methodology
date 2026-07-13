# Changelog

All notable changes to the Fundprint methodology are recorded here. Each
methodology version is a frozen point that one or more dataset releases were
produced under. Older versions are retained so that older releases stay
interpretable.

The version format is `YYYY.MM-<label>`.

## 2026.07-sites-v1

Governs dataset release `2026.07-beta` and later, until superseded. The
confidence floors are unchanged from `2026.06-floors-v0`. What changed is the
definition of a countable site, which is a change to a countable relation and so
requires a version bump under section 12.

### One site is one clinic, even across a parent firm's brands

A firm that buys two chains and merges them leaves both registrations standing at
the same suite. KKR registers BlueSprig and Florida Autism Center at 2437 SE 17th
Street, Suite 102 in Ocala; Gryphon's LEARN registers Total Spectrum and Wisconsin
Early Autism Project at the same six Wisconsin streets; Zenyth registers Helping
Hands Family and Mission Autism Clinics at the same five suites. Site
de-duplication was scoped to a single brand, so each of these counted twice.

Twenty-three sites were affected and are now counted once. **Tracked clinics fall
from 1,558 to 1,535**, and the ranking moves: KKR 258 to 246, Gryphon 158 to 152,
Zenyth 69 to 64. Nothing was deleted; the duplicate row is marked superseded and
points at the survivor, and its underlying registration stays published as a claim.
The surviving row is chosen on evidence, not on brand: a center the owner lists in
its own directory beats a registry record, and between two registry records the
freshly re-certified one beats the one abandoned years ago.

The previous release disclosed this as a residual counting limit rather than
correcting it, on the reasoning that merging two separately registered legal
entities into one center was a stronger claim than the registry supported. That
was backwards, and the error is worth naming because it is a trap: **declining to
merge is not the neutral option.** It asserts that two centers exist at one suite,
which is the claim the evidence actually contradicts, and the abandoned brand's
record is typically years staler than the survivor's. "Publish less" means publish
the smaller count, not the one that takes less thought.

### The market share was computed as a sum of groups, not a count of sites

Section 8b has always claimed that the numerator and denominator are computed over
one universe with one site key, so that the numerator is a strict subset of the
denominator. The implementation did not do that. It summed the number of sites
held by each owner, and separately summed the number of sites in each chain.
Because one address can carry two brands or two legal entities, both sums counted
some addresses more than once, on both sides of the ratio, and the numerator was
not a subset of the denominator but a multiset that could in principle exceed it.

Every figure is now a count of distinct sites. The effects, all in the direction of
a smaller and more defensible claim:

- The headline share of chain-run clinics falls from **28.4% to 27.4%** (799 of
  2,921).
- The national ABA site count falls from 23,213 to **21,170**, and chain-run sites
  from 3,070 to 2,921. The registry itself carries roughly two thousand addresses
  shared between legal entities.
- The share of *all* ABA sites rises from 3.9% to **4.2%**, because the denominator
  shrank proportionally more than the numerator did.

**Private equity alone is now reported separately: 23.2% of chain-run clinics.**
The 27.4% figure includes a pension fund and a family office, so a headline that
says "private equity" has to be built on the smaller number. The public site and
the social card were both corrected accordingly; the card previously led with the
share of Fundprint's own dataset that is PE-owned (about 91%), which is a fact
about our coverage and not about the market, and which is exactly the misreading
this methodology exists to prevent.

A rule that is stated but not implemented is not a rule.

### Owner discovery: 11 new owners, and one correction to a widely repeated claim

Ranking every ABA organization in the national registry by size and researching
the owners of the largest untracked chains took the dataset from 1,325 to 1,558
clinics, before the site-counting correction above brought it to its published
figure of **1,535**.

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


### Corrections to the clinic definition, and the counts that follow from it

The confidence floors are unchanged throughout.

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
