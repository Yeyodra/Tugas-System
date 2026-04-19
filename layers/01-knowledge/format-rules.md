---
layer: 01-knowledge
type: skill
name: format-rules
---

# Purpose

Reads and parses format rules from a task-type YAML configuration file. Provides a normalized format rules object that downstream layers (format-checker, content-generator, latex-compiler) consume.

This layer is **generic** — it does NOT hardcode any specific format. All rules come from the YAML.

# Input

- Path to a task-type YAML file (e.g., `templates/task-types/makalah.yaml`, `templates/task-types/tugas-pemrograman.yaml`).
- If no path is provided, attempt to resolve from `semester.json → active_task → task_type` → look up in `templates/task-types/`.
- If YAML is missing or unreadable, fall back to sensible defaults.

# Output

A **Format Rules Object** with the following normalized structure:

```yaml
format:
  paper_size: "A4"           # from YAML or default
  margins:                   # cm
    top: 3
    bottom: 3
    left: 3
    right: 3
  font:
    family: "Times New Roman"
    size_body: 12
    size_title: 14
  line_spacing: 1.5
  paragraph_indent: 1.25     # cm
  text_alignment: "justified"

cover:
  required: true/false
  elements: [title, logo, name, npm, class, course, lecturer, university, year]

sections:
  numbering: true/false
  title_format: "14pt bold"
  subsection_format: "12pt bold"

code_listings:
  enabled: true/false
  style: "frame"             # frame, plain, colored

citations:
  required: true/false
  style: "APA"               # if applicable
  min_references: 0          # 0 = no minimum

output_formats:
  - pdf
  - docx                     # optional
```

# Process

## 1. Locate YAML

Priority order:
1. Explicit path argument (if provided by caller).
2. `semester.json → tasks[active] → task_type` → resolve to `templates/task-types/{task_type}.yaml`.
3. If task type has a `_defaults/` base, load that first, then overlay the specific YAML.

## 2. Parse YAML

Read the YAML file. Extract the `format` section. If the YAML uses a `base:` field (e.g., `base: _defaults/standard.yaml`), load the base first, then merge the specific YAML on top (specific values override base values).

## 3. Normalize

Map YAML fields to the normalized Format Rules Object structure above. For any missing field, apply these defaults:

| Field | Default |
|-------|---------|
| paper_size | A4 |
| margins (all) | 3cm |
| font.family | Times New Roman |
| font.size_body | 12 |
| font.size_title | 14 |
| line_spacing | 1.5 |
| paragraph_indent | 1.25cm |
| text_alignment | justified |
| cover.required | true |
| sections.numbering | true |
| code_listings.enabled | false |
| citations.required | false |
| citations.min_references | 0 |

## 4. Validate

Check for contradictions or invalid values:
- Margins must be > 0 and < 10cm.
- Font size must be between 8 and 24.
- Line spacing must be between 1.0 and 3.0.

If validation fails, log a warning and use the default for that field.

## 5. Return

Return the Format Rules Object to the caller. This object is consumed by:
- **format-checker** (layer 03) — to validate documents against rules.
- **content-generator** (layer 02) — to apply correct formatting during generation.
- **latex-compiler** (layer 04) — to select the correct cls and compile options.

# Error Handling

| Scenario | Behavior |
|----------|----------|
| YAML file not found | Log warning, return full defaults object |
| YAML parse error | Log error with line number, return full defaults object |
| Missing `format` section | Use defaults for all format fields |
| Unknown field in YAML | Ignore (forward-compatible) |
| Invalid value (e.g., margin: -1) | Log warning, use default for that field |

# Integration Points

- **format-checker.md** (layer 03): Primary consumer — validates documents against these rules.
- **content-generator** (layer 02): Uses rules to format generated content correctly.
- **latex-compiler.md** (layer 04): Uses rules to select cls file and compile options.
- **semester.json**: Source of active task context for auto-resolution.
