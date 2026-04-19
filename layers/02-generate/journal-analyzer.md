---
name: journal-analyzer
description: >
  Specialized layer for "analisis jurnal" type tasks. Searches for journal papers,
  reads/scrapes content, extracts structured analysis (latar belakang, metode,
  pembahasan), and generates proper APA 7 citations. Works for any matkul/topic.
---

# Purpose

Find a relevant journal paper for a given topic, read its content, and produce structured analysis data that `content-generator.md` uses to fill sections. This is NOT limited to one subject — it works for any matkul by adapting search queries to the topic.

# Input

| Parameter | Required | Description |
|-----------|----------|-------------|
| `topic` | YES | Topic or keyword to search for |
| `matkul` | YES | Course name (for context in analysis) |
| `journal_url` | NO | If user already has a specific paper URL |
| `criteria.max_age_years` | NO | Max paper age (default: 5) |
| `criteria.open_access` | NO | Prefer open access (default: true) |
| `criteria.language` | NO | Paper language preference (default: any) |

# Output

```json
{
  "paper": {
    "title": "...",
    "authors": "...",
    "journal": "...",
    "year": 2024,
    "doi": "10.xxxx/xxxxx",
    "url": "https://...",
    "abstract": "...",
    "citation_apa7": "Author, A. A., & Author, B. B. (2024). Title. Journal, Vol(Issue), pp-pp. https://doi.org/..."
  },
  "analysis": {
    "latar_belakang": "Generated background text...",
    "metode": "Generated methodology analysis...",
    "hasil": "Generated results summary...",
    "pembahasan": "Generated discussion/analysis...",
    "kelebihan": ["strength 1", "strength 2"],
    "kekurangan": ["weakness 1", "weakness 2"],
    "relevansi": "How this relates to {matkul}..."
  },
  "search_metadata": {
    "query_used": "...",
    "sources_searched": ["firecrawl", "semantic_scholar"],
    "candidates_found": 8,
    "selected_reason": "Most relevant, open access, recent"
  }
}
```

---

# Process

## Step 1: SEARCH — Find Candidate Papers

### If `journal_url` provided:

Skip search, go directly to Step 2 (Scrape).

### If searching by topic:

#### 1.1 Build Search Queries

Construct 2-3 search queries from the topic:

```
Query 1 (specific): "{topic}" research paper journal {year_range}
Query 2 (broader):  {topic_keywords} study analysis {year_range}
Query 3 (academic): site:doi.org OR site:ieee.org OR site:sciencedirect.com "{topic}" {year_range}
```

**Topic keyword extraction:**
- Split topic into key terms
- Add field-specific synonyms based on `matkul`
- Example: topic "machine learning untuk prediksi cuaca"
  - Keywords: "machine learning", "weather prediction", "forecasting"
  - Matkul context: if "Kecerdasan Buatan" → add "artificial intelligence", "neural network"

#### 1.2 Execute Parallel Searches

**Primary: Firecrawl Search**
```
firecrawl_search(
  query="{constructed_query}",
  limit=10,
  sources=[{"type": "web"}]
)
```

**Secondary: Exa Web Search**
```
web_search_exa(
  query="research paper journal article about {topic} published {year_range}",
  numResults=5
)
```

**Fallback: Semantic Scholar API**
```
GET https://api.semanticscholar.org/graph/v1/paper/search
  ?query={topic_keywords}
  &limit=5
  &fields=title,authors,year,externalIds,journal,abstract,isOpenAccess,url
  &year={min_year}-
```

Note: Semantic Scholar free tier has strict rate limits (1 req/3 sec). Use delay between requests.

#### 1.3 Rank and Select Best Paper

Score each candidate:

| Criterion | Weight | Scoring |
|-----------|--------|---------|
| Relevance to topic | 30% | Keyword match in title + abstract |
| Recency | 20% | Newer = higher score |
| Open access | 20% | Open access = +20, closed = 0 |
| Has DOI | 15% | DOI present = +15, absent = 0 |
| Journal reputation | 15% | Known publisher (IEEE, Springer, Elsevier) = +15 |

Select the highest-scoring paper. If top 2 are close (within 5%), prefer the one with open access.

**Present selection to user:**
```
Ditemukan paper yang relevan:
  "{paper_title}"
  {authors} ({year})
  {journal}
  DOI: {doi}
  
Lanjut analisis paper ini? [Y/n]
```

If user rejects → show next candidate or refine search.

---

## Step 2: SCRAPE — Read Paper Content

### 2.1 Scrape via DOI/URL

**Primary method: Firecrawl Scrape**
```
firecrawl_scrape(
  url="{paper_url_or_doi_url}",
  formats=["markdown"],
  onlyMainContent=true
)
```

**If DOI available, also scrape metadata:**
```
firecrawl_scrape(
  url="https://doi.org/{doi}",
  formats=["json"],
  jsonOptions={
    "prompt": "Extract: title, authors (full list), journal name, year, volume, issue, pages, DOI, abstract",
    "schema": {
      "type": "object",
      "properties": {
        "title": {"type": "string"},
        "authors": {"type": "string"},
        "journal": {"type": "string"},
        "year": {"type": "integer"},
        "volume": {"type": "string"},
        "issue": {"type": "string"},
        "pages": {"type": "string"},
        "doi": {"type": "string"},
        "abstract": {"type": "string"}
      }
    }
  }
)
```

### 2.2 Extract Key Sections

From the scraped content, identify and extract:

| Section | What to Look For |
|---------|-----------------|
| Abstract | Usually clearly labeled; first substantial paragraph |
| Introduction / Background | Section 1 or "Introduction" heading |
| Methodology | "Method", "Methodology", "Approach", "Materials and Methods" |
| Results | "Results", "Findings", "Experiments" |
| Discussion | "Discussion", "Analysis" |
| Conclusion | "Conclusion", "Summary" |

**If full text not accessible** (paywall):
1. Use abstract + any available preview text
2. Search for preprint version (arXiv, ResearchGate)
3. Generate analysis from abstract only — mark as `[Analisis berdasarkan abstrak — full text tidak tersedia]`

### 2.3 Build Paper Data Object

```json
{
  "title": "extracted title",
  "authors": "Author, A., Author, B., & Author, C.",
  "journal": "Journal Name",
  "year": 2024,
  "doi": "10.xxxx/xxxxx",
  "url": "https://...",
  "abstract": "full abstract text",
  "sections": {
    "introduction": "extracted or summarized intro",
    "methodology": "extracted or summarized methods",
    "results": "extracted or summarized results",
    "discussion": "extracted or summarized discussion",
    "conclusion": "extracted or summarized conclusion"
  },
  "full_text_available": true
}
```

---

## Step 3: ANALYZE — Generate Structured Analysis

Using the extracted paper content, generate analysis sections in formal Indonesian academic style.

### 3.1 Latar Belakang (Background)

**Prompt pattern:**
```
Based on the paper "{title}" by {authors} ({year}), write a background section that:
1. Introduces the research domain and its importance
2. Identifies the problem the paper addresses
3. States the paper's research objective
4. Explains why this topic is relevant to {matkul}

Write in formal Indonesian academic style. 150-300 words.
Use the paper's introduction as primary source.
```

### 3.2 Metode Penelitian (Methodology)

**Prompt pattern:**
```
Analyze the methodology of "{title}":
1. Research approach (qualitative/quantitative/mixed)
2. Data collection methods
3. Tools/technologies used
4. Analysis techniques
5. Evaluation metrics (if applicable)

Write in formal Indonesian. Be specific — name the actual methods used.
150-300 words.
```

### 3.3 Hasil dan Pembahasan (Results & Discussion)

**Prompt pattern:**
```
Analyze the results and findings of "{title}":
1. Key findings (with specific numbers/metrics if available)
2. How findings relate to the research objectives
3. Comparison with previous work (if mentioned in paper)
4. Strengths of the approach
5. Limitations acknowledged by the authors
6. Your critical assessment

Write in formal Indonesian. 200-500 words.
```

### 3.4 Kelebihan dan Kekurangan (Strengths & Weaknesses)

Extract as bullet points:
- **Kelebihan**: methodology rigor, novelty, practical applicability, dataset quality
- **Kekurangan**: limited scope, small dataset, no comparison, reproducibility issues

### 3.5 Relevansi (Relevance to Course)

**Prompt pattern:**
```
Explain how "{title}" is relevant to the course {matkul}.
Connect the paper's topic, methods, or findings to concepts taught in {matkul}.
2-3 sentences. Formal Indonesian.
```

---

## Step 4: CITE — Generate Proper Citation

### APA 7th Edition Format

**Journal article:**
```
Author, A. A., Author, B. B., & Author, C. C. (Year). Title of article. Title of Periodical, Volume(Issue), Page–Page. https://doi.org/xxxxx
```

**Examples:**
```
# 1-2 authors: list all
Smith, J. A., & Lee, K. (2024). Deep learning for OCR. Pattern Recognition, 145, 109-120. https://doi.org/10.1016/j.patcog.2024.109

# 3-20 authors: list all
Smith, J. A., Lee, K., Wang, X., & Chen, Y. (2024). Title. Journal, 10(2), 50-65.

# 21+ authors: first 19 ... last author
Smith, J. A., Lee, K., Wang, X., ... & Zhang, Z. (2024). Title. Journal, 10(2), 50-65.
```

**Conference paper:**
```
Author, A. A. (Year). Title of paper. In Proceedings of Conference Name (pp. Page–Page). Publisher. https://doi.org/xxxxx
```

### Indonesian Adaptation

For Indonesian academic context, also provide the Gunadarma-style citation:
```
Author, A. A., Author, B. B., dan Author, C. C. Year. Title. \textit{Journal}. Vol. X, No. Y, hal. Z-ZZ.
```

Note: "dan" instead of "&", "hal." instead of "pp.", "Vol." and "No." spelled out.

---

## Step 5: PACKAGE — Return Structured Data

Package all analysis into the output format defined above. This data is consumed by `content-generator.md` to fill sections in the task-type YAML.

The `analysis` object maps directly to section IDs in the YAML:
- `analysis.latar_belakang` → section `latar_belakang`
- `analysis.metode` → section `metode`
- etc.

---

# Quality Checks

Before returning analysis:

| Check | Action |
|-------|--------|
| Analysis mentions specific details from paper | PASS — not generic filler |
| Numbers/metrics from paper are accurate | Verify against scraped data |
| Citation format is correct APA 7 | Validate structure |
| DOI resolves | Quick check via URL |
| No AI-tell patterns in generated text | Run quick A1-A12 scan |
| Analysis is in correct language | Match YAML `language` setting |

---

# Error Handling

| Error | Action |
|-------|--------|
| No papers found for topic | Broaden search terms, suggest alternative keywords |
| Paper behind paywall | Use abstract-only analysis, note limitation |
| DOI doesn't resolve | Search by title instead, warn user |
| Semantic Scholar rate limited | Fall back to Firecrawl only |
| Scrape returns empty content | Try alternative URL (preprint, ResearchGate) |
| Paper not in expected language | Translate key sections, note in analysis |

---

# Integration Points

| Layer | Relationship |
|-------|-------------|
| `content-generator.md` | Primary consumer — receives structured analysis data |
| `reference-finder.md` | Can be called for additional references beyond the analyzed paper |
| `paraphraser.md` | Post-processes analysis text if needed |
| `plagiarism-guard.md` | Verifies analysis is not verbatim from paper |
