# Revision Pipeline

Pipeline sederhana untuk revisi tugas berdasarkan feedback dosen atau self-review.
Tidak ada peer review loop atau integrity check — langsung dan cepat.

---

## Stage 1: PARSE FEEDBACK

```
Input  : feedback (text dari dosen, screenshot, atau self-review notes)
Output : structured revision list
```

- Terima feedback dalam bentuk apapun (text, foto, voice transcript)
- Parse jadi list revisi terstruktur:
  ```
  - section: "Pembahasan"
    issue: "Kurang referensi pendukung"
    action: "regenerate"  # or "edit"
  - section: "Format"
    issue: "Margin kiri harusnya 4cm"
    action: "edit"
  ```
- Prioritaskan: content issues > format issues > minor fixes

---

## Stage 2: LOAD CONTEXT

```
Input  : revision list
Output : current document + format rules loaded
```

- Load file tugas yang akan direvisi
- Load YAML template yang dipakai (dari task type)
- Load format rules untuk validasi nanti
- Identifikasi section mana yang perlu diubah

---

## Stage 3: APPLY REVISIONS

```
Input  : document + revision list
Output : revised document
```

Dua mode revisi:

### Mode A: Direct Edit
Untuk perubahan kecil (typo, format, tambah kalimat):
- Edit langsung di source file
- Preserve bagian yang tidak direvisi

### Mode B: Regenerate Section
Untuk perubahan besar (tulis ulang section, tambah referensi):
- Regenerate section pakai YAML `prompt_hint` + feedback context
- Apply paraphrase jika `needs_paraphrase: true`
- Merge dengan bagian yang tidak berubah

---

## Stage 4: RE-CHECK FORMAT

```
Input  : revised document
Output : validation report
```

- Jalankan format check dari YAML `quality` block
- Pastikan revisi tidak merusak format yang sudah benar
- Advisory only — tampilkan warning, jangan block

---

## Stage 5: RECOMPILE

```
Input  : revised document (validated)
Output : compiled file (PDF/DOCX/PPTX)
```

- Backup versi sebelumnya (file-safety protocol)
- Compile ulang ke target format
- Update timestamp di semester.json

---

## Flow Diagram

```
Feedback
    │
    ▼
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  PARSE  │───▶│  LOAD   │───▶│  APPLY  │───▶│ RECHECK │───▶│RECOMPILE│
│ feedback│    │ context │    │revision │    │ format  │    │         │
└─────────┘    └─────────┘    └─────────┘    └─────────┘    └─────────┘
```

---

## Contoh Penggunaan

```
User: "revisi makalah KDM, dosen bilang kurang referensi di bab 2"

→ Parse: section=Tinjauan Pustaka, action=regenerate, issue=kurang referensi
→ Load: makalah KDM + makalah.yaml
→ Apply: search tambahan referensi, regenerate Tinjauan Pustaka
→ Recheck: format OK
→ Recompile: PDF baru, backup versi lama
```
