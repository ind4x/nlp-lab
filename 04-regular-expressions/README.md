# Advanced Regular Expressions for NLP and Text Engineering

This laboratory explores advanced regular expressions (`re`) and lexical pattern matching for Natural Language Processing (NLP) and text engineering pipelines. The implementation covers verbose pattern compilation, named capture groups, zero-width assertions (lookaheads and lookbehinds), dynamic regex substitutions via callable dispatch handlers, backreference-based lexical deduplication, and parsing nested multi-line semi-structured text blocks into relational tabular schemas.

---

## Lab Structure

```text
04-regular-expressions/
├── README.md                          # Technical specification, regex architecture, and methodologies
├── docs/                              # Course specifications and assignment briefs
│   └── TP2_Regex.pdf                  # Laboratory specification by Pr. Zouhair ELAMRANI ABOU ELASSAD
├── files/                             # Raw input datasets and source text corpora
│   ├── exo1_server.log                # Web server log stream with heterogeneous formats and corruption
│   ├── exo2_contacts.txt              # Unstructured contact directory with non-standardized formats
│   ├── exo3_article.html              # Raw web article export with markup, comments, and mixed dates
│   └── exo4_orders.txt                # Multi-line purchase order transaction blocks
├── exo3_article_clean.txt             # Sanitized article with ISO dates and stripped tags
├── order_items.csv                    # Granular order line items export
├── orders.csv                         # Consolidated order metadata and totals export
└── main.ipynb                         # Interactive notebook with complete solutions and execution
```

### File Responsibilities

- **`main.ipynb`**: Interactive notebook containing complete, verified solutions for all four laboratory exercises.
- **`docs/TP2_Regex.pdf`**: Academic assignment specification formulated by Pr. Zouhair ELAMRANI ABOU ELASSAD.
- **`files/`**: Storage directory holding input corpora, raw server logs, unstructured directory lines, markup documents, and nested order records.
- **`exo3_article_clean.txt`**: Persisted output of the document sanitization pipeline featuring normalized ISO 8601 dates and stripped HTML markup.
- **`orders.csv`**: Structured relational export mapping order identifiers, execution dates, customer entities, delivery statuses, and monetary totals.
- **`order_items.csv`**: Normalized item-level relational export mapping order foreign keys, item quantities, product descriptions, unit costs, and line totals.

---

## Technical Methodologies & Pattern Architecture

```text
+-----------------------------------------------------------------------------------+
|                            Raw Semi-Structured Text                               |
|   +--------------------+   +--------------------+   +-------------------------+   |
|   | Server Event Logs  |   | Contact Directory  |   | HTML Documents & Blocks |   |
|   |  (exo1_server.log) |   | (exo2_contacts.txt)|   | (exo3_article, exo4)    |   |
|   +--------------------+   +--------------------+   +-------------------------+   |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                          Lexical Processing & Extraction                          |
|  - Multi-line verbose compilation (re.VERBOSE | re.MULTILINE)                      |
|  - Named capture groups (?P<name>...) & groupdict mapping                         |
|  - Strict octet & protocol validation (bounded IP regexes)                        |
|  - Zero-width lookaround assertions: (?<=...), (?<!...), (?=...), (?!...)         |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                       Transformation & Normalization Engine                       |
|  - Callback-driven substitutions: re.sub(pattern, callable, text)                 |
|  - Polymorphic date parsing to ISO 8601 (YYYY-MM-DD)                              |
|  - Consecutive token deduplication via backreferences (\b(\w+)\s+\1\b)           |
|  - Privacy-preserving identifier masking (local-part email redaction)             |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                        Structured Relational Serialization                        |
|  - Filtered frequency analysis via collections.Counter                            |
|  - Deterministic CSV tabular export (csv.DictWriter / csv.DictReader)             |
|  - Cleaned text corpora serialization                                             |
+-----------------------------------------------------------------------------------+
```

### 1. Verbose Pattern Compilation & Named Group Extraction

Complex patterns become unreadable and error-prone when compacted into single-line strings. The `re.VERBOSE` flag (`re.X`) ignores unescaped whitespace and enables inline comments (`#`), allowing pattern logic to be documented structurally. Note that literal hash characters inside verbose regular expressions must be explicitly escaped (`\#`) to avoid commenting out downstream pattern segments:

```python
LOG_PATTERN = re.compile(
    r"""
    ^(?P<date>\d{4}-\d{2}-\d{2})           # ISO date (YYYY-MM-DD)
    \s+
    (?P<time>\d{2}:\d{2}:\d{2})           # Timestamp (HH:MM:SS)
    \s+
    \[(?P<level>[A-Za-z]+)\]               # Case-insensitive log level
    \s+
    (?P<ip>\S+)                             # Raw IP candidate
    \s+-\s+
    (?P<message>.*)$                         # Message body
    """,
    re.VERBOSE | re.IGNORECASE
)
```

Matched spans are extracted directly into key-value dictionaries using `match.groupdict()`, decoupling positional indexing from semantic token retrieval.

### 2. Deterministic Bounded Validation (IPv4 & Email RFC Standards)

Simple wildcards such as `\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}` allow invalid numbers like `999.12.1.4` or non-standard leading zeros like `010.0.0.7`. Strict token validation bounds individual numerical octets to the range $[0, 255]$:

$$\text{Octet} \in [0, 255] \implies (25[0-5] \mid 2[0-4]\d \mid 1\d{2} \mid [1-9]?\d)$$

Similarly, RFC-compliant email validation requires:
- Local part starting and ending with an alphanumeric character without contiguous separators (`..`, `__`).
- Hostname labels separated by single periods.
- Top-level domain (TLD) containing at least two alphabetical characters.

### 3. Zero-Width Assertions & Lookaround Boundary Enforcement

Lookaround assertions match positions rather than consuming characters, preserving string context:
- **Positive Lookbehind `(?<=...)`:** Extracts names located strictly after `Name:\s` without capturing the label.
- **Positive Lookahead `(?=...)`:** Extracts percentages `\b(\d+)(?=%)` and isolates domain names without retaining punctuation.
- **Negative Lookbehind `(?<!\S)` & Lookahead `(?=[.,;]?(?:\s|$))`:** Isolates monetary amounts (`$1,299,450.75` and `45,300.50 EUR`) avoiding partial numerical splits or trailing sentence punctuation consumption.

### 4. Dynamic Substitution with Functional Callbacks

While standard string substitutions replace tokens with static templates, `re.sub()` accepts a callable dispatch handler `f(match) -> str`. This allows arbitrary programmatic transformations within a single regex pass:

```python
def normalize_date(m):
    if m.group("iso"):
        return m.group("iso")
    elif m.group("dmy"):
        d, mon, y = m.group("d_day"), m.group("d_mon"), m.group("d_yr")
        return f"{y}-{int(mon):02d}-{int(d):02d}"
    elif m.group("text_date"):
        d = int(m.group("t_day"))
        mon = months[m.group("t_mon").lower()]
        y = m.group("t_yr")
        return f"{y}-{mon}-{d:02d}"

final_text = DATE_PATTERN.sub(safe_normalize_date, text)
```

### 5. Grammatical Stutter Elimination via Backreferences

Corpus ingestion from OCR, manual typing, or voice recognition frequently introduces consecutive duplicate words (e.g., `"The the"`, `"was was"`). Using capturing groups and backreferences `\1` combined with word boundaries `\b`:

```python
text_dedup = re.sub(r"\b(\w+)\s+\1\b", r"\1", text, flags=re.IGNORECASE)
```

This ensures case-insensitive matching while preserving the casing of the first encountered token.

### 6. Multi-Line Transaction Parsing & Relational CSV Serialization

Parsing hierarchical documents (e.g., order headers followed by repeated line items) requires two coordinated patterns:
1. An outer pattern compiled with `re.MULTILINE | re.VERBOSE` capturing block boundaries from header to status.
2. An inner pattern iterating over the captured items block to extract quantity, product description, and unit price.

Data is subsequently serialized using Python `csv.DictWriter` to guarantee correct quotation escaping (e.g., products containing dimension indicators like `27" Monitor`).

---

## Laboratory Exercises & Results

### Exercise 1: Web Server Log Analysis (`exo1_server.log`)

The ingestion pipeline inspects 14 raw log records, categorizes entries by log severity, isolates corrupted records, and validates IPv4 strings.

- **Total Lines Ingested:** 14
- **Valid Structured Entries:** 13
- **Corrupted Lines:** 1 (`Line 8: "corrupted line without proper format"`)
- **Invalid IP Addresses Identified:** 3 (`010.0.0.7`, `256.1.1.1`, `999.12.1.4`)
- **Severity Distribution:** `ERROR: 6`, `INFO: 3`, `WARNING: 2`, `DEBUG: 1`, `CRITICAL: 1`
- **Error Distribution per Valid IP:**
  - `192.168.1.23`: 2
  - `10.0.0.5`: 2
  - `172.16.4.8`: 1
- **Extracted Error Identifiers:** `[('query_id', 'Q-4521'), ('query_id', 'Q-4533'), ('code', 'E-107'), ('query_id', 'Q-4602'), ('code', 'E-210'), ('code', 'E-300')]`
- **Database Timeout Metrics:** Average latency of 29.0s evaluated over 3 timeout occurrences.

### Exercise 2: Contact Directory Validation & Normalization (`exo2_contacts.txt`)

Validates heterogeneous contact strings, normalizes Moroccan phone numbers (+212 / 00212 / 0 prefix) to international E.164-like notation `+212XXXXXXXXX`, filters out invalid email/phone patterns, masks sensitive local parts for privacy compliance, and aggregates domain occurrences.

- **Total Contacts Extracted:** 9
- **Accepted Valid Contacts:** 6
  - `Amine El Fassi` (`amine.elfassi@univ-cadi.ac.ma`, `+212612345678`)
  - `Salma Benali` (`salma_benali@gmail.com`, `+212661234567`)
  - `Yassine Idrissi` (`y.idrissi@outlook.fr`, `+212661234567`)
  - `Lina O'Brien` (`lina.obrien@my-startup.io`, `+212698765432`)
  - `Kenza Alaoui` (`kenza.alaoui@gmail.com`, `+212711223344`)
  - `Hamza Berrada` (`hamza@univ-cadi.ac.ma`, `+212700112233`)
- **Rejected Records & Failure Root Cause:**
  - `John Smith`: Invalid email syntax
  - `Bad Entry`: Invalid email syntax, invalid phone syntax
  - `Omar Tazi`: Invalid email syntax
- **Email Masking:** Outputs format `a***@univ-cadi.ac.ma` via regex callback substitution.
- **Domain Distribution:** `univ-cadi.ac.ma`: 2, `gmail.com`: 2, `outlook.fr`: 1, `my-startup.io`: 1.

### Exercise 3: HTML Corpus Sanitization & Entity Extraction (`exo3_article.html`)

Sanitizes raw HTML document exports, eliminates multi-line comments and markup tags, normalizes whitespace, eliminates duplicate consecutive tokens, unifies mixed date conventions to ISO 8601 (`YYYY-MM-DD`), and extracts financial currencies and percentage metrics.

- **Stripped Artifacts:** HTML comments (`<!-- ... -->` multi-line), markup tags (`<h1>`, `<p>`), redundant whitespace.
- **Lexical Deduplication:** Repetitive occurrences (`The the` $\to$ `The`, `was was` $\to$ `was`) eliminated.
- **Date Standardization:**
  - `03/11/2015` $\to$ `2015-11-03`
  - `7 March 2016` $\to$ `2016-03-07`
  - Existing ISO dates (`2026-04-18`, `2026-12-21`) preserved intact.
- **Extracted Financial Values:**
  - `('USD', 1299450.75)`
  - `('USD', 980200.0)`
  - `('EUR', 45300.5)`
- **Extracted Statistical Percentages:** `[94, 88]`
- **Corpus Persistence:** Sanitized plain text saved to `exo3_article_clean.txt`.

### Exercise 4: Semi-Structured Text to Relational CSV Serialization (`exo4_orders.txt`)

Parses multi-line order transaction blocks into two relational database tables (`orders.csv` and `order_items.csv`), computing line totals and validating numeric integrity.

- **Orders Parsed:** 4 orders identified.
- **Relational Tables Generated:**
  - `orders.csv`: 4 rows (`order_id`, `date`, `customer`, `email`, `status`, `nb_items`, `total`).
  - `order_items.csv`: 8 line items (`order_id`, `qty`, `product`, `unit_price`, `line_total`).
- **Quote Character Handling:** Successfully preserves double quotes in item names (`"27"" Monitor"`).
- **Status Breakdown:** `SHIPPED`: 1, `DELIVERED`: 1, `CANCELLED`: 1, `PENDING`: 1.
- **Net Revenue (Excluding CANCELLED):** $509.88.

---

## Execution Instructions

### Prerequisites

The implementation relies strictly on the Python Standard Library. No external third-party dependencies are required:

- Python 3.10 or higher
- Standard modules: `re`, `csv`, `collections`, `io`

### Running the Notebook

From the repository root or the `04-regular-expressions` directory:

```bash
cd 04-regular-expressions
jupyter notebook main.ipynb
```

Execute all notebook cells sequentially to reproduce the parsing, validation, transformation, and CSV serialization pipelines.

---

## Technology Stack

- **Programming Language:** Python 3
- **Regular Expressions:** `re` (`re.VERBOSE`, `re.MULTILINE`, `re.DOTALL`, `re.IGNORECASE`)
- **Data Structures & Aggregations:** `collections.Counter`
- **Tabular Serialization:** `csv` (`DictWriter`, `DictReader`)
- **Interactive Development:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Instructor:** Pr. Zouhair ELAMRANI ABOU ELASSAD
- **Institution:** Mundiapolis University, Engineering Cycle
