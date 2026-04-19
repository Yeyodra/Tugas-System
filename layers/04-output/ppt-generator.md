---
layer: 04-output
type: skill
name: ppt-generator
---

# Purpose

Generates .pptx presentation slides from tugas (assignment) content. Supports generic academic presentations for class assignments, group projects, and course presentations.

# Input

- Completed assignment content (Markdown, LaTeX, or structured text).
- Task metadata from `semester.json` (judul, nama, npm, kelas, matakuliah, dosen).
- Diagram files and figures from the assignment output directory.
- User-specified presentation style and language.

# Output

- Professional .pptx file.
- Language: id (default), en, or id-en (bilingual).

# Process

## 1. Content Extraction

Read the assignment content and identify:
- Title and metadata (from semester.json or document header).
- Section structure (headings → slide topics).
- Key points, definitions, and explanations.
- Code snippets (if programming assignment).
- Figures, tables, and diagrams.
- Conclusions or results.

## 2. Content Structuring

Map content to the selected presentation template:

### Templates

**standard** (default): ~10-15 slides for a 10-15 min presentation.
- Slide 1: Cover (title, name, NPM, class, course, university logo)
- Slide 2: Outline / Daftar Isi
- Slides 3-N: Content sections (one section per 1-3 slides)
- Slide N+1: Conclusion / Kesimpulan
- Slide N+2: References / Daftar Pustaka (if applicable)
- Slide N+3: Thank You / Terima Kasih

**brief**: ~5-8 slides for a 5 min presentation.
- Cover, 3-5 content slides, conclusion, thank you.

**detailed**: ~15-25 slides for a 20-30 min presentation.
- Cover, outline, detailed content with sub-sections, discussion, conclusion, references, Q&A.

## 3. Figure Integration

- Embed existing images/diagrams from the assignment.
- For data-heavy assignments, generate summary charts via Matplotlib/Seaborn.
- Size figures appropriately for slide readability (min 60% slide width).

## 4. Slide Generation

Construct the presentation using `python-pptx`:
- Apply consistent styling: font family, colors, layout.
- Use university branding colors where appropriate.
- Ensure text readability (min 18pt for body, 24pt+ for titles).
- Limit bullet points to 5-6 per slide.
- Include slide numbers.

## 5. Code Slides (for programming assignments)

- Use monospace font (Consolas or Courier New).
- Syntax highlighting via color coding.
- Split long code across multiple slides with "continued" labels.
- Max 15-20 lines of code per slide for readability.

## 6. Quality Check

- Verify slide count matches template budget.
- Check font sizes are readable.
- Ensure no text overflow or clipping.
- Verify all images are embedded (not linked).

# Integration Points

- **content-generator** (layer 02): Provides the source content.
- **format-rules** (layer 01): Provides presentation format preferences from task-type YAML.
- **progress-tracker** (layer 04): Updates presentation generation status.
- **semester.json**: Source of task metadata.

# Dependencies

`python-pptx`, `matplotlib`, `seaborn`, `Pillow`.
