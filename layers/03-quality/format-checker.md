---
layer: 03-quality
type: skill
name: format-checker
---

# Purpose

Validates assignment document formatting against rules loaded from the task-type YAML configuration. Rules are NOT hardcoded — they come from the format-rules layer.

# Config Loading (MANDATORY — do this first)

> **Load format rules** by invoking the `format-rules` layer (01-knowledge) with:
> 1. Explicit task-type YAML path (if provided by caller), OR
> 2. Auto-resolve from `semester.json → tasks[active] → task_type`
>
> The format-rules layer returns a normalized Format Rules Object.
> All checks below reference fields from that object.

# Input

- Document to validate (Markdown, DOCX, or LaTeX).
- Format Rules Object (from format-rules layer).

# Output

- Validation Report (JSON or Markdown).
- Status: PASS/FAIL per rule.
- List of violations with location (section, line number, or page).

# Checks

## Document Dimensions

- **Paper Size**: MUST match `rules.format.paper_size`.
- **Margins**: MUST match `rules.format.margins` (top, bottom, left, right in cm).

## Typography

- **Body Text**: Font family from `rules.format.font.family`, size from `rules.format.font.size_body`.
- **Section Titles**: Font size from `rules.format.font.size_title`, bold.
- **Subsection Titles**: Size from `rules.format.font.size_body`, bold.

## Paragraphs and Spacing

- **Line Spacing**: Must match `rules.format.line_spacing`.
- **Indentation**: First line indent must match `rules.format.paragraph_indent`.
- **Alignment**: Must match `rules.format.text_alignment`.

## Cover Page (if required)

If `rules.cover.required` is true, verify presence of:
- Title (bold, centered, uppercase).
- University logo.
- Student name and NPM.
- Class.
- Course name (if in `rules.cover.elements`).
- Lecturer name (if in `rules.cover.elements`).
- University name.
- Year.

## Section Structure

- If `rules.sections.numbering` is true, verify sections are numbered.
- Verify section title formatting matches `rules.sections.title_format`.

## Code Listings (if applicable)

If `rules.code_listings.enabled` is true:
- Verify code blocks use monospace font.
- Verify code blocks have line numbers (if style requires).
- Check that code is not cut off or overflowing margins.

## Citations (if applicable)

If `rules.citations.required` is true:
- Verify citation style matches `rules.citations.style`.
- Verify minimum reference count meets `rules.citations.min_references`.
- Check that all `\cite{}` references have matching bibliography entries.

# Severity Levels

- **CRITICAL**: Margin violations, wrong font family/size, missing cover page elements (when required), missing indentation.
- **WARNING**: Minor spacing inconsistencies, code block formatting issues, image sizing.
- **INFO**: Suggestions for readability improvement, paragraph length optimization.

# Report Format

```markdown
## Format Validation Report
**Task**: {task_title}
**Date**: {date}
**Rules Source**: {yaml_path}

### Summary
- CRITICAL: {count}
- WARNING: {count}
- INFO: {count}
- **Result**: PASS / FAIL

### Violations

#### CRITICAL
1. [Line 15] Font size is 11pt, expected 12pt (rules.format.font.size_body)
2. [Cover] Missing university logo (rules.cover.elements)

#### WARNING
1. [Section 2.1] Line spacing appears to be 2.0, expected 1.5

#### INFO
1. [Section 3] Paragraph exceeds 200 words — consider splitting.
```

# Integration Points

- **format-rules** (layer 01): Provides the Format Rules Object — MUST be loaded first.
- **content-generator** (layer 02): Receives feedback for auto-correction of violations.
- **progress-tracker** (layer 04): Updates QA status after validation.
- **latex-compiler** (layer 04): Only compiles documents with zero CRITICAL violations.
