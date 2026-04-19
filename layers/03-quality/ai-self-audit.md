---
name: ai-self-audit
description: >
  Document-level AI writing pattern detection. Scans the entire thesis for
  macro-level patterns that per-sentence checks (paraphraser Phase 9) would miss.
  Complementary to Phase 9: micro vs macro analysis.
---

# Purpose

Detect AI-generated writing patterns at the DOCUMENT level. The paraphraser's Phase 9 (AI-Tell Audit) works per-sentence during paraphrasing. This skill works across the entire document during final QA — catching patterns that only emerge when you look at pages and chapters as a whole.

**Phase 9 (paraphraser)** = micro: "Is this sentence AI-like?"
**AI Self-Audit (this skill)** = macro: "Does this document read like it was AI-generated?"

# Input

- All BAB `.tex` files
- Optional: `style-profile.json` from style calibration (for baseline comparison)

# Output

- `ai-audit-report.json` (see `shared/schemas/ai-audit-report.json`)

# Relationship to Paraphraser Phase 9

| Aspect | Phase 9 (Paraphraser) | AI Self-Audit (This Skill) |
|--------|----------------------|---------------------------|
| Scope | Per-sentence, per-paragraph | Per-BAB, per-document |
| When | During paraphrasing (Stage 2) | During final QA (Stage 4.5) |
| Detects | Individual AI vocabulary, copula avoidance, hedging | Uniform sentence length distribution, low burstiness, cross-BAB style inconsistency |
| Patterns | A1-A12, B1-B4 (16 patterns) | S1-S4, V1-V4, C1-C3, B1-B3, I1-I5 (19 patterns) |
| Action | Fix during writing | Flag for review before submission |

---

# Phase 1: STRUCTURAL ANALYSIS

Measure the statistical shape of the writing. Human writing is irregular; AI writing is uniform.

## S1: Sentence Length Distribution

- Compute mean and standard deviation of sentence length (word count) per BAB
- **AI signal**: Low standard deviation (< 5 words). Human writing varies between 8-word punchy sentences and 30-word complex ones. AI tends toward 15-20 words consistently.
- **Threshold**: std_dev < 5.0 → HIGH risk, 5.0-8.0 → MEDIUM, > 8.0 → LOW

## S2: Paragraph Length Distribution

- Count sentences per paragraph across the document
- **AI signal**: Uniform paragraph length (every paragraph has 4-5 sentences). Human writing has 2-sentence paragraphs mixed with 8-sentence ones.
- **Threshold**: paragraph length std_dev < 1.0 → HIGH risk, 1.0-2.0 → MEDIUM, > 2.0 → LOW

## S3: Section Opening Patterns

- Extract the first sentence of every `\section{}` and `\subsection{}`
- **AI signal**: Repetitive opening structure. If 60%+ of sections start with the same syntactic pattern (e.g., "Pada bagian ini...", "Berikut ini..."), flag it.
- **Threshold**: same-pattern ratio > 0.6 → HIGH, 0.4-0.6 → MEDIUM, < 0.4 → LOW

## S4: Paragraph Opening Diversity

- Extract the first word/phrase of every paragraph
- **AI signal**: Low diversity in paragraph openers. If the same opener appears 5+ times (e.g., "Selain itu," starts 8 paragraphs), flag it.
- **Threshold**: any single opener used > 5 times → WARNING per instance

---

# Phase 2: VOCABULARY ANALYSIS

Measure lexical diversity and AI-specific word patterns across the full document.

## V1: Type-Token Ratio (TTR)

- Compute TTR per BAB: unique words / total words
- **AI signal**: Low TTR indicates repetitive vocabulary. AI text typically has TTR 0.3-0.4 for academic text. Human academic text: 0.4-0.6.
- **Threshold**: TTR < 0.35 → HIGH risk, 0.35-0.45 → MEDIUM, > 0.45 → LOW
- **Note**: TTR naturally decreases with text length. Compare per-BAB, not across entire document.

## V2: AI Vocabulary Frequency (Document-Level)

- Count occurrences of known AI vocabulary across ALL BABs (aggregate of Phase 9 patterns A1):
  - English: "delve", "landscape", "tapestry", "foster", "multifaceted", "pivotal", "crucial", "robust", "comprehensive", "innovative", "cutting-edge", "paradigm"
  - Indonesian: "sangat krusial", "sangat penting untuk", "memainkan peran penting", "di era modern ini", "dalam era digital"
- **Threshold**: > 5 instances total → WARNING, > 10 → HIGH

## V3: Connector Repetition

- Count frequency of each academic connector across the document
- **AI signal**: One connector dominates. "Selain itu" appears 15 times while "Lebih lanjut" appears 0 times.
- **Threshold**: any single connector used > 10 times → WARNING, > 15 → HIGH
- **Common connectors to track**: "Selain itu", "Lebih lanjut", "Oleh karena itu", "Dengan demikian", "Namun", "Akan tetapi", "Di samping itu", "Berdasarkan"

## V4: Hedging Density

- Count hedging words per 1000 words per BAB
- **Hedging words (ID)**: "dapat", "mungkin", "cenderung", "dimungkinkan", "diharapkan", "tampaknya", "kemungkinan"
- **Hedging words (EN)**: "may", "might", "could", "potentially", "possibly", "arguably"
- **AI signal**: Hedging density > 15 per 1000 words (AI over-hedges). Academic norm: 5-10 per 1000 words.
- **Threshold**: > 15/1000 → HIGH, 10-15/1000 → MEDIUM, < 10/1000 → LOW

---

# Phase 3: CROSS-BAB STYLE CONSISTENCY

Human writers have a consistent personal style. AI-generated text per-BAB may have different "voices" if generated in separate sessions.

## C1: Style Shift Detection

- Compare per-BAB metrics: avg sentence length, TTR, passive voice ratio, hedging density
- **AI signal**: Dramatic shift between BABs. If BAB 2 has avg sentence length 22 and BAB 3 has avg 14, something changed.
- **Threshold**: > 30% deviation from document mean for any metric → WARNING

## C2: Formality Level Consistency

- Estimate formality per BAB based on: passive voice ratio, nominalization density, average word length
- **AI signal**: One BAB is noticeably more/less formal than others
- **Threshold**: formality score deviation > 25% from mean → WARNING

## C3: Vocabulary Overlap Between BABs

- Compute vocabulary overlap (Jaccard similarity) between consecutive BABs
- **AI signal**: Very low overlap (< 0.15) suggests different "authors" or generation sessions. Very high overlap (> 0.6) suggests copy-paste.
- **Threshold**: Jaccard < 0.15 or > 0.6 → WARNING

---

# Phase 4: BURSTINESS ANALYSIS

Human writing alternates between high and low complexity within paragraphs. AI writing maintains uniform complexity.

## B1: Intra-Paragraph Complexity Variance

- For each paragraph, compute the variance of sentence complexity (measured by: word count + clause count)
- **AI signal**: Near-zero variance within paragraphs. Every sentence in a paragraph has similar complexity.
- **Threshold**: mean intra-paragraph variance < 2.0 → HIGH risk, 2.0-5.0 → MEDIUM, > 5.0 → LOW

## B2: Complexity Alternation Pattern

- Measure whether short and long sentences alternate or cluster
- **AI signal**: No alternation — all sentences in a paragraph are medium-length. Human writing has short-long-short patterns.
- **Method**: Compute autocorrelation of sentence length within each BAB. Low autocorrelation = good (varied). High autocorrelation = bad (uniform).

## B3: Paragraph Complexity Progression

- Track average sentence complexity across paragraphs within each section
- **AI signal**: Flat progression. Human writing tends to start simple (topic sentence), build complexity (evidence/analysis), then simplify (conclusion). AI maintains constant complexity.
- **Threshold**: flat progression in > 70% of sections → WARNING

---

# Phase 5: INDONESIAN-SPECIFIC PATTERNS

Patterns specific to AI-generated Bahasa Indonesia academic text.

## I1: "Dalam era..." / "Di era..." Opener Frequency

- Count instances of era-based openers across the document
- **Threshold**: > 2 instances → WARNING, > 4 → HIGH

## I2: "Sangat penting" / "Sangat krusial" Inflation

- Count significance inflation phrases
- **Threshold**: > 3 instances → WARNING, > 6 → HIGH

## I3: "Berbagai" Filler

- Count "berbagai" used as vague quantifier without specifics following
- **Threshold**: > 5 instances → WARNING

## I4: Connector Stacking

- Detect instances where 2+ connectors appear in sequence within 2 sentences
- Example: "Selain itu, lebih lanjut, di samping itu..."
- **Threshold**: any instance → WARNING

## I5: Passive Voice Ratio

- Compute passive voice percentage per BAB
- **Indonesian academic norm**: 40-60% passive is natural
- **AI signal**: > 70% passive (AI over-uses passive in Indonesian academic text)
- **Threshold**: > 70% → HIGH, 60-70% → MEDIUM, 40-60% → LOW (normal), < 40% → INFO (unusually active)

---

# Phase 6: RISK SCORING

## Per-BAB Risk Score

For each BAB, aggregate all findings:

```
HIGH_count = count of HIGH severity findings in this BAB
MEDIUM_count = count of MEDIUM severity findings
WARNING_count = count of WARNING findings

IF HIGH_count >= 3 → BAB risk: HIGH
ELSE IF HIGH_count >= 1 OR MEDIUM_count >= 3 → BAB risk: MEDIUM
ELSE → BAB risk: LOW
```

## Overall Document Risk Score

```
IF any BAB is HIGH → overall: HIGH_RISK
ELSE IF 2+ BABs are MEDIUM → overall: MEDIUM_RISK
ELSE IF any BAB is MEDIUM → overall: LOW_RISK
ELSE → overall: CLEAN
```

## Remediation Suggestions

For each finding, provide actionable fix:

| Finding | Suggestion |
|---------|-----------|
| Low sentence length std_dev | Vary sentence length: add some short (8-12 word) sentences between longer ones |
| Uniform paragraph length | Break some paragraphs into 2-3 sentences, extend others to 6-7 |
| Repetitive section openers | Rewrite opening sentences to use different syntactic structures |
| Low TTR | Replace repeated words with contextually appropriate alternatives (but maintain technical term consistency) |
| High hedging density | Remove redundant hedges. One hedge per claim is enough. |
| High passive ratio | Convert some passive sentences to active. Name the actor. |
| Connector repetition | Use the synonym bank from the paraphraser's Phase 7 for connector alternatives |

---

# Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| HIGH | Likely detectable by AI detection tools (Turnitin, GPTZero) | Must fix before submission |
| MEDIUM | Borderline — may or may not trigger detection | Should fix |
| LOW | Natural writing patterns | No action needed |
| INFO | Observation only | For awareness |

# Verdict

```
IF overall risk = HIGH_RISK → verdict: HIGH_RISK (do not submit without fixes)
ELSE IF overall risk = MEDIUM_RISK → verdict: MEDIUM_RISK (review flagged sections)
ELSE IF overall risk = LOW_RISK → verdict: LOW_RISK (minor improvements possible)
ELSE → verdict: CLEAN
```

**This skill does NOT block the pipeline.** It produces an advisory report. The decision to fix or proceed is the student's. However, the Submission Readiness Gate (if implemented) should require at least LOW_RISK or CLEAN.

# Integration Points

- **Pipeline**: Runs at Stage 4.5 (Final Integrity) alongside `integrity-checker.md`. Both produce separate reports.
- **Paraphraser Phase 9**: Complementary. Phase 9 catches micro-level tells during writing. This catches macro-level patterns during QA.
- **Style Calibration**: If a `style-profile.json` exists, use it as baseline — deviations from the student's natural style are more meaningful than absolute thresholds.
- **Submission Readiness**: AI audit verdict feeds into the readiness gate.
- **Disclosure Generator**: If HIGH_RISK sections are found, the disclosure statement should be more detailed about AI usage in those sections.
