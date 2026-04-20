# Tugas System

AI-powered assignment assistant for Indonesian university students. YAML-driven, extensible, works with any AI coding assistant.

## What is this?

Tugas System is an orchestration layer that helps university students generate, format, and compile coursework assignments. You describe what you need in natural language ("buat analisis jurnal PC"), and the system handles everything: finding references, generating content, paraphrasing, formatting to university standards, and compiling to PDF/DOCX/PPTX. Built for Universitas Gunadarma, but adaptable to any Indonesian university.

This is **not** a thesis/skripsi tool. For that, see [PI System](https://github.com/Yeyodra/PI.git).

## Features

- **YAML-driven task types** — analisis jurnal, makalah, presentasi, resume, plus custom templates
- **Laporan Akhir (LA) generator** — praktikum reports with Gunadarma formatting
- **Smart output routing** — files go to `MATKUL/{course}/{task}/` or `LAB/{course}/M{N}/` automatically
- **AI self-audit** — detects AI-generated patterns before submission
- **Academic paraphraser** — 6 NLP phenomena, not just synonym swapping
- **Template creator** — describe your format, get a YAML template generated
- **Multi-format output** — LaTeX (PDF), DOCX, PPTX
- **Semester-aware** — auto-detects courses from abbreviations (e.g., "PC" resolves to "Pengolahan Citra")
- **Natural language commands** — "buat makalah KDM tentang clustering" just works

## Quick Start

1. Clone the repo and configure `config/semester.json` with your courses
2. Point your AI assistant to `SKILL.md` (see [Setup Guide](docs/SETUP-MANUAL.md))
3. Say: `buat analisis jurnal PC`

For detailed setup, see [SETUP-MANUAL.md](docs/SETUP-MANUAL.md) or paste [SETUP-AGENT.md](docs/SETUP-AGENT.md) to your AI assistant for automatic setup.

## Architecture

```
Tugas-System/
├── SKILL.md                    # Orchestrator (entry point for AI agents)
├── config/
│   ├── defaults.json           # System config (user info, paths, output settings)
│   └── semester.json           # Current semester (courses, base_path)
├── templates/
│   ├── task-types/
│   │   ├── _defaults/          # Built-in: analisis-jurnal, makalah, presentasi, resume
│   │   └── custom/             # Your custom templates
│   └── latex/                  # LaTeX classes (.cls) + university logo
├── layers/
│   ├── 01-knowledge/           # Reference search, format rules
│   ├── 02-generate/            # Content gen, paraphraser, diagrams, LA generator
│   ├── 03-quality/             # AI audit, format checker, plagiarism guard
│   └── 04-output/              # LaTeX compiler, DOCX converter, PPTX generator
├── pipelines/                  # tugas-pipeline.md, revision-pipeline.md
├── shared/schemas/             # JSON schemas for template validation
└── docs/                       # Guides and setup docs
```

35 files total. No runtime dependencies beyond LaTeX and your AI assistant.

## Task Types

| Template | Description | Output |
|----------|-------------|--------|
| `analisis-jurnal` | Find and analyze a scientific journal | PDF (LaTeX) |
| `makalah` | Research paper with 5-15 references | PDF (LaTeX) |
| `presentasi` | Slide deck from topic or existing document | PPTX |
| `resume` | 1-2 page summary, no cover page | PDF (LaTeX) |
| *custom* | Define your own via template creator | Any |

## Creating Custom Templates

Use menu 10 (template creator) or manually create a YAML file:

```yaml
task_type: "essay"
display_name: "Essay"
description: "Short essay assignment"
category: "matkul"

sections:
  - name: "Pendahuluan"
    type: "generate"
    prompt_hint: "Write introduction for the essay topic"
  - name: "Pembahasan"
    type: "generate"
    prompt_hint: "Main discussion"
  - name: "Kesimpulan"
    type: "generate"
    prompt_hint: "Conclusion"

format:
  output: "latex"

generation:
  needs_paraphrase: true
  language: "id"
```

Save to `templates/task-types/custom/essay.yaml` and it's immediately available.

## Requirements

- **LaTeX** — MiKTeX or TeX Live (for PDF compilation)
- **Pandoc** — optional, for DOCX conversion
- **AI coding assistant** — Claude Code, OpenCode, Cursor, or similar

## Related

- [PI System](https://github.com/Yeyodra/PI.git) — for skripsi / Penelitian Ilmiah (separate system, different workflow)

## License

MIT
