---
name: diagram-generator
description: Generates and renders UML/ERD diagrams for PI BAB 3 from source code.
---

# Purpose
Automates the creation of technical diagrams required for the "Perancangan" section of BAB 3 by analyzing the project's source code.

# Input
- `source_path`: Path to source code (from `pi-project.json` `paths.source_code`)
- `diagram_type`: ERD, Use Case, Activity, Sequence, or Class
- `output_dir`: Path for rendered images

# Process: Scan-Generate-Render Pipeline

## Step 1: Scan
- **ERD**: Scans `schema.ts`, `models/`, or database migration files to identify entities and relationships.
- **Sequence/Activity**: Scans `routes/`, `actions/`, or controller logic to map execution flows.
- **Class**: Scans core service classes or type definitions.

## Step 2: Generate
Creates source files for the appropriate tool:
- **Mermaid (.mmd)**: Used for ERD and Class Diagrams.
- **PlantUML (.puml)**: Used for Use Case, Activity, and Sequence Diagrams.

## Step 3: Render
Invokes CLI tools to produce PNG output:
- **Mermaid**: `mmdc -i input.mmd -o output.png`
- **PlantUML**: `java -jar plantuml.jar input.puml`

# LaTeX Integration
Generates the corresponding LaTeX code to include the rendered image:
```latex
\begin{figure}[H]
    \centering
    \includegraphics[width=0.8\textwidth]{figures/diagram_name.png}
    \caption{Description of the diagram in Indonesian}
    \label{fig:diagram_name}
\end{figure}
```

# Tool Selection Logic
- Use **Mermaid** for: ERD (Entity Relationship), Class Diagrams (simpler syntax for data structures).
- Use **PlantUML** for: Use Case (complex actor relationships), Activity (detailed logical flow), Sequence (time-ordered interactions).

# Integration Points
- **Figure Generation Skill**: Bridges to `skills/figure-generation`.
- **Draft Generator**: Injects the generated `\includegraphics` blocks into BAB 3 `.tex` files.
- **Source Code**: Direct dependency on the path defined in `pi-project.json`.
