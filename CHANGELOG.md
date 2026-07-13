# Changelog

All notable changes to the Fundprint methodology are recorded here. Each
methodology version is a frozen point that one or more dataset releases were
produced under. Older versions are retained so that older releases stay
interpretable.

The version format is `YYYY.MM-<label>`.

## 2026.07-directory-v2

Governs dataset release `2026.07-beta` and later. The previous release established
that an owner's own complete directory beats the provider registry, and applied it
to one owner. This release applies it to every owner that publishes one, and fixes
the market share, which was still being computed from the registry and so still
counted the clinics the rule had just removed.

Published clinics move from 1,721 to **1,705**, private-equity clinics to
**1,564**, and states covered to **44**. The published private-equity share of all
ABA locations moves from 3.5% to **3.0%**.

### Three more owners now read from their own directories

Autism Learning Partners, Acorn Health and Hopebridge published complete center
lists that Fundprint was not reading. Reading them changed all three, and in both
directions at once, exactly as it did for Action Behavior Centers:

| Owner | Was | Now | Why |
|---|---|---|---|
| Autism Learning Partners | 1 | 45 | 44 centers the registry never saw |
| Acorn Health | 90 | 70 | directory lists 70; the rest are stale, duplicated or other companies |
| Hopebridge | 145 | 101 | directory lists 101; ditto |

**Autism Learning Partners corrects an error of our own.** ALP was previously
recorded as a chain that publishes service-area pages with no street address, and
was left at one clinic deliberately. That was true of its state and county index
pages. It was not true of the 59 center pages *beneath* them, each of which
publishes a street address. The lesson is general and is now written into the
method: "this directory has no addresses" must be checked at the leaf page, not at
the index that links to it. Its six genuinely address-less city pages are still
excluded, because a service area is not a clinic.

**Acorn Health and Hopebridge were inflated by three separate faults**, each of
which the directory rule removes without needing to be special-cased:

- *Registrations for other companies.* The name matcher collapses "Hope Bridge" to
  `hopebridge`, which is a prefix of `HOPE BRIDGE COUNSELING LLC`, `HOPE BRIDGE
  LIVING LLC` and `HOPE BRIDGE CARE LLC`; likewise `ACORN HEALTHCARE SERVICES
  INCORPORATED` for Acorn. These are unrelated businesses and were being published
  as private-equity-owned clinics.
- *Closed centers.* The registry showed Hopebridge in Colorado (12), Arkansas (8),
  Texas, Massachusetts, Maryland, Minnesota and five other states. Hopebridge's own
  directory operates in none of them. The registry reports existence-ever.
- *Head offices.* Acorn's Coral Gables headquarters was filed as a practice
  location and published twice, as was Hopebridge's home office.

Quarantining a registry row is safe wherever the directory is complete, because a
real center survives via its directory row. That bar is checked, not assumed:
Hopebridge's sitemap lists 114 center pages, which reconcile exactly as 103 centers
plus 11 regional landing pages, and its index carries all 103.

### The market share was computed from the registry, so it ignored all of this

`compute_market_share.py` built its numerator by matching owner names against the
NPPES archive directly, and never consulted the published dataset. Every correction
above was therefore invisible to it. Arkansas was published as 7.6% private equity
on the strength of eight Hopebridge centers Hopebridge does not operate, and
`HOPE BRIDGE COUNSELING LLC` was counted toward private equity's share of the
market.

The numerator is now **the published dataset**, intersected with the registry
universe. One correction now moves the map, the owner tables and the share
together, which is the only way the three can agree.

Two details make that intersection honest:

- An owner and the registry often spell one building differently (`551 36th St SE
  Unit 2` against `551 36th St. SE`). Requiring an exact site-key match would have
  dropped 94 real sites, 82 of them private-equity, and understated the share. The
  visibility test is therefore made at building level. **This does not relax the
  site key**, which still keeps the unit, because two suites in one office park are
  two clinics. It resolves to the registry's key, never ours, so the numerator
  stays a strict subset of the denominator, and a building the registry splits into
  several suites is skipped rather than guessed at.
- **The correction is applied to the numerator and cannot be applied to the
  denominator.** Fundprint removes closed centers from its own count because it can
  read its owners' directories; it cannot read the directories of the other 17,000
  operators, whose closed centers therefore remain in the 21,083. The published
  private-equity share is consequently a floor, not a point estimate. This is the
  conservative direction, and it is disclosed rather than corrected, because
  correcting it would mean guessing.

### Two parse faults that were putting wrong addresses in the data

Both were caught before publication and both are now refused rather than guessed:

- An address with no comma before the city (`7041 Transit Rd East Amherst New York
  14051`) cannot be split by rule: the last number is the house number, so peeling
  trailing words takes the street name into the city and leaves the street as
  `7041`. Where the source states the city separately (Acorn's post titles,
  Hopebridge's card headings, ALP's URLs) the split is now a subtraction rather than
  a guess; where it does not, the row is dropped.
- A directory entry is not always an address. Acorn lists Alpena, Michigan with the
  text "In-home services available now" where the street belongs, which parsed
  cleanly and staged a clinic whose street was that sentence. A street must now
  carry a house number.

## 2026.07-directory-v1

Governs dataset release `2026.07-beta` and later. Two changes to what counts as a
clinic, and they pull in opposite directions: one merges rows that were the same
site, the other replaces a whole owner's registry footprint with its own directory.
Published clinics move from 1,559 to **1,721**.

### One address, spelled two ways, was two clinics

The site key stripped punctuation but did not canonicalize abbreviations, so the
registry's `18301 N 79TH AVE, BUILDING A STE 101` and the owner's own
`18301 N 79th Avenue, Building A, Suite 101` were two different clinics. **44 real
sites were being counted twice**, once from each source, across Hopebridge, Blue
Sprig, Florida Autism Center, Proud Moments, ACES, Trumpet, Key Autism, BACA and
Action Behavior Centers. They are now one row each; the losers are superseded, not
deleted.

The key now folds USPS street suffixes, directionals and unit markers (`AVE`/
`Avenue`, `STE 101`/`Suite 101`/`#101`). It remains a canonicalization and not a
fuzzy match: every pair folded together is an abbreviation and its expansion, and
two different suites in one building stay two clinics, which is the whole reason
the unit is in the key.

This also shrank the national denominator, from 21,172 ABA locations to **21,083**:
the registry was double-counting addresses against itself for the same reason.

### Action Behavior Centers: 212 registry rows became 413 sourced ones

ABC publishes 414 centres. The registry gave us 212, and they were wrong in **both
directions at once**:

- **It missed 212 Texas centres.** ABC's directory lists 227 in Texas; NPPES gave
  us 15. A chain does not separately register every centre, and the registry cannot
  see what was never filed.
- **It invented about 50.** Nine clinics in Ohio, three in Virginia, one in
  Georgia, none of which are states ABC operates in. An *apartment* in Palm Beach
  Gardens. ABC's own corporate headquarters at 6300 Bee Caves Road, filed as a
  "practice location" on the same NPI that carries its 109 real centres as
  secondary locations. And 34 in Colorado, where ABC lists 42 and the registry gave
  us 67: centres that closed and were never deregistered.

So this release adds an ABC directory source and applies a new rule: **where an
owner's own complete directory and the registry disagree about whether a centre
exists, the directory wins.** A registration is filed once and never revoked; a
directory is what the company tells parents today. The 105 contradicted registry
rows are quarantined, not deleted, and every real centre survives because the
directory row carries it.

ABC's per-state counts now match its own published directory exactly.

### Also

- HTML entities were reaching the database unescaped from JSON-LD addresses
  ("Suite 100 &amp; 400"). Nineteen clinic addresses are corrected, and the parser
  now unescapes the address, not just the name.

## 2026.07-no-threshold-v1

Governs dataset release `2026.07-beta` and later, until superseded. No clinic,
owner or ownership claim changed in this release. What changed is that a
published number was **withdrawn**, which is a change to a countable relation and
so requires a version bump under section 12.

### The chain-run share is withdrawn

Fundprint headlined a **share of chain-run clinics**: 28.1% of clinics run by ABA
operators with five or more locations were held by a financial owner, and 23.2%
by private equity specifically. That number is gone. It is not restated at a
different threshold and it is not replaced by another ratio of the same kind.

It was withdrawn for three reasons, and the second is the one that made it
indefensible:

1. **The five-site threshold was arbitrary.** Nothing in the data chose it, and no
   sensitivity analysis defended it. A share whose value depends on an unargued cut
   is an editorial choice wearing the costume of a measurement.
2. **The denominator was endogenous to the thing being measured.** An operator is a
   "chain" because it has many locations, and many of them have many locations
   because private equity rolled them up. Private equity's own buying therefore
   inflated the numerator and the denominator together. A firm that bought forty
   four-site operators and merged them into a single forty-site chain would barely
   move its "share of chain-run clinics" while its real market power exploded,
   because it had manufactured its own denominator. The measure was partly blind to
   the thing it existed to measure.
3. **No one else uses the measure,** so it could not be compared with or checked
   against any other estimate.

Fundprint publishes a claim only if it can be defended in front of a journalist, an
academic, or a Senate staffer. This one could not survive the second question from
any of the three, and the fact that it was the most flattering number in the
dataset is precisely why it had to go.

### What replaces it: facts, not another ratio

- **The whole operator-size distribution** is published (15,133 operators run one
  location; 2,148 run 2-4; 195 run 5-9; 77 run 10-24; 14 run 25 or more). A reader
  who wants a chain share can compute one, and must state their own threshold out
  loud to do it.
- **The national share, which needs no threshold:** of 21,172 ABA locations,
  Fundprint names the owner of 915 (4.3%), of which 754 (3.6%) are private-equity
  held.
- **Per-state shares,** because a national share of a profession this fragmented
  says little about market power. Care is bought locally. Private equity holds
  17.3% of Minnesota's ABA locations, 14.1% of Colorado's, 12.3% of New Mexico's,
  11.3% of Arizona's and 10.7% of Pennsylvania's. States with fewer than 25 ABA
  locations are not ranked, because a percentage of a handful of clinics is noise.

### Consequences for anyone quoting Fundprint

Any citation of "28.1% of chain-run clinics" or "23.2% of chain-run clinics"
should be considered withdrawn and should not be repeated. The headline figure is
now a count (private equity owns 1,398 of the 1,559 clinics traced, across 43
states) and the only national shares are 4.3% and 3.6% of all ABA locations.

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

### One new owner, and five chains ruled out

Ranking the registry's remaining untracked chains by size left 69 with ten or more
locations. Working the largest of them produced one addition and, more usefully,
five firm exclusions that no longer need re-researching.

**Added: Anacapa Partners (Woven Care), 24 clinics.** Woven Care, formerly the
Shandy Clinic, registers every Colorado center as "Buck Jack LLC", which is why
it had never been matched to a brand. Anacapa's own portfolio page lists Shandy
Clinic as a current operating company, acquired through the search vehicle Buck
Jack Capital. Recorded as `other` rather than private equity, because a search
fund is neither a buyout fund, a pension, nor a family office. It is the first
owner admitted under the new multi-service rule in section 2: Woven Care also
provides speech, occupational and physical therapy, but it holds 22 identifiers
whose primary taxonomy is behavior analysis and runs ABA at 22 of its 24 sites.

Chain-run share moves from 27.4% to **28.1%**. Private equity's own share is
unchanged at 23.2%, because Anacapa is not a private-equity firm, and this is the
point of labelling owner types honestly rather than folding them all into "PE".

**Ruled out, with reasons, so they stay ruled out:**

- **Center for Autism and Related Disorders (CARD)** still has around twenty live
  registrations, which invited the question of who owns them now. Nobody
  institutional does. When CARD went bankrupt in 2023, the bankruptcy court split
  it: Pantogran, a vehicle led by founder Doreen Granpeesheh, took ten state
  markets for $37.4m, and the rest went to portfolio companies of Audax Group for
  $11.1m. CARD today is founder-owned. Blackstone remains in this dataset as a
  former owner with a clinic count of zero, which is the correct treatment.
- **DNA Comprehensive Therapy Services** (62 sites) is Elite DNA Behavioral Health,
  a general behavioral-health provider. One of its 65 identifiers has behavior
  analysis as its primary taxonomy. Out of scope under the multi-service rule.
- **Associates in Pediatric Therapy** (32 identifiers) is a speech and occupational
  therapy provider; two of its sites carry an ABA taxonomy. Out of scope.
- **Behavior Analysis Support Services** (22 Florida sites) is a genuine ABA chain,
  founder-led since 2003, with no institutional owner in any public source.
- **Woven Care's** own acquirer chain was checked against **Proud Moments**, which
  the bankruptcy reporting names as an Audax portfolio company. That is history,
  not an error: Audax held Proud Moments from 2019 and sold it to Nautic Partners
  in February 2025. Fundprint's attribution of Proud Moments to Nautic is correct.

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
