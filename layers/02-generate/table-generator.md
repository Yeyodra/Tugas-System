---
name: table-generator
description: Bridge to table-generation skill for generating publication-quality LaTeX tables
---

# Purpose
Convert experimental results and literature data into booktabs-styled LaTeX tables. This skill ensures that tables meet the high standards of academic publication and specific university requirements.

# Input
- Table type (comparison, ablation, descriptive, literature, custom)
- Data source (JSON/CSV from pi-project.json paths or inline data)

# Output
- LaTeX table code utilizing booktabs, multirow, multicol, and threeparttable packages.
- Properly formatted caption and label.

# Table Types
- **comparison**: Main results comparing methods against metrics with bolded best results.
- **ablation**: Component removal study, typically using checkmarks.
- **descriptive**: Dataset characteristics, hyperparameters, or summary statistics.
- **literature**: BAB 2 Tinjauan Pustaka comparison table (Author, Year, Method, Dataset, Results, Limitation). This is the "tabel perbandingan" often required by dospem.
- **custom**: Free-form table structure.

# Gunadarma PI Format Rules
- Table caption MUST be placed ABOVE the table.
- Numbering format: Tabel X.Y (e.g., Tabel 2.1).
- Captions must be in Bahasa Indonesia.
- Tables must be centered.
- Reference tables in text by number (e.g., "lihat Tabel 2.1").

# Process
## 1. Data Parsing
Extract data from specified sources or handle inline input provided in the task.

## 2. Table Construction
Map data to the selected table type. Apply academic formatting such as bolding the best performing metrics and setting up multi-row layouts where necessary.

## 3. Package Verification
Ensure the generated LaTeX uses standard booktabs rules (no vertical lines, specific horizontal line weights).

# Integration Points
- **Data Analysis Layer 02**: Provides experimental results for comparison and ablation tables.
- **Literature Search Layer 02**: Provides the data for BAB 2 comparison tables.
- **Draft Generator Layer 03**: Embeds the generated table code into the document drafts.

# Advisor Insight
Dospem's Rule: "Tinjauan pustaka yang bagus itu bukan sekadar ngerangkum. Bikin tabel perbandingan itu ampuh banget buat metain riset yang udah ada."
