# Tugas Pipeline

4-stage pipeline untuk mengerjakan tugas kuliah dari awal sampai output final.

---

## Stage 1: RESOLVE

Identifikasi dan persiapan sebelum generate.

```
Input  : perintah user (e.g. "buat analisis jurnal PC")
Output : resolved task config
```

**Steps:**

1. **Match task type** — Cocokkan ke YAML template di `templates/task-types/`
   - Cek `_defaults/` dulu, lalu `custom/`
   - Jika tidak ada match → tawarkan buat template baru via `template-creator.md`
2. **Identify course** — Cocokkan mata kuliah dari `semester.json`
   - Resolve alias (e.g. "PC" → "Pengantar Komputer")
   - Ambil info dosen, kode matkul, semester
3. **Determine output path** — Bangun path dari semester config
   - Pattern: `{semester_dir}/{course}/{subfolder_pattern}/`
4. **Load format rules** — Baca `format` block dari YAML template
   - Document class, margins, font, spacing, etc.
5. **Resolve user overrides** — Jika user kasih instruksi spesifik, override YAML defaults

**Output artifact:** `task-config` object berisi semua info untuk Stage 2.

---

## Stage 2: GENERATE

Buat konten berdasarkan YAML spec.

```
Input  : task-config dari Stage 1
Output : generated content per section
```

**Steps:**

1. **Execute pre_steps** — Jalankan step dari YAML `generation.pre_steps`
   - `search_journal` → cari jurnal via web search, validasi DOI
   - `search_references` → cari referensi akademik
   - Simpan hasil search sebagai context
2. **Generate per section** — Iterasi `sections` dari YAML
   - `type: "auto"` → generate otomatis (metadata, daftar pustaka, cover)
   - `type: "generate"` → generate konten pakai `prompt_hint` + context
   - `type: "group"` → iterasi children
3. **Apply paraphrase** — Jika `needs_paraphrase: true`
   - Jalankan `layers/02-generate/paraphraser.md`
   - Target: bahasa akademik natural, bukan terjemahan mentah
4. **Generate visual assets** — Jika konten butuh diagram/tabel/figure
   - `layers/02-generate/diagram-generator.md`
   - `layers/02-generate/table-generator.md`
   - `layers/02-generate/figure-generator.md`

**Output artifact:** complete content per section, siap compile.

---

## Stage 3: CHECK

Quality check ringan. **Semua advisory — tidak ada hard gate.**

```
Input  : generated content
Output : validation report (advisory)
```

**Steps:**

1. **Format check** — Jika `quality.check_format: true`
   - Cek margins, font, spacing sesuai YAML `format` block
   - Cek struktur section sesuai YAML `sections`
2. **AI self-audit** — Jika `quality.check_ai_pattern: true`
   - Jalankan `layers/03-quality/ai-self-audit.md`
   - Deteksi pola bahasa AI yang terlalu obvious
   - Sarankan perbaikan (TIDAK auto-fix)
3. **Plagiarism guard** — Jika `quality.check_plagiarism: true`
   - Jalankan `layers/03-quality/plagiarism-guard.md`
   - Cek similarity dengan sumber asli

**Output artifact:** validation report (JSON per `shared/schemas/validation-report.json`).
Report ditampilkan ke user sebagai info, bukan blocker.

---

## Stage 4: OUTPUT

Compile dan simpan file final.

```
Input  : content + validation report
Output : compiled file (PDF/DOCX/PPTX)
```

**Steps:**

1. **Backup existing** — Jalankan `layers/04-output/file-safety.md`
   - Jika file sudah ada, backup ke `_backup/` dengan timestamp
2. **Create subfolder** — Buat folder output di course directory
   - Pattern dari YAML `naming.subfolder_pattern`
3. **Copy template files** — Salin `.cls`, logo, assets ke output folder
   - Dari `templates/latex/` atau `templates/docx/`
4. **Compile** — Compile ke target format
   - LaTeX → PDF (via `latexmk` atau `xelatex`)
   - Markdown → DOCX (via `pandoc`)
   - Content → PPTX (via python-pptx atau skill)
5. **Update progress** — Update `semester.json` status tugas
   - Mark task as completed, record timestamp

**Output artifact:** final file di output path + updated semester.json.

---

## Flow Diagram

```
User Command
    │
    ▼
┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────────┐
│ RESOLVE  │───▶│ GENERATE │───▶│  CHECK  │───▶│  OUTPUT  │
│          │    │          │    │(advisory)│    │          │
└─────────┘    └──────────┘    └─────────┘    └──────────┘
 match YAML     pre_steps       format chk     backup
 find course    gen sections    ai audit       compile
 build path     paraphrase      plag guard     save file
 load format    visuals                        update json
```

---

## Error Handling

| Error | Action |
|-------|--------|
| No YAML match | Tawarkan buat template baru |
| Course not found | Tanya user, suggest closest match |
| Pre-step gagal (search) | Lanjut tanpa search, minta user kasih sumber manual |
| Compile gagal | Tampilkan error, simpan source file (.tex/.md) |
| File conflict | Auto-backup via file-safety protocol |
