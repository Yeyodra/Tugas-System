---
name: content-generator
description: >
  Universal YAML-driven content generation engine. Reads a task-type YAML definition,
  resolves each section by type (auto, generate, user_input, code_listing), assembles
  the result into a complete LaTeX document, and saves to the correct output path.
  The generator knows NOTHING about specific task types — it is driven entirely by YAML.
  New task types = new YAML file, zero code changes.
---

# Purpose

The core engine of the Tugas System. Given a task-type YAML and user context (matkul, topic, deadline, etc.), it produces a complete `.tex` file ready for compilation.

**Design principle**: This file is a PROTOCOL, not a task-specific generator. It interprets YAML declarations and dispatches work to specialized layers. Adding a new assignment type means adding a new YAML — never editing this file.

# Input

| Parameter | Source | Required |
|-----------|--------|----------|
| `task_type` | YAML filename (e.g., `analisis-jurnal`) | YES |
| `matkul` | User-provided or from `semester.json` | YES |
| `topic` | User-provided | YES |
| `output_format` | `latex` (default) or `docx` | NO |
| `extra_context` | Dict of additional data (code paths, journal URL, etc.) | NO |

# Output

- Complete `.tex` file at the path resolved from `semester.json` course config
- Backup of any existing file (via `file-safety.md` protocol)
- Generation report (sections generated, skipped, errors)

---

# Architecture

```
User Request + Context
        │
        ▼
┌──────────────────────────────────────────────────────────┐
│  content-generator.md (THIS ENGINE)                      │
│                                                          │
│  Step 1: RESOLVE — Load task-type YAML + semester.json   │
│  Step 2: CONTEXT — Build section context from user data  │
│  Step 3: GENERATE — Process each section by type         │
│    ├── "auto"         → Fill from available data         │
│    ├── "generate"     → AI generate using prompt_hint    │
│    ├── "user_input"   → Prompt user for content          │
│    ├── "code_listing" → Read file, format as lstlisting  │
│    ├── "reference"    → Delegate to reference-finder     │
│    └── "delegate"     → Delegate to specialized layer    │
│  Step 4: ASSEMBLE — Combine sections into LaTeX doc      │
│  Step 5: SAVE — Apply file-safety, write output          │
└──────────────────────────────────────────────────────────┘
```

---

# Step 1: RESOLVE — Load Task-Type YAML

## 1.1 Locate the YAML

Search order:
1. `templates/task-types/custom/{task_type}.yaml` — user-created types
2. `templates/task-types/{task_type}.yaml` — built-in types

If not found → error: "Task type '{task_type}' not found. Use template-creator to define it."

## 1.2 Parse and Validate

Load the YAML and validate against the task-type schema. Required top-level keys:

```yaml
# Minimum valid task-type YAML
name: "analisis-jurnal"
display_name: "Tugas Analisis Jurnal"
description: "Analyze a journal paper and write structured analysis"

document_class: "article"          # or custom class name
paper_size: "a4paper"
font_size: "12pt"
language: "indonesian"

naming_convention: "{matkul}_Analisis-Jurnal_{nama}_{npm}"

cover:
  enabled: true
  template: "gunadarma-default"    # or "custom"
  fields: [matkul, judul_tugas, nama, npm, kelas, dosen, tanggal]

sections: []                       # The core — see Step 3
bibliography:
  enabled: false                   # true if references needed
  style: "thebibliography"        # or "biblatex"
```

## 1.3 Resolve Output Path

From `semester.json`, resolve the output directory:

```
semester.json → courses[matkul].output_dir → {output_dir}/{filename}.tex
```

Apply `naming_convention` from YAML:
- `{matkul}` → course code/name
- `{nama}` → student name (from semester.json `student.nama`)
- `{npm}` → student NPM
- `{topic}` → sanitized topic string
- `{date}` → YYYYMMDD

---

# Step 2: CONTEXT — Build Section Context

Assemble a context object that sections can reference:

```json
{
  "student": {
    "nama": "from semester.json",
    "npm": "from semester.json",
    "kelas": "from semester.json"
  },
  "course": {
    "matkul": "user-provided",
    "dosen": "from semester.json courses[matkul].dosen",
    "semester": "from semester.json"
  },
  "task": {
    "topic": "user-provided",
    "task_type": "from YAML name",
    "judul_tugas": "user-provided or auto-generated"
  },
  "extra": {
    "code_path": "if provided",
    "journal_url": "if provided",
    "journal_data": "if journal-analyzer ran"
  },
  "generated": {}
}
```

The `generated` dict accumulates output from each section — later sections can reference earlier ones.

---

# Step 3: GENERATE — Process Each Section

The YAML `sections` array defines what to generate and how. Each section has:

```yaml
sections:
  - id: "latar_belakang"
    title: "Latar Belakang"
    type: "generate"              # auto | generate | user_input | code_listing | reference | delegate
    required: true
    prompt_hint: "Write background context for {topic} in {matkul} context"
    min_words: 150
    max_words: 500
    latex_env: "section"          # section | subsection | enumerate | verbatim | custom
    
  - id: "kode_program"
    title: "Kode Program"
    type: "code_listing"
    required: true
    source: "{extra.code_path}"
    language: "java"
    latex_env: "lstlisting"
```

## Section Type Handlers

### Type: `auto`

Fill from available data without AI generation. Used for metadata sections.

```yaml
- id: "identitas_paper"
  title: "Identitas Paper"
  type: "auto"
  source: "journal_data"          # key in context.extra or context.generated
  fields: [title, authors, journal, year, doi, url]
  format: "table"                 # table | list | paragraph
```

**Process:**
1. Look up `source` key in context
2. Extract specified `fields`
3. Format according to `format` type
4. If source data missing → fall back to `generate` type with a prompt asking AI to fill

### Type: `generate`

AI-generated content using prompt hints and context.

```yaml
- id: "pembahasan"
  title: "Pembahasan"
  type: "generate"
  prompt_hint: >
    Analyze the methodology and results of the paper about {topic}.
    Discuss strengths, weaknesses, and relevance to {matkul}.
    Write in formal Indonesian academic style.
  min_words: 200
  max_words: 800
  context_refs: ["latar_belakang", "identitas_paper"]
```

**Process:**
1. Resolve `{variables}` in `prompt_hint` from context
2. If `context_refs` specified, include output of those sections as additional context
3. Generate content with constraints:
   - Language: from YAML `language` field
   - Word count: between `min_words` and `max_words`
   - Style: formal academic (default) or as specified
4. Run through `plagiarism-guard.md` quick check (AI slop detection only, not full check)
5. Store result in `context.generated[section.id]`

**Generation guidelines:**
- Write in the language specified by the YAML (default: Indonesian)
- Use formal academic register — no slang, no abbreviations
- Avoid AI-tell patterns (see `ai-self-audit.md` patterns A1-A12, B1-B4)
- Each paragraph should have a clear topic sentence
- Vary sentence length and structure naturally

### Type: `user_input`

Prompt the user to provide content. Used for sections only the student can write.

```yaml
- id: "kesimpulan_pribadi"
  title: "Kesimpulan Pribadi"
  type: "user_input"
  prompt: "Tulis kesimpulan pribadi kamu tentang paper ini (min 100 kata):"
  min_words: 100
  fallback: "generate"            # if user says "generate for me"
```

**Process:**
1. Display `prompt` to user
2. Wait for user input
3. If user provides text → use as-is (with minimal formatting cleanup)
4. If user says "generate" or "skip" and `fallback` is set → switch to that type
5. If no fallback and user skips → mark section as `[PLACEHOLDER — isi manual]`

### Type: `code_listing`

Read source code file and format as LaTeX listing.

```yaml
- id: "source_code"
  title: "Source Code"
  type: "code_listing"
  source: "{extra.code_path}"     # resolved from context
  language: "python"
  line_numbers: true
  caption: "Implementasi {topic}"
  max_lines: 100                  # truncate if longer
  sections_to_include: []         # empty = full file; or specify function/class names
```

**Process:**
1. Resolve `source` path from context
2. Read the file
3. If `sections_to_include` specified → extract only those functions/classes
4. If file exceeds `max_lines` → truncate with `% ... (truncated)` comment
5. Format as `lstlisting` environment:

```latex
\begin{lstlisting}[language=Python, caption={Implementasi Topic}, label={lst:source_code}, numbers=left]
# code here
\end{lstlisting}
```

6. Optionally delegate to `code-analyzer.md` for explanation generation

### Type: `reference`

Delegate to `reference-finder.md` to find and format references.

```yaml
- id: "daftar_pustaka"
  title: "Daftar Pustaka"
  type: "reference"
  count: 5
  topic: "{task.topic}"
  max_age_years: 5
  format: "thebibliography"       # or "biblatex"
```

**Process:**
1. Call `reference-finder.md` with topic and count
2. Receive formatted bibliography entries
3. Wrap in appropriate LaTeX environment

### Type: `delegate`

Delegate to a specialized layer for complex generation.

```yaml
- id: "analisis_jurnal"
  title: "Analisis Jurnal"
  type: "delegate"
  delegate_to: "journal-analyzer"
  delegate_params:
    topic: "{task.topic}"
    journal_url: "{extra.journal_url}"
  output_sections: ["latar_belakang", "metode", "pembahasan"]
```

**Process:**
1. Call the specified layer with `delegate_params`
2. Receive structured output (multiple sub-sections)
3. Map `output_sections` to the generated content
4. Each sub-section becomes a `\subsection{}` under this section

---

# Step 4: ASSEMBLE — Combine into LaTeX Document

## 4.1 Document Preamble

```latex
\documentclass[{font_size},{paper_size}]{article}

% === Packages ===
\usepackage[indonesian]{babel}
\usepackage[utf8]{inputenc}
\usepackage{geometry}
\usepackage{graphicx}
\usepackage{hyperref}
\usepackage{listings}
\usepackage{xcolor}
\usepackage{float}
\usepackage{booktabs}

% === Geometry ===
\geometry{
  a4paper,
  top=3cm, bottom=3cm,
  left=4cm, right=3cm
}

% === Listings Config ===
\lstset{
  basicstyle=\ttfamily\small,
  breaklines=true,
  frame=single,
  numbers=left,
  numberstyle=\tiny\color{gray},
  keywordstyle=\color{blue},
  commentstyle=\color{green!60!black},
  stringstyle=\color{red!70!black},
  tabsize=4
}
```

If the YAML specifies a custom `document_class` (e.g., `gunadarma-pi`), use that instead of `article` and skip redundant package declarations.

## 4.2 Cover Page

If `cover.enabled` is true:

```latex
\begin{titlepage}
\begin{center}
  % Logo if specified
  \includegraphics[width=3cm]{logo-gunadarma.png}\\[0.5cm]
  
  {\large\textbf{UNIVERSITAS GUNADARMA}}\\
  {\large FAKULTAS ILMU KOMPUTER DAN TEKNOLOGI INFORMASI}\\[1cm]
  
  {\Large\textbf{{judul_tugas}}}\\[0.5cm]
  {\large {matkul}}\\[2cm]
  
  Disusun oleh:\\[0.3cm]
  \begin{tabular}{ll}
    Nama  & : {nama} \\
    NPM   & : {npm} \\
    Kelas & : {kelas} \\
  \end{tabular}\\[2cm]
  
  Dosen: {dosen}\\[1cm]
  {tanggal}
\end{center}
\end{titlepage}
```

Cover fields are resolved from context. The template can be customized per YAML.

## 4.3 Section Assembly

For each section in order:

```latex
\section{Section Title}
% Generated content here

\section{Next Section Title}
% More content
```

Respect `latex_env` from YAML:
- `section` → `\section{}`
- `subsection` → `\subsection{}`
- `enumerate` → wrap in `\begin{enumerate}...\end{enumerate}`
- `verbatim` → wrap in `\begin{verbatim}...\end{verbatim}`
- `lstlisting` → wrap in `\begin{lstlisting}...\end{lstlisting}`
- `none` → no wrapper (raw content injection)

## 4.4 Bibliography

If `bibliography.enabled`:

```latex
\begin{thebibliography}{99}
  \bibitem{key1} Author. Year. Title. \textit{Journal}. Vol. X, hal. Y.
  % ... more entries
\end{thebibliography}
```

## 4.5 Final Assembly Order

```
1. \documentclass + preamble
2. \begin{document}
3. Cover page (if enabled)
4. \tableofcontents (if enabled in YAML)
5. Sections in YAML order
6. Bibliography (if enabled)
7. \end{document}
```

---

# Step 5: SAVE — Apply File Safety and Write

## 5.1 Apply File Safety Protocol

Before writing, follow `layers/04-output/file-safety.md`:
1. Check if target file exists
2. If yes → create timestamped backup
3. Enforce max 5 backups per file

## 5.2 Write Output

Write the assembled `.tex` content to the resolved output path.

## 5.3 Generation Report

After saving, produce a summary:

```
✓ Generated: {filename}.tex
  Task type: {task_type}
  Sections: {n} generated, {m} user-input, {k} skipped
  Word count: ~{total_words}
  Output: {output_path}
  Backup: {backup_path or "none (new file)"}
  
  Sections:
    ✓ cover          — auto
    ✓ latar_belakang — generated (287 words)
    ✓ identitas      — auto (from journal data)
    ✓ pembahasan     — generated (412 words)
    ⚠ kesimpulan     — placeholder (user_input skipped)
    ✓ daftar_pustaka — 5 references found
```

---

# Error Handling

| Error | Action |
|-------|--------|
| YAML not found | Error message + suggest `template-creator` |
| YAML parse error | Show line number + validation error |
| Section source missing (auto/code_listing) | Fall back to `generate` type with warning |
| Code file not found | Skip section with `[FILE NOT FOUND: {path}]` placeholder |
| Reference finder returns 0 results | Include empty bibliography with comment |
| AI generation fails | Retry once; if still fails, insert placeholder |
| Output path not writable | Error message + suggest alternative path |

---

# Extensibility Contract

To add a new task type, create a YAML file with:

1. **Metadata**: name, display_name, description
2. **Document settings**: document_class, paper_size, font_size, language
3. **Naming convention**: template string for output filename
4. **Cover config**: enabled, template, fields
5. **Sections array**: ordered list of sections with type + config
6. **Bibliography config**: enabled, style, count

The content-generator will handle the rest. No code changes needed.

## Example: Adding "Review Paper" task type

```yaml
name: "review-paper"
display_name: "Tugas Review Paper"
description: "Critical review of a research paper"
document_class: "article"
paper_size: "a4paper"
font_size: "12pt"
language: "indonesian"
naming_convention: "{matkul}_Review-Paper_{nama}_{npm}"

cover:
  enabled: true
  template: "gunadarma-default"
  fields: [matkul, judul_tugas, nama, npm, kelas, dosen, tanggal]

sections:
  - id: "identitas_paper"
    title: "Identitas Paper"
    type: "auto"
    source: "journal_data"
    fields: [title, authors, journal, year, doi]
    format: "table"

  - id: "ringkasan"
    title: "Ringkasan"
    type: "generate"
    prompt_hint: >
      Summarize the paper about {topic}. Cover: research objective,
      methodology, key findings, and contribution. 
      Write in formal Indonesian. Max 2 paragraphs.
    min_words: 150
    max_words: 400

  - id: "critical_review"
    title: "Critical Review"
    type: "generate"
    prompt_hint: >
      Provide critical analysis of the paper. Discuss:
      1. Strengths of methodology and findings
      2. Weaknesses or limitations
      3. Suggestions for improvement
      4. Relevance to {matkul}
    min_words: 300
    max_words: 800
    context_refs: ["identitas_paper", "ringkasan"]

  - id: "kesimpulan"
    title: "Kesimpulan"
    type: "generate"
    prompt_hint: >
      Write conclusion summarizing the review. Include personal
      assessment of the paper's contribution to the field.
    min_words: 100
    max_words: 300

  - id: "daftar_pustaka"
    title: "Daftar Pustaka"
    type: "reference"
    count: 1
    topic: "{task.topic}"
    format: "thebibliography"

bibliography:
  enabled: true
  style: "thebibliography"
```

---

# Integration Points

| Layer | Relationship |
|-------|-------------|
| `journal-analyzer.md` | Delegated to for `delegate_to: journal-analyzer` sections |
| `code-analyzer.md` | Delegated to for code explanation generation |
| `reference-finder.md` | Called for `type: reference` sections |
| `template-creator.md` | Creates the YAML files this engine consumes |
| `paraphraser.md` | Post-processes generated text if paraphrase requested |
| `plagiarism-guard.md` | Quick AI-slop check on generated sections |
| `file-safety.md` | Backup protocol before any file write |
| `ai-self-audit.md` | Optional full-document audit after generation |
| `table-generator.md` | Called when section format is "table" with complex data |
| `diagram-generator.md` | Called when section type involves diagram generation |
| `figure-generator.md` | Called when section needs chart/figure generation |
