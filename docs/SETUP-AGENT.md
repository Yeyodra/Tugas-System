# Tugas System — Auto Setup

**Paste this entire file to your AI coding assistant.** It will set up the Tugas System for you automatically.

---

## Instructions for AI Agent

You are setting up the **Tugas System** for a new user. This is an AI-powered assignment assistant for Indonesian university students. Follow the steps below.

### Step 1: Gather User Info

Ask the user for the following (in Indonesian if they prefer):

- **Nama lengkap** (full name)
- **NPM** (student ID number)
- **Kelas** (class code, e.g., "3IA21")
- **Semester** ke berapa (e.g., "S6")
- **Tahun ajaran** (e.g., "2025/2026 Genap")
- **Universitas, fakultas, program studi** (default: Universitas Gunadarma, Teknologi Industri, Informatika)
- **Daftar mata kuliah** — both lab and lecture courses. For each course, get:
  - Abbreviation (e.g., "PC", "KDM", "SBD2")
  - Full name (e.g., "Pengolahan Citra", "Konsep Data Mining")
  - Whether it's a lab course, lecture course, or both
- **Base path** — where assignment output should go (e.g., `C:\Users\{name}\Documents\Kampus\S6` on Windows, `~/Documents/Kampus/S6` on macOS/Linux)

If the user has a photo of their class schedule, parse it to extract course names and abbreviations.

### Step 2: Clone the Repository

```bash
git clone https://github.com/Yeyodra/Tugas-System.git {install_path}
```

Ask the user where they want to install it, or suggest a sensible default:
- Windows: `C:\Users\{name}\Documents\Projek\Tugas-System`
- macOS/Linux: `~/projects/tugas-system`

### Step 3: Generate semester.json

Create `config/semester.json` from the gathered info. Follow this structure exactly:

```json
{
  "semester": "{semester}",
  "tahun_ajaran": "{tahun_ajaran}",
  "kelas": "{kelas}",
  "base_path": "{base_path}",
  "courses": {
    "lab": {
      "{ABBREV}": {
        "full_name": "{Full Course Name}",
        "path": "LAB/{ABBREV}"
      }
    },
    "matkul": {
      "{ABBREV}": {
        "full_name": "{Full Course Name}",
        "path": "MATKUL/{Full Course Name}"
      }
    }
  }
}
```

Rules:
- Lab course paths use the abbreviation: `LAB/PC`, `LAB/SBD2`
- Matkul course paths use the full name: `MATKUL/Pengolahan Citra`
- If a course appears in both lab and matkul, add it to both sections
- Use `\\` for Windows paths in `base_path`, or `/` for macOS/Linux

### Step 4: Update defaults.json

Update `config/defaults.json` with the user's info:

```json
{
  "system": {
    "name": "Tugas System",
    "version": "1.0.0",
    "description": "Asisten AI untuk tugas kuliah sehari-hari"
  },
  "user": {
    "nama": "{nama}",
    "npm": "{npm}",
    "kelas": "{kelas}"
  },
  "university": {
    "name": "{university}",
    "faculty": "{faculty}",
    "program": "{program}"
  },
  "paths": {
    "semester_config": "config/semester.json",
    "task_types_default": "templates/task-types/_defaults",
    "task_types_custom": "templates/task-types/custom",
    "latex_templates": "templates/latex",
    "logo": "templates/latex/logo-gunadarma.png"
  },
  "output": {
    "default_format": "latex",
    "compiler": "pdflatex",
    "backup": {
      "enabled": true,
      "max_backups_per_file": 5
    }
  },
  "tools": {
    "laporan_praktikum": "layers/02-generate/laporan-praktikum.md",
    "laporan_praktikum_cls": "templates/latex/gunadarma-la.cls"
  }
}
```

If the user is not at Universitas Gunadarma, update the university section and note that they may need to customize the LaTeX templates in `templates/latex/`.

### Step 5: Create Folder Structure

Create the output directory tree based on `base_path` and courses:

```
{base_path}/
├── LAB/
│   ├── {lab_course_1_abbrev}/
│   ├── {lab_course_2_abbrev}/
│   └── ...
└── MATKUL/
    ├── {matkul_course_1_full_name}/
    ├── {matkul_course_2_full_name}/
    └── ...
```

Create all directories. The system auto-creates assignment subfolders later, but the course-level folders must exist.

### Step 6: Register in Agent Config

Add the following auto-detection rule to the user's AI agent configuration. This could be `AGENTS.md`, a system prompt, `.cursorrules`, or whatever their AI tool uses:

```markdown
- **Tugas System Auto-Detection:** When user mentions "tugas", "buat tugas", "kerjain tugas",
  "analisis jurnal", "buat makalah", "buat presentasi", "buat resume", "buat LA",
  "laporan akhir", "laporan praktikum", or any university assignment context:
  1. Read {TUGAS_SYSTEM_PATH}/SKILL.md
  2. Read {TUGAS_SYSTEM_PATH}/config/semester.json
  3. Follow the orchestrator routing in SKILL.md
  4. Auto-detect course from context: "tugas PC" resolves to Pengolahan Citra, etc.
  5. Auto-detect task type: "analisis jurnal" maps to analisis-jurnal.yaml, etc.
```

Replace `{TUGAS_SYSTEM_PATH}` with the actual install path.

### Step 7: Verify Prerequisites

Check that required tools are installed:

```bash
pdflatex --version
```

If LaTeX is not installed, tell the user to install MiKTeX (https://miktex.org) or TeX Live (https://tug.org/texlive/).

Optionally check Pandoc (needed only for DOCX output):

```bash
pandoc --version
```

### Step 8: Test

Run a quick test by simulating a user command. Read the orchestrator (`SKILL.md`) and semester config, then verify:

1. Course abbreviation "PC" resolves to "Pengolahan Citra"
2. Task type "analisis jurnal" resolves to `templates/task-types/_defaults/analisis-jurnal.yaml`
3. Output path resolves to `{base_path}/MATKUL/Pengolahan Citra/`

If all three resolve correctly, the system is ready.

### Step 9: Report to User

Tell the user what was set up. Use this format:

```
Tugas System setup complete.

Install path:  {install_path}
Semester:      {semester} ({tahun_ajaran})
Mahasiswa:     {nama} — {npm} — {kelas}
Universitas:   {university}

Courses configured:
  Lab:    {list of lab courses}
  Matkul: {list of matkul courses}

Output folder: {base_path}

How to use:
  "buat analisis jurnal PC"
  "buat makalah KDM tentang clustering"
  "buat LA PC modul 2"
  "buat presentasi RPL1"

Or just say "tugas" to see the interactive menu.
```

---

## Notes for the AI Agent

- The orchestrator is `SKILL.md` at the repo root. Always read it first when the user triggers the system.
- `semester.json` changes every semester. `defaults.json` rarely changes.
- The system ships with 4 default task templates: analisis-jurnal, makalah, presentasi, resume. Users can create more via the template creator (menu 10).
- Laporan Akhir (LA) uses a separate embedded layer, not the main tugas pipeline.
- This system is separate from the PI System (skripsi/thesis). Never load `pi-project.json` from this context.
