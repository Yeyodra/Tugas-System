---
name: paraphraser
description: >
  Data-driven academic paraphrasing engine for PI. Uses taxonomy from
  peer-reviewed NLP research (ParaSCI, THESAI 2018) with 6 paraphrase
  phenomena, 3 conditional categories, and scored few-shot examples
  from scientific corpora. Not a simple synonym swapper.
---

# Purpose

Expert academic paraphrasing engine that applies a **taxonomy of 6 paraphrase phenomena** extracted from peer-reviewed NLP research, combined with **conditional constraints** for academic writing. Replaces naive synonym replacement with structured, scoreable transformations.

# Input

- `text`: Original prose to be paraphrased (Indonesian or English)
- `chapter_context`: BAB number for context-aware terminology
- `mode`: `single` (one sentence/paragraph with scoring) or `bulk` (entire section, clean output)
- `style_profile`: Optional Style Profile from `layers/01-knowledge/style-calibration.md`

# Output

- Paraphrased text with formal academic register
- Phenomena scoring (single mode) or clean text (bulk mode)
- Meaning preservation verification

---

# PHASE 0: Input Analysis

Before paraphrasing, ALWAYS:

1. **Identify the text type**: scientific claim, definition, methodology description, result statement, or general academic prose
2. **Identify protected elements** (NEVER paraphrase these):
   - Technical terms and domain-specific jargon
   - Proper nouns, software names, algorithm names
   - Mathematical expressions and formulas
   - Exact numerical values and statistics
   - Acronyms/abbreviations and their definitions
   - Direct quotations marked with quotation marks
   - Citations in (Author, Year) format
3. **Identify the target language**: Indonesian academic (default for PI) or English academic
4. **Assess the sentence complexity**: simple, compound, or complex — determines which techniques to apply

---

# PHASE 1: Paraphrase Phenomena Taxonomy

Source: ParaSCI (Dong et al., 2021, EACL) — empirical frequency analysis of 100 scientific paraphrase pairs.

## The 6 Phenomena (MANDATORY — apply at least 3 per sentence)

| # | Phenomenon | Description | Avg Frequency | Priority |
|---|-----------|-------------|---------------|----------|
| 1 | **Synonym (SYN)** | Substitute a word with its contextually appropriate synonym | 1.04 | HIGH |
| 2 | **Structure (STR)** | Use different sentence structures to express the same thing | 0.45 | HIGH |
| 3 | **Definition (DEF)** | Substitute a word with its definition, or vice versa | 0.68 | MEDIUM |
| 4 | **Break (BRK)** | Break a long sentence into smaller ones, or combine short ones | 0.32 | MEDIUM |
| 5 | **Word-Form (WFM)** | Change morphological form (noun→verb, adj→adverb) | 0.15 | LOW |
| 6 | **Voice (VOC)** | Change active↔passive voice | 0.14 | LOW |

## Application Rules

- **MINIMUM**: Apply at least 3 different phenomena per sentence
- **MAXIMUM**: Don't over-transform — meaning must remain identical
- **PRIORITY ORDER**: STR > SYN > DEF > BRK > WFM > VOC
- **NEVER** apply only SYN alone — that's the hallmark of lazy paraphrasing and plagiarism detectors catch it easily

---

# PHASE 2: Conditional Paraphrasing Constraints

Source: Al-Ghidani & Fahmy (2018), THESAI — Conditional Text Paraphrasing Taxonomy.

## A. Morphology-Based Constraints
- **Tense Preservation**: Academic text uses present tense for established facts, past tense for specific studies. PRESERVE the original tense intent.
- **Word-Form Shifts**: noun↔verb, adj↔noun, adverb↔adj transformations
  - `implementation → implement` (noun→verb)
  - `efficient → efficiency` (adj→noun)
  - `significantly → significant` (adverb→adj)

## B. Syntax-Based Constraints
- **Clause Reordering**: Move subordinate clauses to main position and vice versa
- **Coordination↔Subordination**: Convert "A and B" → "A, which also B" or reverse
- **Fronting**: Move important info to sentence-initial position
- **Cleft Sentences**: "X does Y" → "It is X that does Y" (for emphasis)
- **Parse Tree Divergence**: The syntactic parse tree of the output MUST differ from input

## C. Readability-Based Constraints
- **Academic Register**: Output must maintain formal academic register
- **Complexity Matching**: Output complexity should match input
- **Conciseness**: Remove wordiness while preserving all information
- **Nominalization Level**: Match the nominalization density of the field

---

# PHASE 3: Technique Execution Patterns

## Pattern 1: Clause Restructuring (STR)

```
INPUT:  "Critical thinking is essential for developing analytical skills in higher education."
OUTPUT: "Analytical skills in university settings depend significantly on the development of critical thinking abilities."

APPLIED: STR (clause reorder) + SYN (essential→depend significantly) + WFM (developing→development) + SYN (higher education→university settings)
SCORE: 4/6 — EXCELLENT
```

## Pattern 2: Definition Expansion/Compression (DEF)

```
INPUT:  "Word sense disambiguation (WSD) is the task of identifying the correct meaning of a word in context."
OUTPUT: "The process of identifying the correct meaning, or sense of a word in context, is known as word sense disambiguation (WSD)."

APPLIED: STR (passive restructure) + DEF (task→process of identifying) + VOC (active→passive)
SCORE: 3/6 — GOOD
```

## Pattern 3: Sentence Break + Recombine (BRK)

```
INPUT:  "The rapid expansion of digital technology has significantly altered communication patterns in contemporary society."
OUTPUT: "Digital technology has expanded quickly in recent years. This growth has transformed how people communicate in modern society."

APPLIED: BRK (1→2 sentences) + SYN (rapid→quickly, altered→transformed) + WFM (expansion→expanded) + STR (restructured)
SCORE: 4/6 — EXCELLENT
```

## Pattern 4: Indonesian Academic Paraphrase

```
INPUT:  "Sistem rekomendasi merupakan sebuah mekanisme yang dapat memberikan saran kepada pengguna berdasarkan preferensi mereka."
OUTPUT: "Mekanisme pemberian saran yang disesuaikan dengan preferensi pengguna dikenal sebagai sistem rekomendasi."

APPLIED: STR (subject→predicate swap) + VOC (active→passive) + WFM (memberikan→pemberian) + DEF (merupakan→dikenal sebagai)
SCORE: 4/6 — EXCELLENT
```

## Pattern 5: Indonesian Methodology Paraphrase

```
INPUT:  "Pengujian dilakukan dengan menggunakan metode black-box testing untuk memvalidasi fungsionalitas sistem."
OUTPUT: "Validasi fungsionalitas sistem dilaksanakan melalui pendekatan pengujian black-box."

APPLIED: STR (clause restructure, fronting) + SYN (dilakukan→dilaksanakan, menggunakan→melalui) + WFM (memvalidasi→validasi) + DEF (metode→pendekatan)
SCORE: 4/6 — EXCELLENT
```

---

# PHASE 4: Quality Scoring

After paraphrasing, SELF-SCORE using this rubric:

| Score | Criteria | Action |
|-------|----------|--------|
| **6/6** | All 6 phenomena applied, meaning preserved | Ship it |
| **5/6** | 5 phenomena, meaning preserved | Ship it |
| **4/6** | 4 phenomena, meaning preserved | Good — ship it |
| **3/6** | 3 phenomena (minimum), meaning preserved | Acceptable |
| **2/6** | Only 2 phenomena — likely just synonym swap | REJECT — redo |
| **1/6** | Only 1 phenomenon — basically copying | REJECT — redo |
| **0/6** | No transformation | REJECT — not paraphrased |

## Meaning Preservation Check

After scoring phenomena, verify:
- [ ] All factual claims identical to original
- [ ] No information added that wasn't in original
- [ ] No information lost that was in original
- [ ] Technical terms preserved exactly
- [ ] Citation references preserved (Author, Year)
- [ ] Numerical data preserved exactly

If ANY meaning check fails → REJECT and redo, regardless of phenomena score.

---

# PHASE 5: Anti-Plagiarism Verification

## Self-BLEU Check (conceptual)

Target: **Self-BLEU below 30** against the original (ParaSCI benchmark: 26.52 for ACL, 29.90 for arXiv):

- Less than 30% n-gram overlap with the original
- Long consecutive word matches (4+ words) should be ZERO (except protected terms)
- Sentence structure must be visibly different

## Red Flags (auto-reject if detected)

1. **Copy-paste with synonym swap**: Same structure, just 2-3 words changed
2. **Patchwork plagiarism**: Phrases lifted directly, rearranged
3. **Mosaic plagiarism**: Mixing original phrases with your own without transformation
4. **Same opening pattern**: If original starts "This study...", don't start "This research..."
5. **Identical clause order**: If original has A→B→C clauses, don't keep A→B→C

## Structural Divergence Rules

- If original is 1 sentence → output can be 1-2 sentences (BRK)
- If original has active voice → consider passive (VOC), but don't always
- If original starts with subject → start with adverbial/prepositional phrase (STR)
- If original uses relative clause → convert to participial or appositive (STR)
- If original is a definition → restructure to classification or description (DEF)

---

# PHASE 6: Output Format

## Single mode (with scoring):

```
### Parafrase

**Original:**
[original text]

**Paraphrased:**
[paraphrased text]

**Phenomena Applied:** SYN, STR, DEF, WFM (4/6)
**Protected Terms:** [list of terms preserved unchanged]
**Meaning Check:** PASS
```

## Bulk mode (clean output for pipeline):

Process paragraph by paragraph:
1. Identify protected terms
2. Apply 3-6 phenomena per sentence
3. Ensure paragraph-level coherence (transitions, flow)
4. Output only the paraphrased text (no scoring annotation)

## PI Pipeline mode:

1. Receive text from BAB generation or tinjauan pustaka
2. Apply academic Indonesian register
3. Preserve all `(Penulis, Tahun)` citations exactly
4. Preserve Gunadarma format compliance:
   - No abbreviations (dll, sbb, dsb → spell out)
   - No personal pronouns (Saya, Anda, Kita)
   - Foreign terms in *italic*
   - Formal academic Indonesian
5. Return clean paraphrased text ready for LaTeX

---

# PHASE 7: Bahasa Indonesia Academic Synonym Bank

## High-Frequency Academic Verb Substitutions

| Original | Alternatives |
|----------|-------------|
| menggunakan | memanfaatkan, menerapkan, mengimplementasikan, mempergunakan |
| melakukan | menjalankan, melaksanakan, mengerjakan, menyelenggarakan |
| menunjukkan | memperlihatkan, mengindikasikan, membuktikan, mendemonstrasikan |
| menghasilkan | membuahkan, memproduksi, mengembangkan, menciptakan |
| mempengaruhi | berdampak pada, berpengaruh terhadap, memberikan dampak pada |
| meningkatkan | menaikkan, mengoptimalkan, memperbaiki, memperkuat |
| mengatasi | menangani, menanggulangi, memecahkan, menyelesaikan |
| menganalisis | mengkaji, meneliti, menelaah, mengulas |
| menyimpulkan | merangkum, menyarikan, mengambil konklusi |
| membandingkan | menyandingkan, mengomparasikan, mengevaluasi perbandingan |

## High-Frequency Academic Noun Substitutions

| Original | Alternatives |
|----------|-------------|
| penelitian | studi, kajian, riset, telaah |
| metode | pendekatan, teknik, cara, prosedur |
| hasil | temuan, luaran, output, capaian |
| masalah | permasalahan, problematika, isu, tantangan |
| tujuan | sasaran, target, objektif |
| sistem | mekanisme, kerangka, infrastruktur, arsitektur |
| data | informasi, keterangan, fakta |
| proses | tahapan, mekanisme, alur, prosedur |
| pengembangan | perancangan, pembangunan, pembuatan |
| penerapan | implementasi, pemanfaatan, penggunaan, aplikasi |

## High-Frequency Academic Connector Substitutions

| Original | Alternatives |
|----------|-------------|
| oleh karena itu | dengan demikian, maka dari itu, berdasarkan hal tersebut |
| selain itu | di samping itu, lebih lanjut, tidak hanya itu |
| namun | akan tetapi, meskipun demikian, di sisi lain |
| sehingga | dengan begitu, yang mengakibatkan, yang berdampak pada |
| berdasarkan | merujuk pada, mengacu pada, berlandaskan |
| menurut | sebagaimana dikemukakan oleh, berdasarkan pandangan |

## English Academic Synonym Bank (for mixed-language PI)

| Original | Alternatives |
|----------|-------------|
| propose | introduce, present, put forward, suggest |
| demonstrate | show, illustrate, indicate, reveal, establish |
| investigate | examine, explore, analyze, study, assess |
| significant | substantial, considerable, notable, meaningful |
| utilize | employ, leverage, apply, make use of, adopt |
| achieve | attain, accomplish, obtain, realize, reach |
| improve | enhance, optimize, refine, advance, strengthen |
| develop | design, construct, build, create, formulate |
| evaluate | assess, measure, appraise, gauge, benchmark |
| implement | deploy, execute, operationalize, put into practice |

---

# PHASE 8: Paragraph-Level Coherence Rules

When paraphrasing multiple sentences in a paragraph:

1. **Maintain topic sentence position** — if original starts with topic sentence, paraphrased version should too
2. **Preserve logical flow** — cause→effect, general→specific, claim→evidence relationships must be maintained
3. **Vary techniques across sentences** — don't apply the same phenomena pattern to every sentence
4. **Transition consistency** — if original uses additive transitions, paraphrased can use different additive transitions but not contrastive ones
5. **Information density match** — if original is dense with claims, paraphrased should be equally dense
6. **Cross-sentence reference** — ensure pronouns and demonstratives still point to the correct referent after restructuring

---

# PHASE 9: AI-Tell Audit (Post-Paraphrase Detection)

Source: Adapted from Wikipedia:Signs of AI writing (WikiProject AI Cleanup). Filtered to patterns most relevant to academic Indonesian/English writing.

## Purpose

Paraphrasing (Phases 0-8) transforms text structurally. But the OUTPUT may still contain AI-generated artifacts. This phase detects and removes those tells AFTER paraphrasing is complete.

## The 12 Academic AI-Tell Patterns

| # | Pattern | What to Look For | Fix |
|---|---------|-----------------|-----|
| A1 | **AI Vocabulary** | "delve", "landscape", "tapestry", "foster", "crucial", "pivotal", "multifaceted" | Replace with simpler, field-specific terms |
| A2 | **Copula Avoidance** | "serves as" instead of "is", "functions as" instead of "is" | Use direct copula when appropriate |
| A3 | **Synonym Cycling** | Same concept called 3+ different names in one paragraph | Pick ONE term per concept per paragraph |
| A4 | **Significance Inflation** | "This pivotal study...", "This groundbreaking research..." | State what the study DID, not how important it is |
| A5 | **Vague Attribution** | "Experts believe...", "Research suggests..." without citation | Replace with specific (Author, Year) or remove |
| A6 | **Excessive Hedging** | "could potentially possibly", "might perhaps" | One hedge per claim maximum |
| A7 | **Formulaic Transitions** | "Furthermore", "Moreover", "Additionally" at every paragraph start | Vary or use implicit transitions |
| A8 | **Rule of Three** | Forcing ideas into groups of three when only 2 are relevant | Include only what's actually relevant |
| A9 | **Generic Conclusions** | "Further research is needed" without specifics | State WHAT specific research, on WHAT gap |
| A10 | **Superficial -ing Analysis** | "...highlighting the importance of...", "...underscoring the need for..." | State the actual finding directly |
| A11 | **Passive Overuse** | Every sentence in passive voice with no actor | Mix active/passive; name the actor |
| A12 | **Uniform Sentence Length** | All sentences roughly the same length | Vary: mix short, medium, and long |

## Bahasa Indonesia Specific AI-Tells

| # | Pattern | Contoh | Fix |
|---|---------|--------|-----|
| B1 | **"Dalam era..."** opener | "Dalam era digital saat ini..." | Start with the actual subject matter |
| B2 | **"Sangat penting"** inflation | "Hal ini sangat penting untuk..." | State the specific role without inflation |
| B3 | **"Berbagai"** filler | "Berbagai macam penelitian..." | Be specific: name them or give count |
| B4 | **Connector stacking** | "Selain itu, lebih lanjut, di samping itu..." | One connector per transition |

## The 3-Pass Audit Process

```
PASS 1 — SCAN
  Read paraphrased output. Check each sentence against A1-A12 + B1-B4. Mark matches.

PASS 2 — SELF-AUDIT
  Ask: "Jika dosen membaca ini, apakah terasa seperti tulisan AI/ChatGPT?"
  Answer honestly. List remaining tells.

PASS 3 — FIX
  Replace AI vocabulary, break uniform rhythm, remove inflation, ensure term consistency.
  Verify fix doesn't break meaning preservation.
```

## Audit Verdict

| AI-Tell Count | Verdict | Action |
|---------------|---------|--------|
| 0 tells | CLEAN | Ship it |
| 1-2 tells | MINOR | Fix and ship |
| 3-5 tells | MODERATE | Fix all, re-audit once |
| 6+ tells | HEAVY | Re-paraphrase from scratch |

## Term Consistency Rule (Anti-Synonym-Cycling)

Once a technical term is introduced, use THAT EXACT TERM throughout the paragraph. Synonym substitution (SYN) applies to VERBS and CONNECTORS, NOT to technical nouns/concepts.

```
WRONG: "OCR technology... This optical recognition system... The text extraction technology..."
RIGHT: "OCR technology... The OCR system... This technology..."
```

---

# Integration Points

- **Draft Generator**: Triggered after generation to refine tone (bulk mode)
- **Quality Layer**: Used as recovery tool for failed formal-tone checks
- **Style Calibration**: Consumes Style Profile to match author's natural voice
- **Plagiarism Guard**: Paraphraser output should pass anti-plagiarism checks

# References

1. Dong, Q., Wan, X., & Cao, Y. (2021). ParaSCI: A Large Scientific Paraphrase Dataset for Longer Paraphrase Generation. *EACL 2021*.
2. Al-Ghidani, A. H., & Fahmy, A. A. (2018). Conditional Text Paraphrasing: A Survey and Taxonomy. *IJACSA*, 9(11).
3. Boonthum, C. (2004). iSTART: Paraphrase Recognition. 6 paraphrase phenomenon categories.
4. Zhang, Y., et al. (2019). PAWS: Paraphrase Adversaries from Word Scrambling. 108K human-labeled pairs.
5. Wikipedia:Signs of AI writing (WikiProject AI Cleanup). Crowdsourced taxonomy of AI writing patterns. Adapted for Phase 9 AI-Tell Audit.
6. Chen, S. (2025). Humanizer v2.5.1. AI-tell detection and removal skill. github.com/blader/humanizer.
