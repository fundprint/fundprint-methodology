# Fundprint Methodology

**Methodology version:** `2026.07-sites-v1`
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

A clinic is a **physical service location, not a billing registration, and not
an office**. This distinction is load-bearing, and it cuts two ways.

Not every ABA provider operates centers. Some deliver therapy in the client's
home or community. They still hold National Provider Identifiers, and they
register those somewhere, so the registry hands back an address that looks like a
clinic and is not one: Key Autism Services registers exactly one NPI per state
across fourteen states, every one an office suite; Butterfly Effects registers at
apartments. **An in-home provider contributes no clinics.** Its ownership is
still recorded and published, with a clinic count of zero and an explicit label,
because the ownership is a true and sourced fact and deleting it would understate
private-equity presence in ABA.

Nor is a clinic the same thing as a registration. A chain may hold several
National Provider Identifiers at one address, sometimes under more than one
legal-entity name: Action Behavior Centers registers six NPIs at a single
Broomfield, Colorado suite. Each is a real registration, but they are one clinic.
Fundprint therefore counts distinct physical sites, identifying a site by its
street address (including suite) and ZIP. Counting NPI enumerations instead would
inflate any chain that registers many identifiers per center, and would do so
unevenly across owners, making clinic counts non-comparable between firms.

**One site is one clinic even when a parent firm's two brands are registered at
it.** A firm that buys two chains and merges them leaves both registrations
standing at the same suite. KKR registers BlueSprig and Florida Autism Center at
2437 SE 17th Street, Suite 102 in Ocala, Florida; LEARN registers Total Spectrum
and Wisconsin Early Autism Project at the same six Wisconsin streets; Zenyth
registers Helping Hands Family and Mission Autism Clinics at the same five suites.
These are not co-locations. One suite does not hold two competing centers, and
these are not competitors: they are the same owner, and the abandoned brand's
record is typically years staler than the surviving one. They are counted once.

An earlier version of this document disclosed this as a residual counting limit
instead of correcting it, reasoning that merging two separately registered legal
entities into one center was a stronger claim than the registry supported. That
was backwards, and the reasoning is recorded here because it is a trap worth
naming: declining to merge is not the neutral option. It asserts that two centers
exist at one suite, which is the claim the evidence actually contradicts. When a
choice is between two claims, "publish less" is the one that counts fewer clinics,
not the one that requires less thought.

Nor is every address a chain registers a clinic at all. A corporate headquarters
or billing office may be registered as the practice location. Fundprint excludes
such an address only on direct evidence, never on a heuristic: both exclusions in
this release are head offices that the owner's own public directory of centers
omits. A tempting rule, "many NPIs at one address means head office," was tested
against the data and rejected as false, precisely because Action Behavior Centers
registers three to four NPIs at every one of its 35 Colorado centers. Applying
that rule would have deleted dozens of real clinics.

**A multi-service provider is in scope only when ABA is a core line, not an
incidental one.** Some companies deliver ABA alongside speech therapy,
occupational therapy, psychiatry or physical therapy. Counting all of a general
behavioral-health company's locations as autism clinics would badly overstate the
dataset, so the default is to exclude them: Geode Health (a KKR-backed outpatient
mental-health provider) and Invo Healthcare are both excluded on this rule, and
neither carries a behavior-analysis taxonomy at a single location.

The test is what the provider registry says the centers actually are. Woven Care,
a Colorado pediatric therapy provider, holds 22 National Provider Identifiers
whose *primary* taxonomy is behavior analysis and carries a behavior-analysis
taxonomy at 22 of its 24 sites: it runs ABA at essentially every center, and it is
included. Elite DNA Behavioral Health, which registers as DNA Comprehensive
Therapy Services, has exactly **one** primary-ABA identifier out of 65 and is
excluded: it offers ABA here and there alongside psychiatry and talk therapy. The
distinction is not the company's self-description but its registrations.

The test cuts one way only. It can exclude a multi-service company; it can never
*include* a chain that would otherwise be out of scope, and a genuine ABA chain
that registers under no ABA taxonomy at all (Behavioral Innovations and ACES both
do this) is unaffected by it.

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
existence, that is a public provider record, drawn from two sources.

The primary source is NPPES (the CMS National Plan and Provider Enumeration
System), which lists provider organizations with a National Provider Identifier,
name, and location. Clinics are gathered from it two ways: a broad pull by
provider taxonomy, and a targeted pull by organization name for each tracked
brand. The brand-targeted pull matters because a single taxonomy query is capped
by the registry, so it cannot surface every location of a large chain; querying
each tracked brand by name gives it its own record budget and captures locations
the broad pull misses.

The second source is a tracked owner's own public location directory. NPPES
enumerates only the provider organizations a chain registers with an NPI, so it
structurally undercounts a chain that operates many centers under a few NPIs. For
owners that publish a machine-readable directory, Fundprint reads each center's
address from the page's structured data (a schema.org `MedicalBusiness` block, or
the site's semantic per-field address markup) and stages it under the honest
source type `owner_location_directory`. This is a read of published structured
data, not brittle scraping of rendered prose. These records are self-reported by
the owner, so they are treated as a supplement, not a replacement: each directory
center is de-duplicated against the NPPES clinics for the same owner by street
address and ZIP (falling back to city and state where a directory page gives no
street), so a physical center listed in both sources is counted once. The same
key de-duplicates registry records against each other, which is what keeps
several NPIs at one address from being counted as several clinics (see the clinic
definition in section 2). Centers are attributed one of two ways: where the
directory mixes several tracked brands and each center's name carries its brand,
the deterministic name matcher links it; where every center belongs to one known
owner but the pages are generically named, they are attributed to that owner
directly. A center that matches no tracked owner is left unlinked. In this
release, 652 of the 1,535 clinics come from owner directories and rosters, and
the remaining 883 from NPPES.

For an ownership link, the source is a captured primary source: an acquisition
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

## 8b. The denominator, and market share

A count invites a fair question: out of how many? Fundprint answers it from the
same federal bulk registry the clinics come from, so it can state a share rather
than only a total.

**The universe.** An organization counts as an ABA provider if it is a live
(non-deactivated) organization NPI and either carries a behavior-analysis taxonomy
or bears the name of a tracked ABA brand. The second clause matters: not every
real ABA chain registers under the ABA taxonomy, and omitting them would drop
tracked clinics out of the denominator and inflate the share.

**The one rule that makes the share honest.** The numerator and the denominator
are computed in a single pass over that one universe, with one address key, and
every figure is a count of **distinct sites**: a union, never a sum of per-group
sizes. The numerator is therefore a strict subset of the denominator and cannot
count anything the denominator does not.

That the two sentences above must both be true is not a pedantic point, and this
release is where they became true. The share was previously computed by summing
the number of sites held by each owner, and separately summing the number of sites
in each chain. Because one address can carry two brands or two legal entities, both
sums counted some addresses more than once, on both sides of the ratio. The
numerator was not a subset of the denominator but a multiset that could in
principle exceed it. Counting distinct sites moved the headline share from 28.4%
to **27.4%**, and moved the national ABA site count from 23,213 to 21,170. A rule
that is stated but not implemented is not a rule.

This has a further consequence that is stated rather than buried: clinics
Fundprint reads from an owner's own location directory are excluded from **both**
sides, because the registry cannot see them. The published clinic count is larger
than this numerator and the two are not interchangeable.

**What the market looks like.** The registry holds 17,567 ABA provider
organizations operating 21,170 distinct locations. 18,249 of those locations (86%)
belong to independents and very small practices. There are only 286 ABA chains in
the country with five or more locations, running 2,921 clinics between them.

**The two shares.** Of those chain-run clinics, 822 (**28.1%**) are held by a
financial owner Fundprint can name and source. Private equity on its own holds
23.2% of them; the remainder is a pension fund, a family office and two search
funds, which is why a headline that says "private equity" must be built on the
23.2% and not on the 28.1%. Measured against every ABA location in the country,
including the independents, the same holdings are 4.3% (3.6% for private equity
alone).

Both numbers are true and both are published. The second describes a fragmented
profession. The first describes what has happened to the part of it that
consolidated, and it is the one that answers the question the dataset exists to
ask. Quoting either without the other would mislead.

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
- **Site de-duplication now spans a parent firm's brands, and this limitation is
  retired.** Previous releases scoped de-duplication to a single owner brand and
  disclosed the resulting over-count instead of fixing it. Twenty-three addresses
  were affected (twelve under KKR, six under Gryphon's LEARN, five under Zenyth).
  They are now counted once each; see the clinic definition in section 2. The
  disclosure is kept here, struck through rather than deleted, because a reader
  comparing releases is entitled to know why a firm's count fell.
- **The provider registry reports existence-ever, not existence-now.** This is
  the most important limit on the registry-sourced half of the dataset. NPPES is
  a register of identifiers, not an inventory of open businesses: when a clinic
  closes, nothing compels anyone to deactivate its NPI, and the record goes on
  reporting an active status indefinitely. Every registry-sourced clinic in this
  release reports status `A`, including records last certified in 2008. A closed
  clinic and an open one are therefore indistinguishable in the registry, and
  some number of "ghost" clinics, centers that have closed but whose registration
  lives on, are certainly present.

  The only signal the registry offers is how long a record has gone untouched.
  Fundprint now records that date for every registry-sourced clinic. **88 of the
  883 registry-sourced clinics in this release (10%) rest on a registration that
  has not been updated or re-certified in six or more years.** That is not a
  count of ghosts, but it is the pool they are drawn from, and it is published
  rather than hidden. The proportion has fallen (it was 19% a release ago) because
  the bulk registry's practice-location file surfaces many recently re-certified
  secondary sites, not because any stale record became fresh. Clinics sourced from
  an owner's own location directory (652 of 1,535) do not have this problem: a
  directory lists the centers an owner says are open today.

  Ghosts become provable in one situation: when a chain under a *different*
  parent firm registers at the same street address. Two competing chains do not
  operate from one suite, so the one that left is the one whose registration is
  years staler. Seven such clinics were identified in this release, with staleness
  gaps of two to seven years (for example, a Hopebridge record at a Fort Collins,
  Colorado suite last touched in 2020, where an Action Behavior Centers record was
  touched in December 2025), and each was quarantined out of publication. That
  detector only fires when a competitor Fundprint also tracks happens to move in,
  so seven is a floor on the phenomenon, not a measure of it.
- **A tracked owner's clinic count can be far below its real footprint, and the
  dataset will not guess.** Some chains deliver much of their care in homes and
  schools, register a single organization NPI, and publish "service area" pages
  rather than centers: a page per city, carrying a city name and no street
  address. Autism Learning Partners, one of the largest ABA providers in the
  country and a tracked FFL Partners holding, publishes 199 such pages and
  appears here with **one** clinic. That number is not a mistake and it is not
  the company's size. It is what the public record establishes about its physical
  locations. Counting service areas as clinics would inflate the dataset with
  places that are not clinics, so Fundprint publishes the ownership (which is
  sourced) and leaves the count where the evidence leaves it. This is the sharpest
  illustration of the rule stated in Section 1: coverage, not census.
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

- **Clinics tracked:** 1,559
- **Current owners with tracked clinics:** 19, plus one former owner and two
  in-home owners (which operate no centers), all shown with a clinic count of
  zero and an explicit label
- **States covered:** 43
- **Clinic-existence sources:** 907 clinics come from the NPPES provider registry
  and 652 from owners' own public location directories and rosters (see section
  8). Clinics from both sources are de-duplicated on the same key, street address
  plus ZIP within a parent firm, so a center listed in both sources is counted
  once, several NPIs at one address are counted once, and two of one firm's brands
  at one address are counted once.
- **Registry freshness:** 88 of the 907 registry-sourced clinics (10%) rest on a
  registration not updated in six or more years. The registry never marks a
  closed clinic closed, so this is the honest measure of how much of the dataset
  could be stale. Seven clinics whose registration was provably dead (the address
  is now registered to a chain under a different parent firm, with a two-to-seven
  year staleness gap) were quarantined out of this release. See section 9.
- **Market share:** 822 of the country's 2,921 chain-run ABA clinics (**28.1%**),
  and 4.3% of all 21,170 ABA sites. Private equity alone holds 23.2% of chain-run
  clinics. See section 8b, and do not quote one of these without the others.
- **Method breakdown:** every published clinic-to-owner link in this release is
  a high-confidence name match (`fuzzy_high`), and every owner-to-parent link is
  an `exact_match` against a named primary source. No `llm_inferred` claims are
  present in this release.

Current owners, by owner type and tracked clinic count:

| Parent firm                    | Owner type      | Clinics tracked |
|--------------------------------|-----------------|-----------------|
| KKR                            | private equity  | 246             |
| Charlesbank                    | private equity  | 212             |
| Gryphon Investors              | private equity  | 152             |
| Tenex Capital Management       | private equity  | 150             |
| Arsenal Capital Partners       | private equity  | 142             |
| Nautic Partners                | private equity  | 135             |
| Ontario Teachers' Pension Plan | pension fund    | 90              |
| General Atlantic               | private equity  | 81              |
| GTCR                           | private equity  | 79              |
| Zenyth Partners                | private equity  | 64              |
| NexPhase Capital               | private equity  | 41              |
| Petra Capital Partners         | private equity  | 37              |
| Elysium Management             | family office   | 29              |
| Pharos Capital Group           | private equity  | 20              |
| Goldman Sachs Alternatives     | private equity  | 20              |
| Anacapa Partners               | other           | 24              |
| Thomas H. Lee Partners         | private equity  | 18              |
| Trilogy Search Partners        | other           | 18              |
| FFL Partners                   | private equity  | 1               |
| Moran Capital Partners         | family office   | 0 (in-home)     |
| Cane Investment Partners       | other           | 0 (in-home)     |

FFL Partners' single clinic is not an error. See the limitation in section 9 on
Autism Learning Partners: the company is one of the largest ABA providers in the
country, and the public record establishes exactly one physical location for it.

Moran Capital Partners (Butterfly Effects) and Cane Investment Partners (Key
Autism Services) own providers that deliver therapy in the client's home and
operate no centers. Their clinic count is zero because they have no clinics, not
because none were found. The ownership is documented and published.

Gryphon Investors appears through LEARN Behavioral, which runs as a federation
of separately named ABA brands (for example Autism Spectrum Therapies, Trellis
Services, and the Behavior Analysis Center for Autism). Each brand is counted
under its own name and attributed to Gryphon through LEARN. Brands whose names
collide with unrelated organizations in the provider registry are deliberately
left out rather than risk over-counting, so this figure understates LEARN's true
footprint.

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
