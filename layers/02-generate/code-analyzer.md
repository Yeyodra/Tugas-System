---
name: code-analyzer
description: >
  Analyze source code and generate logic explanations in Indonesian academic style.
  Reads code files, breaks them into logical sections (imports, classes, functions,
  main logic), and produces LaTeX-formatted explanations with code snippets.
  Used for "laporan praktikum" and any task requiring code documentation.
---

# Purpose

Read source code, understand its logic, and generate structured explanations suitable for academic reports. The output is LaTeX content with interleaved code snippets and Indonesian-language explanations — the "LOGIKA PROGRAM" section common in Indonesian CS lab reports.

# Input

| Parameter | Required | Description |
|-----------|----------|-------------|
| `code_path` | YES | Path to source code file(s) — single file or directory |
| `language` | YES | Programming language (java, python, javascript, c, cpp, etc.) |
| `explanation_depth` | NO | `brief` (1-2 sentences per block), `normal` (default), `detailed` (paragraph per block) |
| `focus_functions` | NO | List of specific function/class names to analyze (empty = all) |
| `output_style` | NO | `interleaved` (code + explanation alternating, default) or `separate` (all code then all explanation) |

# Output

LaTeX content with code snippets and explanations:

```latex
\subsection{Penjelasan Logika Program}

Berikut adalah penjelasan logika dari program yang telah dibuat.

\subsubsection{Import dan Konfigurasi}

\begin{lstlisting}[language=Python, caption={Import library yang digunakan}]
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
\end{lstlisting}

Pada bagian awal program, dilakukan import terhadap beberapa library yang dibutuhkan.
Library \texttt{pandas} digunakan untuk manipulasi data dalam bentuk DataFrame,
\texttt{numpy} untuk operasi numerik, dan \texttt{sklearn} untuk pembagian data
menjadi data latih dan data uji.

\subsubsection{Fungsi Preprocessing}

\begin{lstlisting}[language=Python, caption={Fungsi preprocessing data}]
def preprocess_data(df):
    df = df.dropna()
    df['amount'] = df['amount'].astype(float)
    return df
\end{lstlisting}

Fungsi \texttt{preprocess\_data} menerima parameter berupa DataFrame dan melakukan
dua tahap pembersihan data. Pertama, baris yang mengandung nilai kosong (NaN)
dihapus menggunakan metode \texttt{dropna()}. Kedua, kolom \texttt{amount}
dikonversi ke tipe data float untuk memastikan konsistensi tipe data numerik.
```

---

# Process

## Step 1: READ — Load Source Code

### 1.1 Single File

Read the file at `code_path`. Detect language from extension if `language` not specified:

| Extension | Language |
|-----------|----------|
| `.py` | Python |
| `.java` | Java |
| `.js`, `.ts` | JavaScript / TypeScript |
| `.c` | C |
| `.cpp`, `.cc` | C++ |
| `.php` | PHP |
| `.go` | Go |
| `.rb` | Ruby |
| `.kt` | Kotlin |
| `.swift` | Swift |
| `.rs` | Rust |

### 1.2 Directory

If `code_path` is a directory:
1. List all source files matching the language extension
2. Sort by logical order: config → models → utils → main/app
3. Process each file sequentially
4. If total lines > 300, focus on key files only (entry point + core logic)

### 1.3 Focus Filter

If `focus_functions` specified:
- Extract only the named functions/classes/methods
- Include their imports and dependencies
- Skip everything else

---

## Step 2: PARSE — Break into Logical Sections

### 2.1 Section Detection

Break the code into logical blocks:

| Block Type | Detection Pattern |
|-----------|-------------------|
| **Imports / Dependencies** | `import`, `require`, `#include`, `using` statements at file top |
| **Constants / Config** | `const`, `final`, `#define`, uppercase variable assignments |
| **Class Definition** | `class`, `struct`, `interface` declarations |
| **Function / Method** | `def`, `function`, `func`, method signatures |
| **Main Logic** | `main()`, `if __name__`, top-level execution code |
| **Event Handlers** | `addEventListener`, `@app.route`, `onClick` |
| **Database Operations** | SQL queries, ORM calls, migration code |
| **Error Handling** | `try/catch/except` blocks |
| **Utility / Helper** | Small functions used by main logic |

### 2.2 Block Extraction

For each detected block:

```json
{
  "type": "function",
  "name": "preprocess_data",
  "start_line": 15,
  "end_line": 28,
  "code": "def preprocess_data(df):\n    ...",
  "dependencies": ["pandas"],
  "complexity": "simple",
  "parameters": ["df: DataFrame"],
  "returns": "DataFrame"
}
```

### 2.3 Ordering

Order blocks for explanation:
1. Imports / Dependencies (what tools are used)
2. Constants / Config (what settings exist)
3. Class definitions (what structures exist)
4. Utility functions (helper logic)
5. Core functions (main business logic)
6. Main execution / entry point (how it all connects)

---

## Step 3: EXPLAIN — Generate Logic Explanations

### 3.1 Explanation Generation Rules

For each code block, generate an explanation that:

1. **States the purpose** — what this block does (1 sentence)
2. **Explains the mechanism** — how it works (1-3 sentences depending on depth)
3. **Notes key details** — important parameters, return values, edge cases
4. **Uses proper terminology** — Indonesian academic register with technical terms preserved

### 3.2 Language Rules

- Write in **formal Indonesian academic style**
- Technical terms stay in English: `DataFrame`, `array`, `function`, `class`, `method`
- Technical terms in *italic* when first introduced: *library*, *framework*
- Use `\texttt{}` for code references: `\texttt{preprocess\_data}`
- Escape underscores in LaTeX: `\_`
- No abbreviations: "dll" → "dan lain-lain", "yg" → "yang"
- No personal pronouns: avoid "kita", "saya", "anda"

### 3.3 Explanation Depth Levels

**Brief** (`explanation_depth: brief`):
```
Fungsi \texttt{preprocess\_data} membersihkan data dengan menghapus baris kosong
dan mengkonversi tipe data kolom \texttt{amount} ke float.
```

**Normal** (`explanation_depth: normal`) — DEFAULT:
```
Fungsi \texttt{preprocess\_data} menerima parameter berupa DataFrame dan melakukan
dua tahap pembersihan data. Pertama, baris yang mengandung nilai kosong (NaN)
dihapus menggunakan metode \texttt{dropna()}. Kedua, kolom \texttt{amount}
dikonversi ke tipe data float untuk memastikan konsistensi tipe data numerik.
```

**Detailed** (`explanation_depth: detailed`):
```
Fungsi \texttt{preprocess\_data} merupakan tahap awal dalam alur pemrosesan data
yang bertujuan untuk memastikan kualitas data sebelum digunakan dalam analisis
lebih lanjut. Fungsi ini menerima satu parameter yaitu \texttt{df} yang bertipe
DataFrame dari library pandas.

Tahap pertama yang dilakukan adalah penghapusan baris yang mengandung nilai kosong
(\textit{missing values}) menggunakan metode \texttt{dropna()}. Hal ini penting
karena keberadaan nilai kosong dapat menyebabkan error pada proses perhitungan
selanjutnya.

Tahap kedua adalah konversi tipe data pada kolom \texttt{amount} menjadi tipe
float menggunakan metode \texttt{astype(float)}. Konversi ini diperlukan karena
data yang dibaca dari sumber eksternal seringkali tersimpan dalam format string,
sehingga perlu diubah ke format numerik agar dapat dilakukan operasi matematika.
```

### 3.4 Explanation Patterns by Block Type

**Imports:**
```
Pada bagian awal program, dilakukan import terhadap beberapa library yang dibutuhkan.
Library {name} digunakan untuk {purpose}.
```

**Class:**
```
Kelas {ClassName} merupakan {purpose}. Kelas ini memiliki {n} atribut yaitu
{attr_list} dan {m} metode yaitu {method_list}.
```

**Function:**
```
Fungsi {func_name} {purpose}. Fungsi ini menerima parameter {params} dan
mengembalikan {return_type}. {mechanism_explanation}.
```

**Main logic:**
```
Pada bagian utama program, {flow_description}. Proses dimulai dengan {step1},
kemudian {step2}, dan diakhiri dengan {step3}.
```

**Error handling:**
```
Blok penanganan error (\textit{error handling}) digunakan untuk mengantisipasi
kemungkinan terjadinya {error_type}. Jika terjadi error, program akan {action}.
```

---

## Step 4: FORMAT — Assemble LaTeX Output

### 4.1 Interleaved Style (Default)

Alternate between code blocks and explanations:

```latex
\subsection{Penjelasan Logika Program}

Berikut adalah penjelasan logika dari program {filename} yang telah dibuat.

\subsubsection{Import dan Konfigurasi Awal}

\begin{lstlisting}[language={lang}, caption={Deskripsi blok}, label={lst:block_id}]
% code block
\end{lstlisting}

% explanation paragraph

\subsubsection{Fungsi Utama}

\begin{lstlisting}[language={lang}, caption={Deskripsi blok}, label={lst:block_id}]
% code block
\end{lstlisting}

% explanation paragraph
```

### 4.2 Separate Style

All code first, then all explanations:

```latex
\subsection{Kode Program}

\begin{lstlisting}[language={lang}, caption={Kode lengkap program}, label={lst:full_code}]
% full code
\end{lstlisting}

\subsection{Penjelasan Logika Program}

% all explanations in order, referencing line numbers
```

### 4.3 lstlisting Configuration

```latex
\lstset{
  language={language},
  basicstyle=\ttfamily\small,
  breaklines=true,
  frame=single,
  numbers=left,
  numberstyle=\tiny\color{gray},
  keywordstyle=\color{blue},
  commentstyle=\color{green!60!black},
  stringstyle=\color{red!70!black},
  showstringspaces=false,
  tabsize=4,
  captionpos=b
}
```

---

# Quality Checks

| Check | Action |
|-------|--------|
| Every code block has an explanation | No orphan code blocks |
| Explanations reference actual code elements | Use `\texttt{}` for code refs |
| LaTeX special chars escaped | `_` → `\_`, `%` → `\%`, `&` → `\&` in text |
| Code compiles/runs (if verifiable) | Note if code has syntax errors |
| Explanation matches code behavior | Don't describe what code doesn't do |
| No AI-tell patterns | Avoid "delve", "crucial", "comprehensive" |
| Consistent terminology | Same term for same concept throughout |

---

# Error Handling

| Error | Action |
|-------|--------|
| File not found | Return placeholder: `[FILE NOT FOUND: {path}]` |
| Binary file (not source code) | Skip with warning |
| File too large (>500 lines) | Analyze key sections only, note truncation |
| Unknown language | Best-effort analysis, warn about accuracy |
| Encoding error | Try UTF-8, then Latin-1, then skip with error |

---

# Integration Points

| Layer | Relationship |
|-------|-------------|
| `content-generator.md` | Called for `code_listing` and `delegate → code-analyzer` sections |
| `diagram-generator.md` | Can generate class/sequence diagrams from analyzed code |
| `paraphraser.md` | Can refine explanation text if needed |
| `plagiarism-guard.md` | Checks explanations aren't copied from tutorials |
