# Panduan Tugas System

Sistem otomatis untuk mengerjakan tugas kuliah: analisis jurnal, makalah, presentasi, resume, dan lainnya.

---

## Apa Itu Tugas System?

Tugas System ≠ PI System. Perbedaan utama:

| | Tugas System | PI System |
|---|---|---|
| Untuk | Tugas kuliah harian | Skripsi / Penelitian Ilmiah |
| Kompleksitas | Rendah-sedang | Tinggi |
| Quality gate | Advisory (tidak blocking) | Mandatory (harus pass) |
| Peer review | Tidak ada | Multi-round review |
| Output | PDF, DOCX, PPTX | LaTeX → PDF |

---

## Struktur Folder

```
Tugas-System/
├── config/              # semester.json, user config
├── templates/
│   ├── task-types/
│   │   ├── _defaults/   # template bawaan (4 tipe)
│   │   └── custom/      # template buatan user
│   ├── latex/           # .cls, logo
│   └── docx/            # template Word
├── layers/              # modul processing
│   ├── 01-knowledge/    # search & retrieval
│   ├── 02-generate/     # paraphraser, diagram, tabel, figure
│   ├── 03-quality/      # ai-self-audit, plagiarism-guard
│   └── 04-output/       # file-safety, compile
├── pipelines/           # tugas-pipeline, revision-pipeline
├── shared/schemas/      # JSON schemas
└── docs/                # panduan ini
```

---

## Cara Pakai

### 1. Analisis Jurnal

```
"buat analisis jurnal PC"
"buat analisis jurnal Pengantar Komputer tentang cloud computing"
```

Sistem akan: cari jurnal → analisis per section → compile PDF dengan cover.

### 2. Makalah

```
"buat makalah KDM tentang clustering"
"buat makalah Struktur Data tentang binary tree, minimal 10 referensi"
```

Sistem akan: cari 5-15 referensi → generate per bab → paraphrase → compile PDF.

### 3. Presentasi

```
"buat presentasi RPL1"
"buat presentasi dari makalah KDM yang tadi"
```

Sistem akan: generate slide dari topik atau dokumen yang sudah ada → output PPTX.

### 4. Resume

```
"buat resume materi TBO pertemuan 3"
"buat resume tentang finite automata"
```

Sistem akan: generate ringkasan 1-2 halaman → format simpel tanpa cover.

### 5. Tugas Custom

```
"buat tugas custom: essay 500 kata tentang etika AI, format DOCX, tanpa cover"
```

Jika tidak ada template yang cocok, sistem tawarkan buat template baru via menu 10 (template-creator).

### 6. Laporan Praktikum

```
"buat LA PC modul 2"
```

Ini di-delegate ke skill `laporan-praktikum` — bukan bagian dari Tugas System pipeline.

---

## Buat Template Tugas Baru

Kalau tipe tugas belum ada di `_defaults/`:

1. Pakai menu 10 (template-creator) — guided wizard
2. Atau copy YAML dari `_defaults/`, edit manual, simpan di `custom/`

Struktur minimal YAML:

```yaml
task_type: "nama-tipe"
display_name: "Nama Tampilan"
description: "Deskripsi singkat"
category: "matkul"

sections:
  - name: "Section 1"
    type: "generate"
    prompt_hint: "Instruksi generate"

format:
  output: "latex"  # atau "docx", "pptx"

generation:
  needs_paraphrase: true
  language: "id"

quality:
  check_format: true

naming:
  file_pattern: "Tugas_{Course}_{Name}"
```

Simpan di `templates/task-types/custom/nama-tipe.yaml` — langsung bisa dipakai.

---

## Ganti Semester

Edit `config/semester.json`:

```json
{
  "semester": "5",
  "tahun_ajaran": "2025/2026",
  "courses": [
    {
      "code": "PC",
      "name": "Pengantar Komputer",
      "aliases": ["PC", "Penkom"],
      "dosen": "Nama Dosen"
    }
  ]
}
```

Tambah/hapus mata kuliah sesuai semester berjalan.

---

## Batasan & Workarounds

| Batasan | Workaround |
|---------|------------|
| Search jurnal kadang tidak akurat | Kasih judul/DOI jurnal langsung |
| Paraphrase masih terasa AI | Minta revisi: "paraphrase ulang lebih natural" |
| Format DOCX terbatas | Pakai LaTeX → PDF untuk format presisi |
| Presentasi plain | Edit manual di PowerPoint setelah generate |
| Template tidak cocok | Buat custom template (menu 10) |

---

## Peta Komponen

```
User Command
    │
    ▼
┌─────────────────┐
│  TUGAS PIPELINE  │ ← pipelines/tugas-pipeline.md
│  (4 stages)      │
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐ ┌────────────┐
│  YAML  │ │   LAYERS   │
│template│ │ 01-04      │
└────────┘ └────────────┘
    │              │
    ▼              ▼
┌────────┐ ┌────────────┐
│semester│ │  compiled   │
│ .json  │ │  output     │
└────────┘ └────────────┘

Revisi? → pipelines/revision-pipeline.md (5 stages)
Custom?  → template-creator (menu 10)
LA?      → delegate ke skill laporan-praktikum
```
