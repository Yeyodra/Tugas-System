---
name: reference-finder
description: >
  Simplified citation engine for general academic tasks. Finds references by topic,
  verifies they exist (DOI check), filters by year, and formats as bibliography entries.
  Adapted from PI System's citation-engine.md but without Q1 Scopus filtering or
  quartile verification. Designed for tugas/homework-level references.
---

# Purpose

Find real, verifiable academic references for any topic and format them as bibliography entries. Unlike the PI System's citation-engine (which enforces Q1 Scopus verification), this layer is designed for homework-level tasks where the bar is "real paper, recent, relevant" not "Q1 journal only."

**Key differences from PI citation-engine:**
- NO quartile verification (no SCImago lookup)
- NO Q1/Q2 filtering
- Accepts conference papers, theses, and reputable web sources
- Simpler output format
- Faster with fewer verification steps

# Input

| Parameter | Required | Description |
|-----------|----------|-------------|
| `topic` | YES | Topic or keywords to search for |
| `count` | NO | Number of references needed (default: 5) |
| `max_age_years` | NO | Max paper age (default: 5) |
| `format` | NO | `thebibliography` (default), `biblatex`, or `apa_text` |
| `source_types` | NO | Allowed: `journal`, `conference`, `thesis`, `book`, `web` (default: all) |
| `language_pref` | NO | Prefer papers in this language (default: any) |

# Output

```json
{
  "references": [
    {
      "cite_key": "smith2024ml",
      "title": "Machine Learning for Weather Prediction",
      "authors": "Smith, J. A., & Lee, K.",
      "year": 2024,
      "source": "Journal of Applied Meteorology",
      "doi": "10.1234/jam.2024.001",
      "url": "https://doi.org/10.1234/jam.2024.001",
      "type": "journal",
      "verified": true
    }
  ],
  "formatted": {
    "thebibliography": "\\bibitem{smith2024ml}\nSmith, J. A., dan Lee, K. 2024. ...",
    "apa_text": "Smith, J. A., & Lee, K. (2024). Machine Learning for..."
  },
  "search_metadata": {
    "queries_used": ["..."],
    "total_found": 15,
    "verified": 8,
    "returned": 5
  }
}
```

---

# Process

## Step 1: SEARCH -- Find Candidate References

### 1.1 Build Search Queries

From the topic, construct 2-3 search queries:

```
Query 1 (academic): "{topic}" research paper journal {year_range}
Query 2 (DOI-targeted): site:doi.org "{topic_keywords}" {year_range}
Query 3 (broader): {topic_keywords} study analysis academic {year_range}
```

**Year range calculation:**
```
current_year = 2026
min_year = current_year - max_age_years  # default: 2021
year_filter = "2021 2022 2023 2024 2025 2026"
```

### 1.2 Execute Searches

**Primary: Firecrawl Search**
```
firecrawl_search(
  query="{query}",
  limit=15,
  sources=[{"type": "web"}]
)
```

**Secondary: Exa Web Search**
```
web_search_exa(
  query="academic paper about {topic} published after {min_year}",
  numResults=10
)
```

**Fallback: Semantic Scholar API**
```
GET https://api.semanticscholar.org/graph/v1/paper/search
  ?query={topic_keywords}
  &limit=10
  &fields=title,authors,year,externalIds,journal,url,isOpenAccess
  &year={min_year}-
```

Note: Rate limit 1 req/3 sec on free tier. Add 3-5 second delay between requests.

### 1.3 Merge and Deduplicate

Combine results from all sources:
- Dedup by DOI (exact match)
- Dedup by title (lowercase, strip whitespace, >85% similarity = same paper)
- Keep the version with most complete metadata

---

## Step 2: VERIFY -- Check References Exist

For each candidate reference, verify it is real:

### 2.1 DOI Verification

If DOI is available:
```
firecrawl_scrape(
  url="https://doi.org/{doi}",
  formats=["json"],
  jsonOptions={
    "prompt": "Extract: title, authors, journal/conference name, year, volume, pages, DOI",
    "schema": {
      "type": "object",
      "properties": {
        "title": {"type": "string"},
        "authors": {"type": "string"},
        "source_name": {"type": "string"},
        "year": {"type": "integer"},
        "volume": {"type": "string"},
        "pages": {"type": "string"},
        "doi": {"type": "string"}
      }
    }
  }
)
```

**Verification result:**
- DOI resolves + metadata matches: `verified: true`
- DOI resolves but metadata differs: update metadata from DOI source
- DOI does not resolve: `verified: false`, try URL verification

### 2.2 URL Verification (Fallback)

If no DOI but URL available:
- Check URL is accessible (not 404)
- Verify it points to an academic source (publisher site, university repository, etc.)
- Extract metadata from page

### 2.3 Drop Unverifiable

Remove references that:
- Have no DOI AND no working URL
- DOI does not resolve AND URL is dead
- Metadata is incomplete (no title or no authors)

---

## Step 3: FILTER -- Apply Criteria

```
KEEP if:
  - year >= min_year (default: 2021)
  - type in allowed source_types (default: all)
  - verified == true
  - has at least: title + authors + year + (doi OR url)

DROP if:
  - year < min_year
  - type not in source_types
  - unverifiable
  - duplicate
```

### Shortfall Handling

If after filtering, count < requested `count`:

1. **Expand search**: Add synonym keywords, broaden topic
2. **Extend year range**: Go back 2 more years
3. **Include more source types**: Add conference papers, theses
4. **Report gap**: "Hanya ditemukan {n} dari {count} referensi yang diminta untuk topik '{topic}'."

---

## Step 4: FORMAT -- Generate Bibliography Entries

### 4.1 Cite Key Generation

Pattern: `{first_author_lastname}{year}{keyword}`
- Lowercase, no spaces
- Keyword: first significant word from title
- Example: `smith2024machine`, `lee2023deep`

### 4.2 thebibliography Format (Default)

Indonesian academic style (Gunadarma convention):

```latex
\begin{thebibliography}{99}

\bibitem{smith2024machine}
Smith, J. A., dan Lee, K. 2024. Machine Learning for Weather Prediction: A Comprehensive Study. \textit{Journal of Applied Meteorology}. Vol. 45, No. 3, hal. 112-128. DOI: 10.1234/jam.2024.001.

\bibitem{wang2023deep}
Wang, X., Chen, Y., dan Zhang, Z. 2023. Deep Learning Approaches in Climate Science. Dalam \textit{Proceedings of ICML 2023}. hal. 456-470.

\end{thebibliography}
```

**Format rules:**
- Authors: Last, F. M., dan Last, F. M. (use "dan" not "&")
- Year after authors, followed by period
- Title in sentence case, followed by period
- Journal in italic, followed by period
- Volume, Number, Pages: "Vol. X, No. Y, hal. Z-ZZ."
- DOI at end if available
- Conference: "Dalam *Proceedings of Conference Name*. hal. X-Y."

### 4.3 BibLaTeX Format

```bibtex
@article{smith2024machine,
  author  = {Smith, John A. and Lee, Kyung},
  title   = {Machine Learning for Weather Prediction: A Comprehensive Study},
  journal = {Journal of Applied Meteorology},
  year    = {2024},
  volume  = {45},
  number  = {3},
  pages   = {112--128},
  doi     = {10.1234/jam.2024.001}
}
```

### 4.4 APA 7 Text Format

```
Smith, J. A., & Lee, K. (2024). Machine learning for weather prediction: A comprehensive study. Journal of Applied Meteorology, 45(3), 112-128. https://doi.org/10.1234/jam.2024.001
```

### 4.5 Source Type Formatting

| Type | Format Pattern |
|------|---------------|
| Journal article | Author. Year. Title. *Journal*. Vol, No, hal. |
| Conference paper | Author. Year. Title. Dalam *Proceedings of Conf*. hal. |
| Book | Author. Year. *Title*. Publisher. |
| Book chapter | Author. Year. Chapter title. Dalam Editor (Ed.), *Book Title* (hal. X-Y). Publisher. |
| Thesis | Author. Year. *Title* (Skripsi/Tesis/Disertasi). University. |
| Web source | Author/Org. Year. Title. Retrieved from URL. |

---

# Quality Checks

| Check | Action |
|-------|--------|
| All references have DOI or URL | Warn if any lack both |
| No duplicate references | Dedup by DOI + title |
| Year range respected | Drop out-of-range entries |
| Metadata complete | At minimum: title, authors, year, source |
| Format consistent | All entries follow same citation style |
| Cite keys unique | No duplicate keys |
| References are real | DOI resolves or URL accessible |

---

# Error Handling

| Error | Action |
|-------|--------|
| No results for topic | Suggest broader keywords |
| All results too old | Extend year range, inform user |
| Semantic Scholar rate limited | Use Firecrawl/Exa only |
| DOI scrape fails | Use available metadata, mark as `verified: false` |
| Requested count not met | Return what is available + shortfall message |

---

# Comparison with PI Citation Engine

| Feature | PI citation-engine | reference-finder (this) |
|---------|-------------------|------------------------|
| Quartile verification | YES (SCImago) | NO |
| Q1/Q2 filtering | YES (strict) | NO |
| Source types | Journals + top conferences | All academic sources |
| Verification depth | DOI + SCImago + ISSN | DOI only |
| Output format | verified_q1_papers.json + bibitem | Simple JSON + bibitem/bibtex/APA |
| Search sources | Firecrawl + Exa + OpenDraft + S2 | Firecrawl + Exa + S2 |
| Speed | Slow (quartile checks) | Fast (skip quartile) |
| Use case | PI/skripsi (strict academic) | Tugas/homework (practical) |

**When to use which:**
- Writing PI/skripsi with dosen requirements for Q1 journals: use PI citation-engine
- Writing homework/tugas that just needs real references: use this reference-finder

---

# Integration Points

| Layer | Relationship |
|-------|-------------|
| `content-generator.md` | Called for `type: reference` sections in task-type YAML |
| `journal-analyzer.md` | Can provide additional references beyond the analyzed paper |
| `plagiarism-guard.md` | Verifies cited sources match actual bibliography |
| `file-safety.md` | Backup protocol if overwriting existing bibliography |
