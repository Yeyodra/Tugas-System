# Setup Manual — Tugas System

Step-by-step guide for setting up Tugas System on your machine.

---

## Prerequisites

Before starting, make sure you have:

| Tool | Required? | Check with | Install from |
|------|-----------|------------|--------------|
| LaTeX (MiKTeX or TeX Live) | Yes | `pdflatex --version` | [miktex.org](https://miktex.org) or [tug.org/texlive](https://tug.org/texlive/) |
| Pandoc | Optional (for DOCX) | `pandoc --version` | [pandoc.org](https://pandoc.org/installing.html) |
| Git | Yes | `git --version` | [git-scm.com](https://git-scm.com) |
| AI coding assistant | Yes | — | Claude Code, OpenCode, Cursor, etc. |

---

## Step 1: Clone the Repository

```bash
git clone https://github.com/Yeyodra/Tugas-System.git
```

Put it somewhere permanent. The AI agent will reference this path every time you use the system.

Example locations:
- Windows: `C:\Users\{you}\Documents\Projek\Tugas-System`
- macOS/Linux: `~/projects/tugas-system`

Remember this path. You'll need it in Step 6.

---

## Step 2: Configure semester.json

Open `config/semester.json` and update it with your info:

```json
{
  "semester": "S6",
  "tahun_ajaran": "2025/2026 Genap",
  "kelas": "3IA21",
  "base_path": "C:\\Users\\{you}\\Documents\\Kampus\\S6",
  "courses": {
    "lab": {
      "PC": {
        "full_name": "Pengolahan Citra",
        "path": "LAB/PC"
      }
    },
    "matkul": {
      "PC": {
        "full_name": "Pengolahan Citra",
        "path": "MATKUL/Pengolahan Citra"
      },
      "KDM": {
        "full_name": "Konsep Data Mining",
        "path": "MATKUL/Konsep Data Mining"
      }
    }
  }
}
```

**What to change:**
- `semester` — your current semester (e.g., "S5", "S6")
- `tahun_ajaran` — academic year
- `kelas` — your class code
- `base_path` — where assignment output goes (use `\\` on Windows)
- `courses.lab` — your lab courses with abbreviations and full names
- `courses.matkul` — your lecture courses with abbreviations and full names

The `path` field under each course is relative to `base_path`. So `"path": "MATKUL/Pengolahan Citra"` means output goes to `{base_path}/MATKUL/Pengolahan Citra/`.

---

## Step 3: Configure defaults.json

Open `config/defaults.json` and update your personal info:

```json
{
  "user": {
    "nama": "Your Full Name",
    "npm": "12345678",
    "kelas": "3IA21"
  },
  "university": {
    "name": "Universitas Gunadarma",
    "faculty": "Teknologi Industri",
    "program": "Informatika"
  }
}
```

If you're not at Gunadarma, update the `university` section. You may also need to modify the LaTeX templates in `templates/latex/` to match your university's format.

---

## Step 4: Create Folder Structure

Create the output folder tree that matches your `semester.json`:

```
{base_path}/
├── LAB/
│   ├── PC/
│   ├── SBD2/
│   └── ...
└── MATKUL/
    ├── Pengolahan Citra/
    ├── Konsep Data Mining/
    └── ...
```

The system auto-creates subfolders for individual assignments, but the top-level course folders should exist.

On Windows:
```powershell
$base = "C:\Users\{you}\Documents\Kampus\S6"
mkdir "$base\LAB\PC", "$base\LAB\SBD2" -Force
mkdir "$base\MATKUL\Pengolahan Citra", "$base\MATKUL\Konsep Data Mining" -Force
```

On macOS/Linux:
```bash
base=~/Documents/Kampus/S6
mkdir -p "$base/LAB/PC" "$base/LAB/SBD2"
mkdir -p "$base/MATKUL/Pengolahan Citra" "$base/MATKUL/Konsep Data Mining"
```

---

## Step 5: Connect Your AI Assistant

Add this rule to your AI assistant's configuration (AGENTS.md, system prompt, or equivalent):

```markdown
When user mentions "tugas", "buat tugas", "kerjain tugas", "analisis jurnal",
"buat makalah", "buat presentasi", "buat resume", "buat LA", or any
university assignment context:

1. Read {TUGAS_SYSTEM_PATH}/SKILL.md
2. Read {TUGAS_SYSTEM_PATH}/config/semester.json
3. Follow the orchestrator routing in SKILL.md
```

Replace `{TUGAS_SYSTEM_PATH}` with the actual path from Step 1.

**How this works:** The AI reads `SKILL.md` (the orchestrator) which tells it how to route your request, which templates to use, and where to save output. `semester.json` gives it your course list and folder paths.

---

## Step 6: Test It

Tell your AI assistant:

```
buat analisis jurnal PC
```

It should:
1. Load the orchestrator and semester config
2. Match "analisis jurnal" to the `analisis-jurnal.yaml` template
3. Match "PC" to "Pengolahan Citra" from your courses
4. Generate the assignment content
5. Save output to `{base_path}/MATKUL/Pengolahan Citra/Analisis Jurnal 1/`

If the output lands in the right folder with the right format, you're done.

---

## Next Semester

When a new semester starts:

1. Update `config/semester.json` — change semester number, courses, and `base_path`
2. Create the new folder structure (Step 4)
3. Done. Everything else adapts automatically.

You don't need to touch `defaults.json` unless your personal info changes.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| AI doesn't recognize "buat tugas" | Check that SKILL.md path is correct in your agent config |
| Course abbreviation not found | Make sure it's in `semester.json` under `courses.lab` or `courses.matkul` |
| PDF compilation fails | Run `pdflatex --version` to verify LaTeX is installed |
| DOCX conversion fails | Run `pandoc --version` to verify Pandoc is installed |
| Output goes to wrong folder | Check `base_path` and course `path` values in `semester.json` |
| Template not found | Check `templates/task-types/_defaults/` for built-in templates |
