# File Manipulation and Document Ingestion Pipeline

This laboratory establishes foundational data engineering and document ingestion
primitives for Natural Language Processing (NLP) pipelines. The implementation covers
low-level file stream input/output (I/O), deterministic resource lifecycle management,
character encoding integrity (UTF-8 for Arabic and non-Latin scripts), structured tabular
serialization (CSV and Excel), binary PDF document parsing and composition, and lexical
pattern extraction via regular expressions.

---

## Lab Structure

```text
file-manipulation/
├── README.md                          # Technical specification, stream architecture, and methodologies
├── data/                              # Data persistence directory
│   ├── arabic_doc.txt                 # Multilingual UTF-8 encoded text sample (Arabic script)
│   ├── data.txt                       # Stream buffer verification sample
│   ├── donnees.csv                    # Structured tabular dataset export
│   ├── output_excel.xlsx              # Multi-sheet Excel workbook generated via NumPy/Pandas
│   ├── output_scores.csv              # Synthetic numerical matrix export
│   └── sample.pdf                     # Compiled multi-page binary document sample
└── main.ipynb                         # Interactive implementation covering I/O, encodings, PDF, and regex
```

### File Responsibilities

- **`main.ipynb`**: Interactive notebook executing file descriptor manipulation, context management protocols, stream pointer positioning (`seek`), multilingual UTF-8 serialization, tabular data extraction and transformations (Pandas, openpyxl), binary PDF stream extraction and concatenation (PyPDF2), and regular expression matching (`re.search`, `re.findall`).
- **`data/`**: Storage directory holding input corpora, intermediary outputs, and generated tabular/document artifacts.

---

## Pipeline Architecture & Methodology

The document ingestion pipeline processes heterogeneous source formats into standardized representations for downstream NLP analysis:

```text
+-----------------------------------------------------------------------------------+
|                            Raw Heterogeneous Corpora                              |
|   +--------------------+   +--------------------+   +-------------------------+   |
|   |  Plain Text Files  |   |  Tabular Datasets  |   | Binary PDF Publications |   |
|   |   (.txt, UTF-8)    |   |   (.csv, .xlsx)    |   |         (.pdf)          |   |
|   +--------------------+   +--------------------+   +-------------------------+   |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                        Stream Processing & Normalization                          |
|  - Descriptor allocation & deterministic context management (with open)           |
|  - Cursor rewinding and buffered byte offsets (seek)                              |
|  - Multilingual Unicode decoding & character normalization (UTF-8)                |
|  - Binary object decoding and page tree traversal (PyPDF2 PdfReader)              |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                       Document Transformation & Extraction                        |
|  - Tabular schema parsing, column indexing, and multi-sheet generation            |
|  - Document slicing (PdfWriter) and binary stream concatenation (PdfMerger)       |
|  - Deterministic lexical pattern search and token span localization (re)          |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                   Standardized NLP Corpus / Feature Matrices                      |
|               (Token streams, text lists, structured DataFrames)                  |
+-----------------------------------------------------------------------------------+
```

---

## Core Methodological Modules

### 1. Low-Level Stream I/O & Context Managers
File operations interact directly with kernel-level file descriptor tables. Python provides two access patterns:
- **Explicit Lifecycle (`open` / `close`):** Requires manual resource reclamation. In the event of an unhandled runtime exception, the file descriptor remains locked, risking resource exhaustion.
- **Context Managers (`with` statement):** Implements PEP 343 (`__enter__` / `__exit__`). Guarantees deterministic stream closure upon scope exit regardless of execution status.

Stream access modes govern pointer placement and truncation:
- Read (`'r'`): Places cursor at byte 0; raises `FileNotFoundError` if missing.
- Write (`'w'`): Truncates existing file to 0 bytes; creates file if nonexistent.
- Append (`'a'`): Places cursor at End-Of-File (EOF); non-destructive additions.

### 2. Stream Ingestion Efficiency: Streaming vs. In-Memory Buffering
- **Lazy Line Streaming (`for line in f`):** Leverages Python file iterator buffers to read lines on demand with $O(1)$ auxiliary memory complexity. Mandatory when parsing multi-gigabyte text corpora.
- **Batch Evaluation (`.readlines()`):** Ingests the entire document into an in-memory Python list with $O(N)$ space complexity.

### 3. Stream Cursor Navigation (`seek`)
File streams maintain an internal byte offset pointer:
- Reading advances the pointer toward EOF.
- `seek(offset, whence=0)` explicitly repositions the pointer back to byte offset 0 (or arbitrary positions), enabling sequential multi-pass parsing over a single descriptor without re-executing operating system open calls.

### 4. Unicode Encodings & Multilingual Integrity (UTF-8)
Standard ASCII encodes characters within a 7-bit space (128 characters), while Latin-1 (ISO-8859-1) expands to 8 bits. Processing non-Latin scripts (e.g., Arabic, Hebrew, Cyrillic, Asian scripts) under legacy encodings triggers character corruption (*mojibake*) or `UnicodeDecodeError`.
Enforcing `encoding="utf-8"` ensures variable-length Unicode encoding (1 to 4 bytes per code point), preserving multilingual characters accurately across different operating systems.

### 5. Tabular Data Serialization (Pandas, CSV, Excel)
Tabular pipelines bridge unstructured text with categorical labels and metadata:
- `pd.DataFrame.to_csv()`: Exports structured corpora with explicit delimiter (`sep=","`), index suppression (`index=False`), and UTF-8 encoding.
- `pd.read_csv(skiprows=N)`: Ingests structured tables while bypassing irregular header annotations or contaminated preamble rows.
- `pd.DataFrame.to_excel()`: Interfaces with `openpyxl` engines to output formatted multi-sheet binary workbooks (`.xlsx`).

### 6. Binary PDF Parsing, Slicing & Merging (`PyPDF2`)
PDFs store content as binary object streams organized in cross-reference tables:
- **Text Extraction:** Ingested in binary mode (`'rb'`), `PdfReader` iterates across document page objects to extract decoded text payloads.
- **Document Slicing:** `PdfWriter` isolates discrete page objects and serializes new binary PDF files.
- **Stream Concatenation:** `PdfMerger` merges multiple PDF documents sequentially into a single consolidated file.

### 7. Lexical Pattern Matching (`re`)
Regular expressions enable deterministic lexical tokenization and string extraction:
- `re.search(pattern, text)`: Locates the first occurrence of `pattern`, providing substring extraction via `.group()` and exact character boundary offsets via `.span()`.
- `re.findall(pattern, text)`: Traverses the text exhaustively, returning all non-overlapping pattern matches as a list of strings.

---

## Execution Instructions

### Prerequisites
Ensure Python 3.10+ and the necessary numerical, tabular, and document processing packages are available:

```bash
pip install pandas numpy openpyxl PyPDF2 jupyter
```

### Running the Notebook
From the repository root or the `file-manipulation` directory:

```bash
cd file-manipulation
jupyter notebook main.ipynb
```

Execute all notebook cells sequentially to verify text persistence, encoding compliance, tabular exports, PDF parsing, and regular expression operations.

---

## Technology Stack

- **Language:** Python 3
- **Data Engineering:** Pandas, NumPy
- **Spreadsheet Engine:** Openpyxl
- **Document Processing:** PyPDF2
- **Lexical Pattern Matching:** Python Regular Expressions (`re`)
- **Interactive Development:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
