---
layer: 04-output
type: skill
name: progress-tracker
---

# Purpose

Manages and visualizes the progress of assignments by maintaining status in `semester.json`. Tracks individual task progress across the semester.

# Input

- `semester.json` file.
- Feedback from generation, QA, or compilation tasks.
- User approval/confirmation.

# Output

- Updated `semester.json`.
- Progress summary report.

# Process

## 1. Status Management

Update the `status` field for individual tasks in `semester.json → tasks[]`.

Allowed status values:
- **pending**: Task has not started.
- **in_progress**: Currently being worked on.
- **draft**: Initial content generated but not reviewed.
- **revision**: Content needs changes based on QA or user feedback.
- **submitted**: Content is final and submitted/ready for submission.

## 2. Approval Rules

- ONLY transition to `submitted` status when the user explicitly confirms (e.g., "done", "submit", "ACC", "final").
- Automatic transitions to `submitted` are strictly forbidden.
- Transition from `draft` → `revision` can happen automatically after QA finds issues.

## 3. Progress Display

Generate a formatted progress report:

```
=== Semester Progress ===
Mata Kuliah: {course_name}

Task 1: {title}
  Status: draft | Due: 2026-04-25
  Word count: 1,200 | Sections: 4/4

Task 2: {title}
  Status: pending | Due: 2026-05-02

Overall: 1/5 submitted, 1/5 draft, 3/5 pending
```

Include per-task:
- **Status**: Current status.
- **Due date**: If specified in semester.json.
- **Metrics**: Word count, section count, code line count (if applicable).
- **QA Summary**: Brief summary of latest quality check (if run).

## 4. Automatic Updates

Run the tracker after every significant event:
- After content generation → set status to `draft`.
- After QA with issues → set status to `revision`.
- After successful compilation → log compilation timestamp.
- After user approval → set status to `submitted`.

## 5. Next-Steps Recommendation

Based on current status, recommend the next action:

| Status | Recommendation |
|--------|---------------|
| pending | Start content generation |
| in_progress | Continue working; run QA when ready |
| draft | Run format-checker and quality checks |
| revision | Apply feedback, then re-run QA |
| submitted | No action needed; move to next task |

## 6. Deadline Awareness

If `semester.json` includes due dates:
- Flag tasks due within 3 days as **URGENT**.
- Flag overdue tasks as **OVERDUE**.
- Sort recommendations by deadline proximity.

# Integration Points

- **All layers**: Call the tracker to update state after checkpoints.
- **latex-compiler** (layer 04): Updates compilation status.
- **format-checker** (layer 03): Updates QA status.
- **semester.json**: Single source of truth for all task state.
