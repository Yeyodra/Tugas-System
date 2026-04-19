---
name: figure-generator
description: Bridge to figure-generation skill for creating publication-quality scientific figures
---

# Purpose
Generate matplotlib or seaborn figures for PI research papers. It handles various types of data visualization required for academic reporting.

# Input
- Figure description
- Data source (CSV, JSON, or NPY files — user-provided path)
- Figure type (bar, training-curve, heatmap, ablation, line, scatter, radar, violin, tsne, attention)

# Output
- PNG file (300 DPI preview)
- PDF file (vector format for final paper)
- LaTeX code snippet to include the figure

# Process
## 1. Query Expansion
Expand the user description into detailed coding specifications, including figure type, data mapping, and visual style.

## 2. Code Generation and Execution
Generate a Python script using the `figure-generation` skill and execute it. The system will retry up to 4 times if errors occur during execution.

## 3. Visual Refinement
Inspect the generated figure. Verify labels, legends, colors, and scales. Regenerate if the visual representation does not meet standards.

# Gunadarma PI Format Rules
- Figure caption MUST be placed BELOW the figure.
- Numbering format: Gambar X.Y (e.g., Gambar 3.1).
- Captions must be in Bahasa Indonesia.
- Figures must be centered.
- Reference figures in text by number (e.g., "pada Gambar 3.1"), not by position.

# Quality Requirements
- Resolution must be at least 300 DPI.
- Use colorblind-friendly palettes.
- All text within the figure must be >= 8pt.
- Style must be consistent across all figures in the project.

# Integration Points
- **Data Analysis Layer 02**: Provides the processed data for visualization.
- **Draft Generator Layer 03**: Embeds the generated figures into the document.
- **LaTeX Compiler Layer 05**: Includes the PDF figures in the final document assembly.
