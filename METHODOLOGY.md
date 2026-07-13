# Fundprint Methodology

**Methodology version:** `2026.07-directory-v2`
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

**One address, spelled two ways, is one address.** The provider registry shouts
abbreviations ("18301 N 79TH AVE, BUILDING A STE 101") and an owner's own directory
writes them out ("18301 N 79th Avenue, Building A, Suite 101"). Stripping
punctuation is not enough to see that these are the same suite, and until this
release they were two clinics. Forty-four real sites were counted twice, once per
source, across Hopebridge, BlueSprig, Florida Autism Center, Proud Moments, ACES
and others. The site key now canonicalizes USPS street suffixes, directionals and
unit markers, so `AVE`/`Avenue`, `STE 101`/`Suite 101`/`#101` all key alike.

This is a canonicalization, not a fuzzy match, and it must never become one. Every
pair folded together is an abbreviation and its expansion. Anything looser, such as
dropping the unit, would merge two genuinely different clinics in one office park,
which is the exact failure the site key exists to prevent.

**Where an owner's own directory and the registry disagree about whether a centre
exists, the directory wins.** A registration is filed once and nobody ever revokes
it. A directory is what the company tells parents today. Action Behavior Centers
forced this rule: its directory lists 414 centres, and the registry gave us 206
rows, of which 105 were addresses ABC does not list anywhere. Nine were in Ohio,
three in Virginia and one in Georgia, states it does not operate in. One was an
apartment in Palm Beach Gardens. One was its own corporate headquarters at 6300
Bee Caves Road, filed as a "practice location" on the very NPI that carries its
109 real centres as secondary locations. Thirty-four were in Colorado, where the
registry gave us 67 against ABC's 42: centres that closed and were never
deregistered, exactly as section 9 describes.

The rule now runs against every owner that publishes a complete directory, and it
has found the same three faults each time. **Acorn Health** falls from 90 rows to
the 70 it lists, **Hopebridge** from 145 to the 101 it lists. Both were inflated by
closed centres (the registry places Hopebridge in Colorado, Arkansas, Texas and
seven other states its directory does not operate in), by head offices filed as
practice locations (Acorn's Coral Gables headquarters, published twice), and by
registrations belonging to *other companies* whose names merely begin the same way:
`HOPE BRIDGE COUNSELING LLC`, `HOPE BRIDGE LIVING LLC` and `ACORN HEALTHCARE
SERVICES INCORPORATED` are unrelated businesses that a name-prefix match cannot
tell apart from the chains. None of the three needed to be handled as a special
case. Reading the owner's own list removes all of them at once.

The rule applies only where an owner publishes a *complete* directory, and it
quarantines rather than deletes: the registration is real, and what is false is
the inference that a registration is a centre. Where the two sources agree, the
registry row is superseded by the directory row rather than counted alongside it.
A directory that covers only one region must never be used this way, because it
would quarantine real clinics elsewhere. Completeness is checked rather than
assumed: Hopebridge's sitemap lists 114 centre pages, which reconcile exactly as
the 103 centres its index carries plus 11 regional landing pages.

Declining to apply the rule is not the neutral option. Leaving the registry rows in
place asserts that a chain still runs a centre its own website does not list, which
is the claim the evidence contradicts.

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
release, 1,045 of the 1,705 clinics are attested by an owner directory or roster,
and the remaining 660 rest on NPPES alone.

A directory has to be read at the leaf. Autism Learning Partners publishes state
and county index pages carrying no street address, and for a release it was
recorded as having one clinic on that basis. The 59 pages *beneath* those indexes
are individual centers, each publishing its address. "This directory has no
addresses" is a claim about the page that was actually opened, and the page to open
is the last one.

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
principle exceed it. In the release where the fix landed, counting distinct sites
moved the headline share from 28.4% to 27.4% and cut the national ABA site count
from 23,213 to 21,170. Both figures have moved again since, as owners were added
and the universe was recomputed; section 10 carries the current values, and the
point here is the rule, not the release. A rule that is stated but not
implemented is not a rule.

This has a further consequence that is stated rather than buried: clinics
Fundprint reads from an owner's own location directory are excluded from **both**
sides, because the registry cannot see them. The published clinic count is larger
than this numerator and the two are not interchangeable.

### The chain-run share, and why it was withdrawn

Until this release Fundprint headlined a **share of chain-run clinics**, where a
chain was any ABA operator with five or more locations. That number is withdrawn.
It is not corrected, not restated, and not replaced by another ratio of the same
kind. Three things were wrong with it, and the second is disqualifying.

1. **The threshold was arbitrary.** Five, not three, not ten. Nothing in the data
   chose it, and no sensitivity analysis defended it. A share whose value depends
   on an unargued cut is an editorial choice wearing the costume of a measurement.
2. **The denominator was endogenous to the thing being measured.** An operator is
   a "chain" *because* it has many locations, and a great many of them have many
   locations *because private equity rolled them up*. Private equity's own buying
   therefore inflated the numerator and the denominator at the same time. Consider
   the limit case: a firm that bought forty four-site operators and merged them
   into one forty-site chain would barely move its "share of chain-run clinics"
   while its real market power exploded, because it had manufactured its own
   denominator. **The measure was partly blind to the thing it existed to measure.**
3. **Nothing else in the literature uses it,** so the figure could not be compared
   with, or checked against, any other estimate. A number nobody can argue with is
   a number nobody can use.

Fundprint's governing rule is that a claim ships only if it can be defended in
front of a journalist, an academic, or a Senate staffer. The chain-run share could
not survive the second question from any of the three.

**What the market looks like, without a threshold.** The registry holds 17,567 ABA
provider organizations operating 21,083 distinct locations. Rather than cut that
population at a number of our choosing, the release publishes the whole
operator-size distribution:

| Locations per operator | Operators | Locations |
|---|---|---|
| 1 | 15,141 | 13,939 |
| 2-4 | 2,145 | 4,867 |
| 5-9 | 191 | 1,137 |
| 10-24 | 76 | 1,033 |
| 25+ | 14 | 774 |

(Locations are a union, not a sum: two operators can share one address.) A reader
who wants a chain share can compute one from this table, and in doing so must
state their own threshold out loud. That is the point.

**The share, which needs no threshold.** Of the 21,083 ABA locations in the
country, Fundprint can name and source the owner of **753 (3.6%)**, of which
**628 (3.0%)** are held by private equity. These are the only national shares
published, because they are the only ones that require no choice.

The numerator is the published dataset, intersected with that same registry
universe: not a second, unreviewed name match against the registry, which is what
it used to be and which quietly re-imported every clinic the corrections in section
9 had removed. Because the numerator is corrected and the denominator cannot be
(Fundprint reads its own owners' directories, not those of the other 17,000
operators), the published share is a **floor**. Closed centers have been taken out
of the top of the fraction and remain in the bottom. That is the conservative
direction, and it is disclosed rather than adjusted, because adjusting it would
mean guessing at the rest of the market's staleness.

**Where private equity actually is.** A national share of a profession this
fragmented says little about market power, because care is bought locally: no
family chooses between a clinic in Denver and one in Tampa. So the release also
publishes private equity's share of the ABA locations **within each state**, which
requires no threshold either. The most concentrated are Minnesota (23 of 139
locations, 16.5%), New Mexico (13 of 106, 12.3%), Arizona (51 of 529, 9.6%),
Pennsylvania (38 of 431, 8.8%) and Colorado (55 of 688, 8.0%). States with fewer
than 25 ABA locations are not ranked, because a percentage of a handful of clinics
is noise; they are still counted in every national figure.

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
- **Name-prefix matching can over-capture, and it demonstrably did.** A brand-name
  prefix match can link a registry record that shares a leading name with an owner
  brand but is not actually part of it. This is not hypothetical. Normalising
  "Hope Bridge" yields `hopebridge`, which is a leading substring of `HOPE BRIDGE
  COUNSELING LLC`, `HOPE BRIDGE LIVING LLC`, `HOPE BRIDGE CARE LLC` and `HOPE
  BRIDGE HEALTH SERVICES LLC`; "Acorn Health" is a leading substring of `ACORN
  HEALTHCARE SERVICES INCORPORATED` and `ACORN HEALTH SOLUTIONS`. These are
  unrelated businesses, and until this release they were published as
  private-equity-owned clinics. They were removed by reading the two chains' own
  directories, which list neither.

  The general lesson is that the matcher cannot police itself: the string is all it
  has, and two companies can share one. What catches this class of error is an
  independent statement of what the owner operates, which is exactly what an owner's
  directory is. Where a tracked owner publishes no complete directory, this risk is
  live and undetected, and that is stated rather than glossed.

  The same match can also capture a correctly-identified owner that is out of scope:
  an early release attached six Geode Health records (a KKR-backed outpatient
  mental-health provider, not an ABA or autism provider) to KKR. Those were held out
  of publication and the linker is guarded against re-capturing them. Conversely, a
  spelling variant or an abbreviated legal name can cause a real clinic to be
  missed. Both directions of error are possible, and the confidence floor, the
  out-of-scope holds, the owner directories and the hand-validation gate are the
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
  Fundprint records that date for every registry-sourced clinic. **Of the 355
  registry-sourced clinics whose registration can be looked up by NPI, 62 (17%)
  rest on a record that has not been updated or re-certified in six or more
  years.** That is not a count of ghosts, but it is the pool they are drawn from,
  and it is published rather than hidden.

  The exposure is now much smaller than it was, and for a better reason than
  before. **1,045 of the 1,705 clinics are attested by an owner's own current
  directory**, and those cannot be ghosts by construction: a directory lists the
  centers an owner says are open today. Only the 660 registry-only clinics carry
  the risk at all. Better still, where an owner publishes a complete directory the
  problem is no longer merely *disclosed* but *removed*: the directory decides what
  the owner operates, and every registration it does not list is quarantined. That
  is how 34 closed Action Behavior Centers registrations in Colorado, and
  Hopebridge's entire registered presence in Colorado and Arkansas, left this
  release. The residual ghost pool is therefore roughly 110 clinics, not a third of
  the dataset.

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
  address. Key Autism Services publishes 195 such pages. A service area is not a
  place, so it is not counted, and the ownership is published with a clinic count
  of zero and an explicit label rather than filled in with guesses.

  **A previous release applied that rule to Autism Learning Partners and got it
  wrong, and the correction is worth stating plainly because the failure is easy to
  repeat.** ALP was recorded here as publishing 199 address-less service-area pages
  and appearing with **one** clinic, and that was described as correct rather than
  as a gap. It was not correct. Its 199 state and county pages are indeed indexes
  with no street address, but the 59 pages *beneath* them are individual centers,
  each publishing its own address in machine-readable form. ALP now stands at
  **45**. Six of its leaf pages genuinely are city-only and remain excluded.

  The rule survives; the reasoning that was applied to it did not. "This directory
  publishes no addresses" is a claim about the page that was actually opened, and
  the page to open is the last one in the tree, not the index that links to it. A
  count left low out of caution is still a wrong count, and caution is not a reason
  to stop looking.
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

- **Clinics tracked:** 1,705
- **Current owners with tracked clinics:** 19, plus one former owner and two
  in-home owners (which operate no centers), all shown with a clinic count of
  zero and an explicit label
- **States covered:** 44
- **Clinic-existence sources:** 1,045 of the 1,705 clinics are attested by an
  owner's own public location directory or roster; the remaining 660 rest on the
  NPPES provider registry alone (see section 8). Clinics from both sources are
  de-duplicated on the same key, street address plus ZIP within a parent firm, so
  a center listed in both sources is counted once, several NPIs at one address are
  counted once, and two of one firm's brands at one address are counted once.
- **Registry freshness:** of the 355 registry-sourced clinics whose registration
  can be looked up by NPI, 62 (17%) rest on a record not updated in six or more
  years. The registry never marks a closed clinic closed, so this is the honest
  measure of how much of the registry-only remainder could be stale. Where an owner
  publishes a complete directory, this problem is not measured but removed: the
  directory decides what the owner operates, and registrations it does not list are
  quarantined. See section 9.
- **Market share:** of the 21,083 ABA locations the registry lists, Fundprint can
  name the owner of **753 (3.6%)**, of which **628 (3.0%)** are private-equity
  held. This is a floor, not a point estimate: closed centers are removed from the
  numerator, because owners' directories reveal them, and cannot be removed from
  the denominator, because the other 17,000 operators' directories are not read.
  There is **no chain-run share**: it was withdrawn because its five-site threshold
  was arbitrary and its denominator was inflated by the very buying it purported to
  measure. See section 8b.
- **Where it is concentrated:** private equity holds 16.5% of Minnesota's ABA
  locations (23 of 139), 12.3% of New Mexico's (13 of 106), 9.6% of Arizona's
  (51 of 529), 8.8% of Pennsylvania's (38 of 431) and 8.0% of Colorado's (55 of
  688). Care is bought locally, so the state figure means more than the national
  one.
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
