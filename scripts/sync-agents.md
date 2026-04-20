# Sync Tugas-System AGENTS.md Entry

## Purpose
Keep the Tugas-System auto-detection block in the global AGENTS.md in sync with this repository's `AGENTS.md.snippet`.

## When to Sync
- After updating triggers (new keywords like "buat tugas", "kerjain tugas", etc.)
- After adding new paths (new template directories, new output locations)
- After adding new features, task types, or courses to Tugas-System
- After modifying the auto-detection instructions
- After semester changes (new semester config, new courses)

## How to Sync (AI Agent Instructions)

1. **Read** the snippet file:
   ```
   C:\Users\Hanni\Documents\Projek\Tugas-System\AGENTS.md.snippet
   ```

2. **Open** the global AGENTS.md:
   ```
   C:\Users\Hanni\.config\opencode\AGENTS.md
   ```

3. **Find** the markers:
   ```
   <!-- TUGAS-SYSTEM:START -->
   ```
   and
   ```
   <!-- TUGAS-SYSTEM:END -->
   ```

4. **Replace** everything BETWEEN the markers (not the markers themselves) with the content of `AGENTS.md.snippet`.

   The result should look like:
   ```markdown
   <!-- TUGAS-SYSTEM:START -->
   {content of AGENTS.md.snippet}
   <!-- TUGAS-SYSTEM:END -->
   ```

5. **Save** the global AGENTS.md.

6. **Verify** the markers are intact and the content between them matches the snippet exactly.

## File Locations
| File | Path |
|------|------|
| Snippet (source of truth) | `C:\Users\Hanni\Documents\Projek\Tugas-System\AGENTS.md.snippet` |
| Global AGENTS.md (target) | `C:\Users\Hanni\.config\opencode\AGENTS.md` |
| Marker start | `<!-- TUGAS-SYSTEM:START -->` |
| Marker end | `<!-- TUGAS-SYSTEM:END -->` |

## Important
- Do NOT modify the markers themselves
- Do NOT modify any content outside the markers
- The snippet file is the **source of truth** — always copy FROM snippet TO global AGENTS.md
- After syncing, the global AGENTS.md should still be valid markdown
