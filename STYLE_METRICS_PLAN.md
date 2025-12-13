# Stylistic Similarity Metrics - Implementation Plan
## Homogenometry Dashboard Enhancement

**Date**: December 12, 2024
**Purpose**: Extend the Homogenometry Dashboard to detect stylistic/voice convergence in addition to semantic/content convergence

---

## Core Philosophy (Do Not Violate)

### ✅ Must Follow
1. **Corpus-internal comparison only** - All metrics compare pairs within the dataset
2. **No external benchmarks** - Never compare against reference LLM outputs or external standards
3. **Pairwise similarity** - Every metric calculates similarity between Text A and Text B
4. **Model-agnostic** - Works regardless of which AI tool students use (current or future)
5. **Future-proof** - No reliance on model-specific "fingerprints" that change with updates
6. **FERPA compliant** - All processing must remain 100% client-side
7. **Transparent** - Users can click any metric to see calculation details

### ❌ Must Avoid
- Comparing texts against ChatGPT/Claude/Gemini signature patterns
- Using external corpora or reference datasets
- Detecting specific LLM "tells" (e.g., "delve" counters)
- Any server-side processing or API calls
- Opaque black-box classification

---

## Rationale: Why Add Style Metrics?

### The "Beigeness" Phenomenon
Research shows that LLMs given identical prompts produce highly homogeneous outputs. This convergence manifests in two dimensions:

1. **Semantic convergence** (already detected):
   - Similar vocabulary, phrases, meanings
   - Detected by: Cosine, Jaccard, N-grams, Transformer Semantic Similarity

2. **Stylistic convergence** (NEW frontier):
   - Similar sentence structures, complexity, register
   - All students writing with eerily similar "voice"
   - **Not yet detected by current metrics**

### Example Scenario
**Prompt**: "Discuss a challenge in modern education"

**Student A** (used ChatGPT): Essay about technology integration
**Student B** (used ChatGPT): Essay about mental health

- **Current semantic similarity**: LOW (different topics)
- **Stylistic similarity**: VERY HIGH (same sentence structures, complexity, register)
- **Current tool**: Misses this ❌
- **Enhanced tool**: Detects this ✅

### Key Insight
If 15 students independently wrote essays, their **styles would naturally vary**. If they all used the same LLM, their **styles would converge** - even with different content. This convergence is **statistically improbable** and signals potential AI use.

---

## Proposed New Metrics

### Category 1: Syntactic Pattern Similarity

#### Metric 1A: POS Tag Sequence Similarity
**What it measures**: How similarly two texts structure their sentences grammatically

**Method**:
1. Use NLP tool (SpaCy.js or Transformers.js) to tag each word with part-of-speech
2. Extract POS n-grams (sequences of 3-5 POS tags)
   - Example: "The quick brown fox" → `[DET, ADJ, ADJ, NOUN]`
3. Calculate Jaccard similarity of POS n-gram sets between two texts
4. Higher score = more similar grammatical structures

**Example detection**:
- Text A: "The implementation represents..." → `DET-NOUN-VERB`
- Text B: "The adoption constitutes..." → `DET-NOUN-VERB`
- Both use same syntactic templates despite different words

**Implementation complexity**: Medium (requires POS tagger)

---

#### Metric 1B: Dependency Pattern Similarity
**What it measures**: How similarly two texts build grammatical relationships

**Method**:
1. Parse dependency trees for sentences in both texts
2. Extract common dependency patterns (e.g., nsubj-verb-dobj)
3. Compare pattern frequency distributions
4. Calculate cosine similarity of pattern vectors

**Example detection**:
- Both texts favor passive constructions at similar rates
- Both use same subject-verb-object arrangements

**Implementation complexity**: High (requires dependency parser)

---

#### Metric 1C: Clause Complexity Similarity
**What it measures**: How similarly two texts use simple vs. complex sentences

**Method**:
1. Count clauses per sentence in each text
2. Calculate distribution: % simple (1 clause), compound (2), complex (3+)
3. Compare distributions using KL divergence or cosine similarity
4. Normalize to 0-1 similarity score

**Example detection**:
- Both texts: 20% simple, 35% compound, 45% complex sentences
- Suspiciously identical complexity profiles

**Implementation complexity**: Medium (requires clause detection)

---

### Category 2: Lexical Style Similarity

#### Metric 2A: Function Word Distribution Similarity
**What it measures**: How similarly two texts use "glue words" (the, of, however, etc.)

**Method**:
1. Define function word list (determiners, prepositions, conjunctions, pronouns)
2. Calculate frequency of each function word in both texts
3. Create frequency vectors for both texts
4. Calculate cosine similarity between vectors

**Why it matters**: Function words reveal unconscious stylistic habits. LLMs have characteristic function word patterns that persist across different content.

**Example detection**:
- Text A: "the"=5.2%, "of"=3.1%, "however"=0.8%
- Text B: "the"=5.1%, "of"=3.2%, "however"=0.9%
- Eerily similar usage rates = potential LLM convergence

**Implementation complexity**: Low (simple word counting)

---

#### Metric 2B: Lexical Sophistication Similarity
**What it measures**: How similarly sophisticated the vocabulary is

**Method**:
1. Calculate metrics for both texts:
   - Type-token ratio (unique words / total words)
   - Average word length in characters
   - Average word frequency rank (using frequency list)
   - Hapax legomena rate (words used only once)
2. Create feature vector for each text
3. Calculate Euclidean distance, normalize to 0-1 similarity

**Example detection**:
- Text A: TTR=0.621, Avg length=5.3, Avg freq rank=8234
- Text B: TTR=0.618, Avg length=5.4, Avg freq rank=8156
- Suspiciously aligned sophistication levels

**Implementation complexity**: Low-Medium (word counting + frequency list)

---

#### Metric 2C: Vocabulary Diversity Similarity
**What it measures**: How similarly "rich" or "repetitive" the vocabulary is

**Method**:
1. Calculate vocabulary diversity metrics:
   - Simpson's Diversity Index
   - Shannon Entropy
   - Moving-average type-token ratio (MATTR)
2. Compare metrics between texts
3. Calculate similarity score

**Why it matters**: LLMs tend to produce consistent vocabulary diversity. Natural human writing varies more.

**Implementation complexity**: Low-Medium

---

### Category 3: Readability & Complexity Similarity

#### Metric 3A: Readability Score Convergence
**What it measures**: How suspiciously similar the reading difficulty levels are

**Method**:
1. Calculate multiple readability scores for both texts:
   - Flesch-Kincaid Grade Level
   - Flesch Reading Ease
   - Gunning Fog Index
   - SMOG Index
2. Create readability feature vector for each text
3. Calculate Euclidean distance between vectors
4. Normalize to 0-1 similarity score

**Why it matters**: Natural writers vary in complexity. LLMs consistently hit the same "sweet spot" (typically 10th-12th grade level).

**Example detection**:
- 15 students all producing texts at exactly 11.8-12.2 grade level
- Statistically improbable if written independently

**Implementation complexity**: Low (formula-based calculation)

---

#### Metric 3B: Sentence Length Pattern Similarity
**What it measures**: How similarly two texts vary their sentence lengths

**Method**:
1. Extract sentence lengths for both texts
2. Calculate distribution statistics:
   - Mean, median, standard deviation
   - Quartiles (25th, 50th, 75th percentile)
   - Coefficient of variation
3. Compare distributions using statistical similarity measures
4. Normalize to 0-1 score

**Why it matters**: Sentence rhythm is a stylistic fingerprint. LLMs have characteristic patterns.

**Example detection**:
- Text A: Mean=19.2, SD=6.3, CoV=0.328
- Text B: Mean=19.1, SD=6.4, CoV=0.335
- Nearly identical rhythm patterns

**Implementation complexity**: Low (statistical calculation)

---

#### Metric 3C: Paragraph Structure Similarity
**What it measures**: How similarly two texts organize content into paragraphs

**Method**:
1. Calculate paragraph-level metrics:
   - Average paragraph length (words/sentences)
   - Paragraph length variance
   - Paragraph count relative to text length
2. Create structure feature vector
3. Calculate cosine similarity

**Implementation complexity**: Low

---

### Category 4: Register & Formality Similarity

#### Metric 4A: Formality Level Similarity
**What it measures**: How similar the "academic voice" formality is

**Method**:
1. Calculate formality indicators:
   - Personal pronoun rate (I, we, you)
   - Contraction rate (don't, can't)
   - Nominalization rate (convert verbs to nouns)
   - Passive voice rate
   - Hedging word rate (perhaps, might, possibly)
2. Create formality feature vector
3. Calculate cosine similarity

**Why it matters**: LLMs maintain consistent register. Natural writers vary more in formality.

**Implementation complexity**: Low-Medium (pattern matching + POS tagging)

---

#### Metric 4B: Discourse Marker Similarity
**What it measures**: How similarly two texts use transitional/organizational phrases

**Method**:
1. Count discourse markers in both texts:
   - Additive (furthermore, moreover, additionally)
   - Contrastive (however, nevertheless, nonetheless)
   - Causal (therefore, thus, consequently)
   - Sequential (firstly, subsequently, finally)
2. Calculate frequency distributions
3. Compare using cosine similarity

**Why it matters**: LLMs overuse certain discourse markers. Convergence in marker usage signals potential LLM use.

**Implementation complexity**: Low (pattern matching)

---

### Category 5: Punctuation & Formatting Similarity

#### Metric 5A: Punctuation Pattern Similarity
**What it measures**: How similarly two texts use punctuation marks

**Method**:
1. Count punctuation marks in both texts:
   - Comma density
   - Semicolon usage
   - Em-dash usage
   - Question/exclamation mark rates
   - Colon usage
2. Create punctuation feature vector
3. Calculate cosine similarity

**Why it matters**: Punctuation habits are stylistic fingerprints. LLMs have characteristic patterns (e.g., ChatGPT often uses em-dashes).

**Implementation complexity**: Low (character counting)

---

## Implementation Roadmap

### Phase 1: Foundation (Low-Hanging Fruit)
**Implement metrics requiring no external NLP libraries**

**Metrics to implement**:
1. Function Word Distribution Similarity (2A)
2. Readability Score Convergence (3A)
3. Sentence Length Pattern Similarity (3B)
4. Paragraph Structure Similarity (3C)
5. Punctuation Pattern Similarity (5A)
6. Lexical Sophistication Similarity (2B) - partial (TTR, word length)

**Technical requirements**:
- Function word list (static array)
- Readability formulas (already have sentence/word counting)
- Statistical calculations (mean, SD, cosine similarity)

**Estimated complexity**: Low
**Implementation time**: 1-2 weeks
**Value**: High (immediate improvement in detection)

---

### Phase 2: NLP Integration (Client-Side)
**Add client-side NLP library for linguistic analysis**

**Option A: Compromise.js** (Lightweight, JavaScript-native)
- Pros: Fast, no WASM, good POS tagging
- Cons: Less accurate than SpaCy
- Size: ~200KB

**Option B: Transformers.js with BERT-based POS tagger**
- Pros: More accurate, already using Transformers.js
- Cons: Larger model download
- Size: ~50-100MB model

**Recommended**: Start with Compromise.js, upgrade to Transformers.js if needed

**Metrics to implement**:
1. POS Tag Sequence Similarity (1A)
2. Clause Complexity Similarity (1C) - basic version
3. Formality Level Similarity (4A) - enhanced with POS
4. Discourse Marker Similarity (4B)

**Estimated complexity**: Medium
**Implementation time**: 2-3 weeks

---

### Phase 3: Advanced Syntactic Analysis
**Add dependency parsing for deep structural analysis**

**Metrics to implement**:
1. Dependency Pattern Similarity (1B)
2. Vocabulary Diversity Similarity (2C)
3. Complete formality analysis (4A)

**Technical requirements**:
- Dependency parser (Transformers.js with syntax model)
- More sophisticated linguistic features

**Estimated complexity**: High
**Implementation time**: 3-4 weeks
**Priority**: Lower (Phase 1 and 2 provide most value)

---

## Technical Architecture

### Integration with Existing System

#### 1. Metric Calculation Layer
**Location**: Inline JavaScript functions (like existing metrics)

```javascript
function functionWordDistributionSimilarity(text1, text2) {
    const functionWords = ['the', 'of', 'and', 'to', 'a', 'in', 'that', 'it', ...];

    // Calculate frequency vectors
    const freq1 = calculateFunctionWordFrequencies(text1, functionWords);
    const freq2 = calculateFunctionWordFrequencies(text2, functionWords);

    // Cosine similarity
    return cosineSimilarity(freq1, freq2);
}
```

#### 2. Web Worker Integration
**Challenge**: New metrics must work in Web Workers for parallel processing

**Solution**:
- Phase 1 metrics: Simple calculations, work in workers ✅
- Phase 2+ metrics: Require NLP library
  - Option A: Include library in worker code
  - Option B: Calculate in main thread (like transformer semantic)

**Recommended**: Use Option B initially (main thread calculation after workers complete)

#### 3. UI Integration

**Metric Settings Panel**:
```html
<!-- New section: Stylistic Metrics -->
<div>
    <h4>Stylistic Similarity Metrics</h4>

    <div class="metric-item">
        <input type="checkbox" id="metric_functionWordDist" checked>
        <label>Function Word Distribution</label>
        <input type="range" id="weight_functionWordDist" min="0.1" max="5.0" step="0.1" value="2.0">
    </div>

    <!-- ... other style metrics ... -->
</div>
```

**Calculation Details Modal**:
Each metric needs a `generateXXXDetails()` function showing:
- How the metric was calculated
- Actual values from both texts
- Examples of what was detected
- Final similarity score

#### 4. Normalization & Weighting
Style metrics integrate into existing system:
- Added to `getSelectedMetrics()`
- Included in `calculateAllMetrics()`
- Normalized with `normalizeMetric()`
- Weighted in `calculateCombinedSimilarity()`

---

## Testing Strategy

### 1. Unit Tests
Test each metric function independently:
```javascript
// Test function word similarity
const text1 = "The cat sat on the mat. The dog ran in the park.";
const text2 = "The bird flew over the tree. The fish swam in the pond.";
const similarity = functionWordDistributionSimilarity(text1, text2);
// Should be HIGH (both use "the" and "in" frequently)
```

### 2. Integration Tests
Test with real student data:
- **Control group**: Human-written essays (expect varied styles)
- **Test group**: Known AI-generated essays (expect convergent styles)

### 3. Validation Approach
Since we're corpus-internal:
1. Analyze corpus of known human-written texts → measure baseline style variance
2. Analyze corpus with planted AI texts → measure style convergence
3. Confirm: AI-heavy corpus shows significantly higher pairwise style similarity

---

## Data Structures & Performance

### Memory Considerations
New metrics require storing additional data per text:

```javascript
// Extend response object
{
    id: 1,
    text: "...",
    // Existing cached data
    tokens: [...],
    sentences: [...],
    // NEW: Cached linguistic features
    posSequences: [...],
    functionWordFreq: {...},
    readabilityScores: {...},
    syntacticPatterns: [...]
}
```

**Strategy**: Calculate once, cache for all pairwise comparisons

### Performance Optimization

**Current bottleneck**: N² pairwise comparisons
- 50 texts = 1,225 pairs
- 100 texts = 4,950 pairs

**With new metrics**:
- Phase 1: Minimal impact (simple calculations)
- Phase 2+: Pre-compute linguistic features once per text
- Use Web Workers for parallelization (where possible)

**Expected performance**:
- Phase 1 metrics: <5% slowdown
- Phase 2 metrics: 10-20% slowdown (NLP processing)
- Still completes in <30 seconds for 100 texts

---

## User Experience

### 1. Progressive Disclosure
Don't overwhelm users with 25+ metrics:

**Default configuration**:
- Show "Basic Metrics" (current 18)
- Show "Style Metrics" (new 8-10) collapsed by default
- Allow users to expand and enable as needed

### 2. Metric Grouping
Organize by category:
- 📝 Lexical Similarity (Cosine, Jaccard, etc.)
- 🔤 Character Patterns (N-grams)
- 📊 Statistical Features (Sentence variance, etc.)
- ✨ Semantic AI (Transformer)
- **🎨 Stylistic Patterns (NEW)**
  - Function words
  - Readability
  - Formality
  - Syntax

### 3. Educational Tooltips
Help users understand what they're looking at:
```
"Function Word Distribution Similarity"
ℹ️ Measures how similarly two texts use common words like
'the', 'of', 'however'. High similarity in function word
usage suggests similar writing style or voice.
```

### 4. Detailed Explanations
Click any metric badge → show calculation with examples from actual texts:
```
Function Word Distribution Similarity

Text A uses:
- "the": 42 times (5.2%)
- "of": 25 times (3.1%)
- "however": 6 times (0.7%)

Text B uses:
- "the": 38 times (5.1%)
- "of": 24 times (3.2%)
- "however": 7 times (0.9%)

Similarity: 94.3% (Very High)

These texts use function words at nearly identical rates,
suggesting similar stylistic patterns or convergence.
```

---

## Documentation Updates

### Update About/Methodology Modal
Add new section:

```markdown
### Stylistic Similarity Metrics (8 metrics)

Beyond content similarity, these metrics detect convergence
in writing style, voice, and sophistication level.

**Why this matters**: When multiple students use the same AI
tool, their responses converge not just in *what* they say,
but *how* they say it. These metrics detect unusual stylistic
homogeneity within your corpus.

1. **Function Word Distribution** - Measures similarity in
   usage rates of common grammatical words...

[... detailed explanations for each metric ...]
```

### Create Tutorial/Help Section
Guide users on interpreting style metrics:
- What does high stylistic similarity mean?
- How is it different from content similarity?
- When should I investigate?
- Example scenarios

---

## Ethical Considerations

### 1. False Positives
Style convergence can occur naturally:
- Students taught by same instructor may develop similar academic voice
- Assignment constraints force similar structures
- Skill-matched students may write at similar complexity

**Mitigation**:
- Tool flags for investigation, not proof of misconduct
- Emphasize: High similarity = warrants review, not automatic guilt
- Provide context in explanations

### 2. Transparency
Every metric must be explainable:
- Show calculation steps
- Display actual text features being compared
- No black-box scores

### 3. Privacy (FERPA)
All processing remains client-side:
- No text sent to servers
- No external API calls
- Models cached in browser

---

## Success Metrics

### How to measure if new metrics add value:

1. **Detection improvement**:
   - Test on corpus with known AI usage
   - Measure: Do style metrics catch cases semantic metrics miss?

2. **Discriminative power**:
   - Human-written corpus: Style variance should be HIGH
   - AI-generated corpus: Style variance should be LOW
   - Tool should distinguish these

3. **User feedback**:
   - Do instructors find style metrics useful?
   - Do they provide actionable insights?

---

## Future Enhancements (Beyond Initial Implementation)

### 1. Cluster Analysis
Identify groups with similar styles:
- "These 8 students all write with identical voice"
- Visual clustering in 2D space (PCA/t-SNE on style features)

### 2. Style Profile Visualization
Per-student style fingerprint:
- Radar chart showing formality, complexity, diversity, etc.
- Compare individual to corpus average
- Identify outliers

### 3. Temporal Analysis
Track style consistency across assignments:
- Student's style suddenly changes → potential AI adoption
- Class average style converges over semester → systemic issue

### 4. Cross-Assignment Comparison
Compare style variance across different prompts:
- Open-ended prompt: High style variance (expected)
- Constrained prompt: Low variance (expected)
- Multiple assignments: Abnormally consistent variance (suspicious)

---

## Open Questions

### 1. Weighting Strategy
How should style metrics be weighted relative to content metrics?
- Equal weight?
- Lower weight (style is noisier)?
- User-configurable presets ("Content-focused" vs "Style-focused")?

### 2. Normalization Approach
Style metrics may have different distributions than content metrics:
- Need separate normalization?
- Log-transform before normalizing?
- Test with real data

### 3. Computational Budget
How many style metrics before it's too slow?
- Target: Keep analysis under 60 seconds for 100 texts
- May need to make some metrics optional/advanced

### 4. Library Choice
Which NLP library balances accuracy vs. size?
- Compromise.js: Lightweight but less accurate
- Transformers.js: Accurate but larger downloads
- Hybrid approach?

---

## Implementation Checklist

### Phase 1: Foundation Metrics
- [ ] Define function word list (150-200 common words)
- [ ] Implement function word distribution similarity
- [ ] Implement readability score convergence (Flesch-Kincaid, Gunning Fog)
- [ ] Implement sentence length pattern similarity
- [ ] Implement paragraph structure similarity
- [ ] Implement punctuation pattern similarity
- [ ] Implement basic lexical sophistication (TTR, word length)
- [ ] Add all to Web Worker metric calculation switch
- [ ] Create detail modal functions for each metric
- [ ] Add to Metric Settings UI
- [ ] Add to About/Methodology documentation
- [ ] Test with sample datasets
- [ ] Validate performance (< 60s for 100 texts)

### Phase 2: NLP Integration
- [ ] Evaluate Compromise.js vs Transformers.js
- [ ] Integrate chosen library (client-side only)
- [ ] Implement POS tag sequence similarity
- [ ] Implement clause complexity similarity
- [ ] Enhance formality level similarity with POS data
- [ ] Implement discourse marker similarity
- [ ] Update calculation flow (main thread vs worker)
- [ ] Create detail modals with linguistic examples
- [ ] Test accuracy on known datasets
- [ ] Performance optimization

### Phase 3: Advanced Features
- [ ] Dependency pattern similarity
- [ ] Vocabulary diversity similarity
- [ ] Complete formality analysis
- [ ] Advanced normalization tuning
- [ ] Cluster visualization (optional)
- [ ] Style profile dashboard (optional)

---

## References & Resources

### Academic Research
- **Mode convergence in LLMs**: Research papers on "beigeness" phenomenon
- **Stylometry**: Authorship attribution literature
- **Readability formulas**: Flesch-Kincaid, Gunning Fog, SMOG validation studies

### Technical Libraries
- **Compromise.js**: https://github.com/spencermountain/compromise
  - Client-side NLP, POS tagging, lightweight
- **Transformers.js**: https://github.com/xenova/transformers.js
  - Already integrated, could extend for syntax models
- **Readability.js**: Existing readability formula implementations

### Testing Datasets
- **Human-written student essays**: Establish baseline variance
- **AI-generated essays**: Test detection capability
- **Mixed corpus**: Validate discriminative power

---

## Notes & Decisions Log

**2024-12-12**: Initial plan created
- Philosophy: Corpus-internal only, no external benchmarks
- Goal: Detect stylistic convergence as proxy for AI use
- Approach: Pairwise style similarity metrics
- Priority: Phase 1 (low-hanging fruit) first

---

## Conclusion

This plan extends the Homogenometry Dashboard to detect the **stylistic dimension of LLM beigeness** while maintaining its core philosophy:

✅ Corpus-internal comparison
✅ Model-agnostic detection
✅ Future-proof approach
✅ Transparent calculations
✅ FERPA-compliant processing

By measuring how similarly students **write** (not just what they write about), the tool becomes significantly more powerful at detecting the homogeneity that emerges when multiple students use the same AI tool.

The phased approach ensures we can deliver value incrementally while managing complexity.
