---
name: file-safety
type: protocol
description: >
  Prevents data loss by creating timestamped backups before any .tex file overwrite.
  This is a PROTOCOL — not a standalone skill. Other skills/pipelines follow these instructions
  before writing output files.
---

# File Safety Protocol

## Purpose

Prevent data loss when the PI System overwrites existing `.tex` files during generation, paraphrasing, or any write operation. Every overwrite MUST be preceded by a backup.

## When to Apply

**BEFORE** any write operation that targets an existing `.tex` file. This includes:
- Draft generation (draft-generator)
- Paraphrasing output
- Format corrections
- Any pipeline stage that saves to `bab/*.tex`

## Backup Procedure

### Step 1: Check if target file exists

If the target file does NOT exist yet (first-time generation), skip backup and proceed with write.

### Step 2: Create timestamped backup

Copy the existing file with a timestamp suffix:

**Windows (PowerShell):**
```powershell
$timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
Copy-Item "{file}" "{file}.bak.$timestamp"
```

**Windows (cmd):**
```cmd
copy "{file}" "{file}.bak.%date:~-4%%date:~-7,2%%date:~-10,2%-%time:~0,2%%time:~3,2%%time:~6,2%"
```

**Pattern:** `{filename}.bak.{YYYYMMDD-HHMMSS}`

**Example:**
```
bab1.tex → bab1.tex.bak.20260419-143022
```

### Step 3: Enforce retention (max 5 backups per file)

After creating the new backup, check how many `.bak.*` files exist for this file. If more than 5, delete the oldest ones.

**PowerShell:**
```powershell
$backups = Get-ChildItem "{file}.bak.*" | Sort-Object LastWriteTime
if ($backups.Count -gt 5) {
    $backups | Select-Object -First ($backups.Count - 5) | Remove-Item
}
```

### Step 4: Proceed with write

After backup is confirmed (or skipped because file didn't exist), proceed with the normal write operation.

## Failure Handling

- If backup fails (permission error, disk full, etc.) — **log a warning but proceed with the write anyway**
- Backup must NEVER block the pipeline
- The write operation is more important than the backup

## Configuration

Controlled by `config/defaults.json`:
```json
{
  "output": {
    "overwrite_existing": false,
    "backup": {
      "enabled": true,
      "max_backups_per_file": 5,
      "backup_suffix": ".bak"
    }
  }
}
```

When `backup.enabled` is `false`, skip all backup steps (useful for CI/testing).
When `overwrite_existing` is `false` and no backup config exists, prompt user before overwriting.

## Integration Points

This protocol is referenced by:
- `layers/03-writing/draft-generator.md` — before Step 6 (Save)
- `pipelines/bab-pipeline.md` — before Stage 5 (Progress & Save)
- Any future skill that writes `.tex` files

## Key Principles

1. **Simple** — just a file copy, no compression, no separate directories
2. **Non-blocking** — failure doesn't stop the pipeline
3. **Local** — backups live next to the original file
4. **Bounded** — max 5 backups per file prevents disk bloat
5. **Transparent** — timestamp in filename makes it obvious which version is which
