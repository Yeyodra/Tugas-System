---
layer: 04-output
type: skill
name: latex-compiler
---

# Purpose

Compiles tugas (assignment) LaTeX source files into PDF documents. Handles class file and asset copying, compilation, and error reporting.

# Input

- Path to the main `.tex` file for the assignment.
- Task metadata from `semester.json` (judul, nama, npm, kelas, matakuliah, dosen, tahun).
- `gunadarma-tugas.cls` class file (from `templates/latex/`).

# Output

- Compiled PDF document.
- Compilation log (for troubleshooting).

# Process

## 1. Environment Preparation

Before compilation, prepare the workspace:

1. **Locate source**: Find the `.tex` file to compile. Resolve from:
   - Explicit path argument, OR
   - `semester.json → tasks[active] → paths.tex_file`

2. **Copy class file**: Check if `gunadarma-tugas.cls` exists in the same directory as the `.tex` file. If not, copy it from `templates/latex/gunadarma-tugas.cls`.

3. **Copy logo**: Check if `logo-gunadarma.png` exists in the same directory. If not, copy from `templates/latex/logo-gunadarma.png`.

4. **Verify images**: Scan the `.tex` file for `\includegraphics{...}` references. Warn if any referenced image files are missing.

## 2. Metadata Injection

If the `.tex` file uses placeholder commands (`\judul{}`, `\nama{}`, etc.) with empty values, offer to populate them from `semester.json`:

| Command | Source |
|---------|--------|
| `\judul{...}` | semester.json → tasks[active] → judul |
| `\nama{...}` | semester.json → student.nama |
| `\npm{...}` | semester.json → student.npm |
| `\kelas{...}` | semester.json → student.kelas |
| `\matakuliah{...}` | semester.json → tasks[active] → matakuliah |
| `\dosen{...}` | semester.json → tasks[active] → dosen |
| `\tahun{...}` | semester.json → tahun |

Do NOT overwrite values that are already filled in the `.tex` file.

## 3. Compilation

Execute in the directory containing the `.tex` file:

```bash
pdflatex -interaction=nonstopmode {filename}.tex
pdflatex -interaction=nonstopmode {filename}.tex
```

Two passes ensure cross-references, TOC, and listings resolve correctly.

If the document uses bibliography (`\cite` commands):
```bash
pdflatex -interaction=nonstopmode {filename}.tex
bibtex {filename}
pdflatex -interaction=nonstopmode {filename}.tex
pdflatex -interaction=nonstopmode {filename}.tex
```

## 4. Post-Compilation

- **Success**: Report the output PDF path and page count.
- **Cleanup (optional)**: Remove auxiliary files (`.aux`, `.log`, `.toc`, `.out`, `.lof`, `.lot`). Keep them if user wants to debug.
- **Error**: Parse the `.log` file for errors. Report the first error with line number and context.

## 5. Troubleshooting

| Problem | Solution |
|---------|----------|
| Package not found | Install via `tlmgr install {package}` or MiKTeX console |
| Logo missing | Copy `logo-gunadarma.png` from `templates/latex/` |
| Overfull hbox | Check for long unbreakable text; use `\sloppy` if needed |
| Font not found | Ensure `newtxtext` package is installed (`tlmgr install newtx`) |
| cls not found | Copy `gunadarma-tugas.cls` to the same directory as the `.tex` file |

# Integration Points

- **format-rules** (layer 01): Provides format configuration to select correct cls.
- **content-generator** (layer 02): Produces the `.tex` source files.
- **progress-tracker** (layer 04): Updates compilation status in `semester.json`.
- **Templates**: `templates/latex/gunadarma-tugas.cls` and `templates/latex/logo-gunadarma.png`.
