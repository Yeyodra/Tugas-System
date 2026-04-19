---
layer: 04-output
type: skill
name: docx-converter
---

# Purpose

Converts tugas (assignment) content from Markdown or LaTeX sources to DOCX format while maintaining formatting standards defined by the task-type YAML.

# Input

- Markdown or LaTeX source file(s).
- Reference DOCX template (from `templates/docx/` if available).
- Format rules from format-rules layer (margins, font, spacing).
- Pandoc tool.

# Output

- Formatted DOCX document.

# Process

## 1. Source Preparation

If starting from LaTeX:
- Convert to clean Markdown first using Pandoc: `pandoc input.tex -o intermediate.md`
- Clean up LaTeX-specific artifacts (e.g., `\textbf{}` → `**bold**`).
- Preserve code blocks, images, and tables.

If starting from Markdown:
- Use directly.

## 2. DOCX Generation via Pandoc

Use the `pandoc_convert-contents` tool or command line:

```bash
pandoc input.md -o output.docx --reference-doc=templates/docx/reference-tugas.docx
```

If no reference document exists, generate without one (Pandoc defaults):
```bash
pandoc input.md -o output.docx
```

## 3. Style Mapping

Map source headings to DOCX styles:

| Source | DOCX Style |
|--------|-----------|
| `# Heading` or `\section{}` | Heading 1 (14pt bold) |
| `## Heading` or `\subsection{}` | Heading 2 (12pt bold) |
| Body text | Normal (12pt, 1.5 spacing, justified) |
| Code blocks | Code style (monospace, smaller font) |
| `> Blockquote` | Quote style |

## 4. Cover Page Handling

If the task-type requires a cover page:
- Generate the cover page content as the first page of the DOCX.
- Include: title (bold, centered, uppercase), logo image, student info table, university name, year.
- Insert a page break after the cover.

If no cover is required (e.g., simple homework), start directly with content.

## 5. Post-Processing Notes

Pandoc handles most formatting, but some elements may need manual adjustment:
- Verify margins match task-type requirements (default: 3cm all sides).
- Check that images are properly sized and positioned.
- Verify code blocks render with correct font and background.
- Refresh Table of Contents if present.

# Integration Points

- **format-rules** (layer 01): Provides format requirements for style mapping.
- **content-generator** (layer 02): Provides the Markdown/LaTeX source.
- **progress-tracker** (layer 04): Updates export status.
- **Templates**: `templates/docx/reference-tugas.docx` (optional reference document).
