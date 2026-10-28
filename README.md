# Natural Language Processing Laboratory Portfolio

This repository hosts practical implementations, machine learning algorithms, and
analytical architectures developed for the Natural Language Processing (NLP) and
Artificial Intelligence curriculum at Mundiapolis University.

---

## Laboratory Overview

| Laboratory | Domain & Topic | Core Methodology | Primary Results | Reference |
| :--- | :--- | :--- | :--- | :--- |
| **[file-manipulation](./file-manipulation)** | Document Ingestion & Corpus Engineering | File descriptor I/O, cursor seek, UTF-8 multilingual normalization, tabular data (CSV/Excel), PDF stream extraction (PyPDF2), regex lexical matching | **Multi-Format Ingestion Pipeline**<br>Decoded corpora, PDF parsing & merging | [Notebook](./file-manipulation/main.ipynb) |
| **[multinomial-nb](./multinomial-nb)** | Supervised Text & Spam Classification | Regex sanitization, stopword filtering, CountVectorizer sparse modeling, Multinomial Naive Bayes | **Accuracy: 99.19%**<br>Spam F1: 0.97 | [Notebook](./multinomial-nb/main.ipynb) |
| **[neural-network](./neural-network)** | Deep Learning Foundations from Scratch | 2-2-1 Multi-Layer Perceptron, vectorized NumPy calculus, analytical backpropagation via chain rule | **Exact XOR Separation**<br>(MSE $\to$ 0) | [Notebook](./neural-network/main.ipynb) |
| **[resources](./resources)** | Theoretical Optimization Foundations | Multiple linear regression, analytical MSE gradient derivation, discrete iteration updates | Theoretical Verification Baseline | [Directory](./resources) |

---

## General Running Instructions

### Prerequisites

- **Python:** 3.10 or higher
- **Git**
- **Virtual Environment Tooling:** `venv`

### Setup and Environment

1. **Clone the repository:**
   ```bash
   git clone git@github.com:ind4x/nlp-lab.git
   cd nlp-lab
   ```

2. **Create and activate a virtual environment:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Install required dependencies:**
   ```bash
   pip install numpy pandas scikit-learn nltk openpyxl PyPDF2 jupyter
   ```

4. **Download NLTK linguistic corpora:**
   ```bash
   python3 -c "import nltk; nltk.download('stopwords')"
   ```

### Executing Laboratories

Launch Jupyter Notebook from the repository root:

```bash
jupyter notebook
```

Navigate to:
- `file-manipulation/main.ipynb` to execute data ingestion, encoding verification, PDF stream parsing, and regex extraction.
- `multinomial-nb/main.ipynb` to execute the spam classification pipeline and review metrics.
- `neural-network/main.ipynb` to verify analytical derivations and run XOR network training.

---

## Technology Stack

- **Programming Language:** Python 3
- **Numerical Computing:** NumPy
- **Data Engineering:** Pandas
- **Document & Spreadsheet Processing:** PyPDF2, Openpyxl
- **Natural Language Processing:** NLTK, Regular Expressions (`re`)
- **Machine Learning:** Scikit-learn (`CountVectorizer`, `MultinomialNB`, `LabelEncoder`, `metrics`)
- **Interactive Development:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
