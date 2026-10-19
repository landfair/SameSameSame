# Homogenometry — measurement roadmap

Working notes on what the tool measures and what needs fixing. Written 2026-09-28.

## The premise, stated plainly

This is not an AI detector. It measures semantic and stylometric similarity between every pair
of responses in a cohort, and surfaces pairs whose resemblance is improbable enough to be worth
a conversation. Showing a student that their work shares a specific, unlikely fingerprint with
someone else's is not the same as accusing them of anything, and the tool should never be built
as though it were.

Two consequences follow, and they shape everything below:

1. **The claim is about improbability, so the null distribution is load-bearing.** "Statistically
   nearly impossible" is only as good as the model of what ordinary looks like. Most of the
   outstanding work is there, not in adding measures.

2. **The evidence is a conjunction, not a magnitude.** One elevated metric is weak. Several
   *independent kinds* of similarity at once is strong. But that argument only holds if the
   kinds really are independent, and in the current metric set they mostly are not — twelve
   variations on lexical overlap are one kind of evidence counted twelve times.

Accepted blind spot: the pairwise approach only sees convergence, so it only works when more
than one person in a cohort does the same thing. That is a known and accepted limit, not a bug
to engineer around. It also means the tool works best on large cohorts, which is where the N²
cost bites.

## Where the code actually stands

- `pair.familyZ` already holds the per-family vector (index.html:14067). The six scores exist.
- That vector is collapsed to a single weighted average at index.html:14081, and the scalar is
  what drives ranking and display. The collapse is what destroys the conjunction information.
- Within-family combination at index.html:14046 is a blind weighted mean of z-scored metrics.
  Every metric contributes at full weight whether or not it works in this cohort, so a degenerate
  metric dilutes rather than abstains.
- `diagnoseMetricSignal()` (index.html:9678) judges a metric by how far its top pairs sit above
  the typical pair, and `suggestFamilyWeights()` (index.html:9752) turns that into weights. This
  is circular: it selects metrics *because* they flagged the pairs that are about to be flagged.

## Family inventory

Six families, badly unbalanced:

| Family | Metrics | Note |
|---|---|---|
| content | 12 | some genuine n-order length diversity |
| semantic | 1 | single point of failure |
| syntax | 10 | some genuine n-order length diversity |
| structure | 1 | single point of failure; degenerates below 2 passages a side |
| style | ~17 | rate-based, so uniformly degraded at short length, not diversified |
| surface | 6 | `ttr`/`mattr` is the one deliberate length-robustness pair |

Redundancy inside a family only buys robustness if the family score *selects* among its metrics.
Averaged blindly, redundancy buys dilution and costs compute.

---

## Phase 1 — Diagnostics layer — DONE

Nothing downstream can be built until these numbers exist. All computable from
`window.__lastPairMetrics`.

- [x] **1.1** Per-metric cohort verdicts: healthy / degenerate / saturated / starved /
      length-confounded, each with its reason. Keep starved distinct from degenerate — a starved
      metric is uninformative for most pairs but potentially decisive when it fires, so its zeros
      mean "no material", not "no similarity". Do not drop it.
- [x] **1.2** Remove the circularity. Verdicts that gate scoring must come from cohort-level
      distribution shape only. "Which metrics fired hardest" survives as description and is
      explicitly barred from setting weights.
- [x] **1.3** Length-confound test: rank-correlate each metric's pairwise value against
      `|len1 - len2|` and `min(len1, len2)`. Strong correlation means the metric is partly
      measuring length rather than authorship.
- [x] **1.4** Empirical redundancy: metric-to-metric within family, and family-to-family across
      the six. Report how many independent views the six families actually represent.
- [x] **1.5** Assignment openness: median pairwise unweighted lexical overlap. High median means a
      closed-ended prompt. Read *unweighted* content evidence down — but rarity-weighted content
      evidence gets **sharper** under a closed prompt, not weaker, so do not discount the family
      wholesale.

## Phase 2 — Fix within-family scoring — DONE

- [x] **2.1** Select before averaging: exclude degenerate and saturated metrics from the family
      score. This is the change that converts redundancy from dilution into robustness.
- [x] **2.2** Replace the weighted mean of z-scores (index.html:14046) with percentile-then-median
      over the surviving metrics. Percentiles make differently-scaled and differently-skewed
      metrics comparable; the median stops one blown-up metric from carrying a family.
- [x] **2.3** Surface the fragility of the single-metric families (`semantic`, `structure`), where
      selection is vacuous and one degeneracy zeroes the family outright.
- [ ] **2.4 (open decision)** Length-confounded measures are currently flagged but still score.
      Excluding them would be defensible and would gut the syntax family on short corpora — 17 of
      the application corpus's measures are confounded. This is a policy call, not a bug fix.

**Note on the combiner.** The family score is the weighted *median* of its members' percentiles,
which deliberately suppresses a lone measure firing. That is correct for a family score — it
answers "does this body of evidence favour the pair", which needs corroboration — but it means a
single rare measure firing alone is not visible at family level. `pair.familyPeak` records the
strongest single member and which measure produced it, precisely so Phase 3 can read both.

## Phase 4 — An honest null — DONE

Deliberately before Phase 3: conjunction counts are only meaningful once "extreme" is defined
honestly. This is the phase with the least built and the most riding on it.

- [x] **4.1** Expected false alarms: at N responses there are N(N-1)/2 pairs. State how many
      should clear the current threshold by chance alone, *before* showing the ranked list.
- [x] **4.2** Empirical null from the trimmed bulk rather than a parametric assumption. Most pairs
      in a cohort are innocent, so fit the reference distribution on the bulk with the top few
      percent trimmed. Heavy tails in human writing (see Przystalski below) are direct evidence
      that a MAD-based SD reading misleads exactly where it is being used.
- [x] **4.3** Replace SD language with counts anywhere a student might see it. "Four of 124,750
      pairs in this cohort reach this" is defensible. "+5.3 SD" silently assumes a distribution
      that has not been verified.

**What Phase 4 found, and why it makes Phase 3 load-bearing.**

Neither corpus contains any excess convergence. Application answers: 7 pairs clear the default
2.0 threshold where the cohort's own shape predicts 8. Op-eds: 1 clears it where 2.2 are
predicted. The excess is negative at every threshold on both. The default threshold sits below
the noise floor, and the flagged list was previously showing pairs that are exactly what a
cohort of that size throws up on its own.

The important consequence: **percentile-extreme is not the same as improbable.** The op-eds top
pair is at or near the top of every one of the six families, and still scores 2.09 against 2.17
expected. Someone always has to be at the top. The composite score, being an average across
families, dilutes a pair that is extreme on all six toward the middle — so the composite
provably cannot see the conjunction, which is the thing the whole approach rests on.

Phase 3 is therefore not a refinement. It is where the discriminating power has to come from.

The tail model keeps its teeth: on the op-ed cohort a pair at +6 would be about 530x above what
chance produces. It is conservative, not inert.

## Phase 3 — Conjunction instead of composite — DONE

- [x] **3.1** Per-family cohort percentile for each pair, alongside the existing `familyZ`.
- [x] **3.2** Conjunction count: how many families is this pair extreme on, at a stated threshold.
- [x] **3.3** Empirical conjunction rarity: across all pairs in this cohort, how many reach that
      count or higher. **This becomes the ranking scalar.** It is assumption-free about family
      independence, because observed correlation is already baked into the observed count.
- [x] **3.4** Decide the fate of the weighted composite. It may stay as a secondary view, but it
      must not order results — it is what makes "extreme on three families" indistinguishable
      from "extreme on one".

**What was built.** Each family's position is now a rank, not a z-score, and the pair score is
the weighted sum of `-ln(1 - rank)` across families — Fisher's combination, where evidence adds
as probabilities multiply. A pair at the 90th percentile on all six families scores 13.8; a pair
at the 99.9th on one and the 50th on the rest scores 10.4, despite holding the more spectacular
single number. That ordering is the whole point, and the old averaged composite had it backwards.

Because this replaces `adjustedScore` in place, the threshold slider, the Phase 4 null, the
flagged-respondent view and the family weights all keep working unchanged.

**Deliberately not corrected for family correlation.** The families are not independent
(effectively 3.4–3.75 views of six), so a raw conjunction count is inflated. Rather than
discount it in the statistic — which would need a correlation model that is itself an
assumption — the correction happens where Phase 4 put it: the null is built from this cohort's
own pairs, which already contain whatever innocent coupling exists. The report shows the
observed conjunction distribution beside the binomial one *as a diagnostic of coupling*, clearly
labelled as a property of the measures rather than evidence about anybody.

**It still finds nothing.** The top application pair now stands out on 4 of 5 families, which no
other pair matches — and the cohort's own shape predicts 0.66 pairs that far out, so it is about
1.5x chance. The modal says both things: the conjunction is real, and it is what a cohort this
size produces unaided. That second sentence is what stops the banner being an accusation dressed
as arithmetic.

## Phase 9 — Reference texts must not define the class — DONE

Found by a code-review brief written against the `NewHomogenometry/` rewrite (its invariant 1:
"document frequency comes from the baseline only"). `index.html` did not do this. All four
rarity indices were built from every response:

```
buildRareNgramIndex(responses.map(r => r.text))
buildStyleProfileStats(responses.map(r => r.text))
buildContentProfileStats(responses.map(r => r.text))
buildPosTemplateStats(responses.map(r => r.text))
```

`isReference` was used for weight-fitting and display, never for excluding references from the
statistics that define what is ordinary in the class.

The weight is `log((N+1)/(df+1)) + 1`, so each reference sharing a phrase raised that phrase's
document frequency and lowered its weight — and the phrases references share are precisely the
ones a reference was added to detect. The more references seeded, the more the signal was
suppressed. This would have quietly broken the reference-essay test before it ran.

- [x] **9.1** `baselineTexts()` returns the non-reference responses, with a fallback when fewer
      than three remain. The four rarity indices read it. Per-text profiles are built on demand,
      so references are still scored normally against every student — only the question "how
      unusual is this in the class?" is now asked of the class alone.
- [x] **9.2** Changing the reference column used to re-tag and redraw, on the stated grounds that
      flags "only affect how results are grouped and reported, not the pairwise metrics
      themselves". That is no longer true: flags now decide every rarity weight. The handler
      re-runs the analysis when the flag count changes, and says why.

Verified on a synthetic 18-response corpus (14 students, 4 references sharing one distinctive
phrase; no real data). Every word of the shared phrase moved from df=4 to df=0, weight 2.335 to
3.708 — **58.8% of the weight restored** to exactly the evidence that identifies a reference. All
six reference-to-reference pairs rank contiguously above the other 147. With no references
flagged, `baselineTexts()` returns every response, so prior behaviour is reproduced exactly.

Open, by the same logic but needing signature changes rather than a different argument:
`buildBurrowsCorpusStats` and `buildSemanticIndex` also compute a cohort baseline over all texts.
Both need every text present (for vocabulary coverage and for embeddings respectively) while
drawing their baseline from students alone, so neither is a one-line fix.

## Phase 5 — Cohort report before pairwise

- [ ] **5.1** Promote `renderSignalReport()` (index.html:14156) from a panel inside the results to
      a screen shown *before* them, carrying: corpus shape and length distribution, heterogeneity
      flag, assignment openness, per-metric verdicts with reasons, the redundancy map, and the
      expected false-alarm count.

A pairwise score is meaningless without the cohort's regime. "94% similar" says nothing until you
know the cohort median is 92%.

## Phase 6 — Open decisions

- [ ] **6.1** `surface` family: demote out of the evidentiary set entirely? Two responses near the
      class average score high on it while sharing nothing.
- [ ] **6.2** `registerMarkers`: a direct AI-detection heuristic inside a tool defined as
      explicitly not doing AI detection. Retire it, or keep it and accept the inconsistency? The
      rarity-weighted vocabulary metric already does the honest version of the same job.
- [ ] **6.3** Single-metric families: broaden `semantic` and `structure`, or accept and document
      the fragility?

## Phase 7 — Representation

- [ ] **7.1** Lemmatise `distinctiveVocab`. Currently "argues" and "argued" are different words.
      At 60-word responses that is a meaningful fraction of the shared-word evidence. Compromise
      can do this today. Quick win.
- [ ] **7.2** Richer feature vector through the existing Delta machinery: lemma 1-3-grams, POS
      1-3-grams, morphological unigrams, cohort-z-scored and compared pairwise. Reuses
      Burrows/Cosine Delta rather than adding a classifier.
- [ ] **7.3** Dependency bigrams. Highest-value missing feature and currently blocked: needs a
      dependency parser, Compromise has none, spaCy is not available in a single HTML file.
      Parked unless the deployment model changes.

## Phase 8 — Isolation view

The pairwise matrix already contains this: the left tail of the pair distribution is the
suspiciously close pair, the right tail of each response's aggregate distance is the isolated
student.

- [ ] **8.1** Per-response isolation scores (min and mean distance to all others), with per-family
      decomposition for the "in what specific ways" part.
- [ ] **8.2** Equity guardrails. On an admissions corpus this will preferentially flag non-native
      speakers, and that is what the measurement does rather than an artefact that can be
      engineered away. The two tails are not symmetric: improbable convergence has few innocent
      explanations, improbable divergence has many. Keep this in a separate register from the
      pairwise findings, and build nothing that puts it in front of a student.

---

## Suggested order

1 → 2 → 4 → 3 → 5, with 6 and 7.1 taken opportunistically and 8 last.

Cost: phases 1-5 are all O(pairs × metrics) over data already held — cheap even at large N.
Phase 8 needs the full matrix retained, which is the part that actually bites at cohort scale.

---

## Reference: Przystalski et al. 2025

*Stylometry recognizes human and LLM-generated texts in short samples*, arXiv:2507.00838v2.
Read as a methods paper, not a results paper — it is supervised classification, which is a
different problem from unsupervised pairwise similarity. What transfers:

- **Human writing is heavy-tailed in POS n-gram space; machine writing is not.** Their §4.5.1:
  narrow "red bulks" for models against "long grey outlying distributions" for humans. This is
  the evidence behind Phase 4.2 — the null is not well-behaved in the tail, which is the only
  place this tool ever looks.
- **Their human corpus was multi-authorial**, with contributors of varying styles and linguistic
  competency (§5.1) — heterogeneous in the same way a student cohort is, which strengthens the
  analogy.
- **Representation beat curation**: ~3,000 raw normalised frequencies scored MCC 0.87; 196
  curated interpretable features scored 0.72. Roughly 0.15 MCC is what explainability cost them.
  Worth knowing the price, since interpretability is non-negotiable here.
- **No culling.** They kept every feature regardless of document frequency. Under a
  rare-coincidence framing the rarest features are where the signal lives, so pruning
  low-frequency features for speed would destroy the evidence. Aligned with the IDF approach.
- **Cross-domain collapse**: macro-F1 fell to 0.48/0.54 on an external benchmark against 0.81 for
  the top submission and 0.66 for a plain logistic-regression baseline. Never carry fitted
  weights from one assignment to another.
- **Group CV by topic** — the control the reference-essay weight fitting needs, or the fit partly
  learns "this prompt" rather than "this authorship".

What does *not* transfer: every accuracy figure (supervised, labelled), the paraphrase-robustness
result (says a trained detector survives paraphrase, says nothing about whether two paraphrased
texts stay similar to each other), the commercial-detector benchmark, and the model-specific
lexical tells.

Also cited there and worth remembering: Brennan et al. 2012 — deliberate manual obfuscation
reduces stylometric accuracy to roughly chance, achievable by people with no prior knowledge of
stylometry. This tool finds unwitting convergence. It does not find determined evasion, and
should not be described as though it does.
