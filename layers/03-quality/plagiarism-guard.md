---
name: plagiarism-guard
description: Detects plagiarism risks and AI-generated patterns in Penelitian Ilmiah (PI) content.
---

# Purpose
To maintain academic integrity by ensuring all ideas are properly attributed, content is not verbatim copied from sources, and the writing avoids identifiable "AI slop" patterns.

# Input
- Manuscript section or full text.
- Source material excerpts (if provided by search agents).

# Output
- Plagiarism Risk Report.
- AI Pattern Score.
- Flagged sections requiring paraphrase.

# Checks

## Attribution
- Idea Source: Every non-original idea, data point, or diagram MUST have an accompanying citation.
- Direct Quotes: MUST be kept to a minimum. If used, they MUST be clearly marked with quotation marks and proper citation.

## AI Slop Detection
- Pattern Identification: Detect and flag overused AI-generated vocabulary ("delve", "crucial", "comprehensive", "robust", "streamline", "leverage", "it's important to note").
- Sentence Variation: Flag long sequences of sentences with similar lengths or repetitive structures.
- Filler Content: Detect and remove empty introductory phrases (e.g., "In the rapidly evolving landscape of...").

## Verbatim Match (Local Check)
- Source Comparison: If source texts are available in the system context, the checker MUST perform a string similarity check to ensure content has been properly paraphrased.

## Paraphrase Quality
- Meaning Preservation: Ensure that changes made to the text (via fixes) do not alter the technical meaning of the research.
- Bridge: Flag sections that score high for plagiarism risk and recommend the use of `academic-paraphraser` skill.

# Severity Levels
- CRITICAL: Verbatim copying without quotes, missing attribution for key data, high AI slop score (over 70%).
- WARNING: Repetitive sentence structure, overuse of "comprehensive" or "delve", poor paraphrase quality.
- INFO: Minor suggestions to vary sentence length.

# Integration Points
- Writing Layer: Filters generated content before it is committed to the main draft.
- Academic-Paraphraser: Receives flagged sections for automated rewriting.
- Final Review: Documents with CRITICAL plagiarism flags are automatically sent back for revision.
