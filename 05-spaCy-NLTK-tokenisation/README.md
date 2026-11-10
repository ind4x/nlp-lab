# Linguistic Tokenisation and Sentence Boundary Disambiguation with spaCy and NLTK

This laboratory investigates the algorithmic, linguistic, and architectural foundations of lexical tokenisation and Sentence Boundary Disambiguation (SBD) in Natural Language Processing (NLP) pipelines. The implementation contrasts industrial-grade production frameworks (`spaCy`) with academic reference libraries (`NLTK`), covering token lifecycle management, orthographic shape analysis, zero-copy span slicing, contraction and clitic resolution, multilingual script processing (Arabic), statistical dependency-driven sentence segmentation, custom in-pipeline boundary components, and unsupervised collocation models (Punkt).

---

## Lab Structure

```text
05-spaCy-NLTK-tokenisation/
├── README.md                          # Technical specification, pipeline architecture, and benchmarks
├── main.ipynb                         # Unified interactive master notebook combining tokenisation and SBD
├── tokenisation/                      # Specialized word and sub-word tokenisation module
│   └── main.ipynb                     # Dedicated notebook for lexical attributes, clitics, and benchmarks
└── segmentation/                      # Specialized sentence boundary disambiguation module
    └── main.ipynb                     # Dedicated notebook for doc.sents, Punkt, and custom pipelines
```

### File Responsibilities

- **`main.ipynb`**: Consolidated master notebook executing the complete end-to-end curriculum: environment configuration, model loading, token attribute analysis, span manipulation, contraction splitting, comparative benchmarks (`str.split()` vs. NLTK vs. spaCy), Arabic non-Latin tokenisation, syntactic sentence segmentation, custom boundary injection, and Punkt evaluation.
- **`tokenisation/main.ipynb`**: Focused module detailing lexical tokenisation mechanics, orthographic shapes (`shape_`), lexical flags (`is_alpha`, `is_stop`), subsequence views (`Span`), string immutability, and comparative tokenisation on literary and multilingual corpora.
- **`segmentation/main.ipynb`**: Focused module detailing sentence boundary disambiguation, `Doc.sents` generator behavior, token boundary flags (`is_sent_start`), ambiguity resolution in periods (abbreviations and decimals), custom component development (`@Language.component`), and comparative evaluation between spaCy and NLTK Punkt.

---

## Pipeline Architecture & Methodologies

```text
+-----------------------------------------------------------------------------------+
|                             Raw Unstructured Text Stream                          |
|     e.g., "Dr. Smith arrived at 10.30 a.m. We're moving to L.A.! Cost: $10.30."   |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                        Stage 1: Lexical Tokenisation Engine                       |
|  - Prefix / Infix / Suffix Regex Matching: Isolate punctuation from token roots   |
|  - Exception Lexicon Lookups: Protect multi-period abbreviations ("L.A.", "U.S.") |
|  - Clitic and Contraction Splitting: "We're" -> ["We", "'re"]                    |
|  - Web and Digital Entity Preservation: Protect email URIs and web domains        |
|  - Currency and Quantity Isolation: "$10.30" -> ["$", "10.30"]                    |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                         Stage 2: spaCy Memory & Object Model                      |
|  - Doc Container: Efficient C-level array of Token structs without string copying  |
|  - Token Views: Lexical flags (is_alpha, is_stop), Orthographic shape (shape_)    |
|  - Span Views: Contiguous slices [start, end) sharing parent memory buffers        |
|  - StringStore Vocabulary Hashing: Global uint64 integer hash mapping             |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                 Stage 3: Sentence Boundary Disambiguation (SBD)                   |
|  - Statistical Dependency Parsing: Transition-based parser assigns syntactic heads |
|  - Token Boundary Annotation: Explicit assignment of token.is_sent_start           |
|  - Custom Pipeline Injection: @Language.component adds pre-parser boundary rules   |
|  - Unsupervised Punkt Alternative (NLTK): Heuristic abbreviation collocations      |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                             Downstream NLP Ingestion                              |
|   - Part-of-Speech Tagging    - Named Entity Recognition    - Syntactic Parsing   |
+-----------------------------------------------------------------------------------+
```

---

## Technical Methodologies

### 1. The spaCy Object Model: `Doc`, `Token`, and `Span` Views

Unlike naive approaches that return standard Python lists of strings, spaCy constructs an immutable, memory-efficient container hierarchy:
- **`Doc`**: An array of C-level token structs maintaining character offsets, lexical attributes, and dependency links. Modifying text requires string alteration and re-running the processing pipeline.
- **`Token`**: A lightweight view referencing an individual token at index $i$ within the parent `Doc`.
- **`Span`**: A contiguous slice of tokens `doc[start:end]` sharing the parent's memory buffer, avoiding allocations when passing phrases or clauses to downstream extractors.

### 2. Lexical, Morphological, and Orthographic Attributes

Every `Token` exposes rich structural indicators computed during processing:
- `token.text`: The exact Unicode substring.
- `token.shape_`: An orthographic abstraction indicating casing, punctuation, and digits (for example, `Xxxx` for capitalized words, `d` for numbers, `dd.dd` for decimals).
- `token.is_alpha`: Boolean verification of pure alphabetical character content.
- `token.is_stop`: Boolean membership flag against language-specific stopword dictionaries.

### 3. Handling Complex Orthographic Edge Cases

Standard whitespace splitting fails when encountering real-world syntactic patterns:
- **Contractions:** `"We're"` is split into the base pronoun `"We"` and the contracted auxiliary clitic `"'re"`, enabling accurate grammatical tagging.
- **Abbreviations:** Multi-period acronyms such as `"U.S."` and `"L.A."` are identified via exception lexicons, preventing false sentence splits.
- **Digital Identifiers:** Email addresses (`support@oursite.com`) and hyperlinks (`http://wwwoursite.com`) are preserved as single lexical tokens.
- **Currency Symbols:** Currency signs are detached from quantities (`"$10.30"` $\implies$ `["$", "10.30"]`), isolating monetary values for numerical feature extraction.

### 4. Multilingual Processing and Agglutinative Scripts (Arabic)

Processing non-Latin scripts (such as Arabic) introduces morphological complexities:
- Right-to-left (RTL) reading order without uppercase or lowercase orthographic distinctions.
- Proclitics (prepositions such as *bi-*, conjunctions such as *wa-*) joined directly to subsequent nouns.
- In this lab, blank language initialization (`spacy.blank("ar")`) provides baseline rule-based tokenisation without requiring multi-gigabyte neural pipelines, while NLTK `WhitespaceTokenizer` serves as a baseline comparison.

### 5. Sentence Boundary Disambiguation: Statistical Parsing vs. Punkt

Sentence Boundary Disambiguation (SBD) resolves whether punctuation marks (primarily periods) serve as sentence terminators or interior lexical markers.

#### spaCy Dependency-Driven Segmentation
In spaCy, sentence boundaries are identified primarily through syntactic dependency parsing. The transition-based parser determines whether a token represents the root of a grammatical clause or an auxiliary dependent, updating `token.is_sent_start` accordingly. The generator `doc.sents` yields `Span` objects for each detected sentence.

#### NLTK Punkt Unsupervised Model
NLTK provides `sent_tokenize()`, which implements the Punkt sentence boundary detection algorithm (Kiss and Strunk, 2006). Punkt operates without supervised treebanks by evaluating:
1. Type-based classification of period tokens into abbreviations vs. sentence boundaries.
2. Collocation probabilities of subsequent capitalized tokens.
3. Orthographic casing conventions in surrounding contexts.

### 6. Custom Pipeline Component Injection

When domain-specific documents (such as legal statutes or corporate guidelines) treat semicolons (`;`) or specific delimiters as sentence boundaries, spaCy allows injecting custom components before the parser:

```python
from spacy.language import Language

@Language.component("semicolon_sentence_boundary")
def semicolon_sentence_boundary(doc):
    for token in doc[:-1]:
        if token.text == ";":
            doc[token.i + 1].is_sent_start = True
    return doc

nlp.add_pipe("semicolon_sentence_boundary", before="parser")
```

Setting `token.is_sent_start = True` overrides downstream parser heuristics, ensuring deterministic boundary enforcement.

---

## Comparative Evaluation Benchmark

Evaluating tokenisation strategies across representative test sentences:

| Input Text Sample | Python `str.split()` | NLTK `word_tokenize()` | spaCy `nlp()` Pipeline |
| :--- | :--- | :--- | :--- |
| `"Hello Mr.Smith!"` | `["Hello", "Mr.Smith!"]` | `["Hello", "Mr.Smith", "!"]` | `["Hello", "Mr.Smith", "!"]` |
| `"You shouldn't pay $10.30."` | `["You", "shouldn't", "pay", "$10.30."]` | `["You", "should", "n't", "pay", "$", "10.30", "."]` | `["You", "should", "n't", "pay", "$", "10.30", "."]` |
| `"We're moving to L.A."` | `["We're", "moving", "to", "L.A."]` | `["We", "'re", "moving", "to", "L.A."]` | `["We", "'re", "moving", "to", "L.A."]` |
| `"A 5km NYC cab ride cost 10.30$"` | `["A", "5km", "NYC", "cab", "ride", "cost", "10.30$"]` | `["A", "5km", "NYC", "cab", "ride", "cost", "10.30", "$"]` | `["A", "5", "km", "NYC", "cab", "ride", "cost", "10.30", "$"]` |

Evaluating sentence boundary disambiguation strategies:

| Input Text Sample | NLTK `sent_tokenize()` | spaCy `doc.sents` |
| :--- | :--- | :--- |
| `"Dr. Smith arrived at 10.30 a.m. He gave a short talk."` | 2 sentences (`"Dr. Smith arrived at 10.30 a.m."`, `"He gave a short talk."`) | 2 sentences (`"Dr. Smith arrived at 10.30 a.m."`, `"He gave a short talk."`) |
| `"Visit example. com for more information."` | 2 sentences (splits on space after period) | 2 sentences (splits on space after period) |
| `"Management is doing things right; leadership is doing the right things."` | 1 sentence (semicolon not treated as boundary) | 2 sentences (via custom `@Language.component`) |

---

## Execution Instructions

### Prerequisites

- Python 3.10 or higher
- Dedicated virtual environment (`venv`)

### Setup and Environment

1. **Activate the virtual environment:**
   ```bash
   cd 05-spaCy-NLTK-tokenisation
   source .env/bin/activate
   ```

2. **Verify installed packages:**
   ```bash
   pip install spacy nltk
   python -m spacy download en_core_web_sm
   python -c "import nltk; nltk.download('punkt'); nltk.download('punkt_tab')"
   ```

### Running the Notebooks

Launch Jupyter Notebook from the lab directory:

```bash
jupyter notebook main.ipynb
```

You can execute:
- `main.ipynb`: Comprehensive consolidated laboratory notebook.
- `tokenisation/main.ipynb`: Dedicated word and sub-word tokenisation analysis.
- `segmentation/main.ipynb`: Dedicated sentence boundary disambiguation analysis.

---

## Technology Stack

- **Programming Language:** Python 3
- **Industrial NLP Engine:** spaCy (`en_core_web_sm`, `spacy.blank("ar")`, `@Language.component`)
- **Academic NLP Toolkit:** NLTK (`word_tokenize`, `sent_tokenize`, `WhitespaceTokenizer`, Punkt models)
- **Interactive Environment:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
