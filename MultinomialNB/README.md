# Spam Classification with Multinomial Naive Bayes

This laboratory implements an end-to-end supervised natural language processing (NLP)
pipeline to classify text communications as either legitimate (*ham*) or unsolicited (*spam*).
The project demonstrates raw text preprocessing, vocabulary tokenization, bag-of-words
sparse vectorization, and probabilistic Bayesian inference.

---

## Lab Structure

```text
MultinomialNB/
├── README.md                          # Technical specification, pipeline architecture, and metrics
├── SpamCollection.txt                 # Labeled SMS/email dataset (5,574 samples, TSV format)
└── main.ipynb                         # Complete interactive pipeline implementation and evaluation
```

### File Responsibilities

- **`main.ipynb`**: Interactive notebook executing data ingestion, exploratory label balance analysis, regex sanitation, NLTK stopword filtering, sparse feature matrix construction via `CountVectorizer`, `MultinomialNB` model training, holdout evaluation, confusion matrix generation, and arbitrary text inference.
- **`SpamCollection.txt`**: Standard benchmark corpus consisting of 5,574 labeled communications (4,825 ham, 747 spam).

---

## Pipeline Architecture & Methodology

The classification pipeline processes unstructured text through five distinct stages:

```text
+-----------------------+
|  Raw Text Message     |  e.g., "Congratulations! You won a $1000 prize. Call now!"
+-----------------------+
            |
            v
+-----------------------+
|  Text Normalization   |  - Lowercase conversion
|  & Regex Sanitization |  - Strip non-alphabetical tokens: re.sub('[^a-z]', ' ', text)
+-----------------------+
            |
            v
+-----------------------+
|  Stopword Filtering   |  - Tokenize into discrete words
|  (NLTK English Lex.)  |  - Remove non-discriminative syntax (e.g., "you", "a", "now")
+-----------------------+
            |
            v
+-----------------------+
|  Feature Extraction   |  - Build vocabulary from training corpus
|  (CountVectorizer)    |  - Transform message into high-dimensional sparse vector: x in R^{1 x V}
+-----------------------+
            |
            v
+-----------------------+
|  Multinomial NB       |  - Compute posterior: P(Spam | x) proportional to P(Spam) prod P(w_j | Spam)
|  Classifier           |  - Laplace smoothing (alpha = 1.0) prevents zero-frequency zero-out
+-----------------------+
            |
            v
+-----------------------+
|  Class Prediction     |  Output: 1 (Spam) or 0 (Ham)
+-----------------------+
```

### Mathematical Formulation
The classifier assigns class $c^* \in \{\text{ham}, \text{spam}\}$ using Bayes' rule under conditional independence assumptions:

$$c^* = \arg\max_{c \in \{0, 1\}} P(c) \prod_{j=1}^{V} P(w_j \mid c)^{x_j}$$

Where conditional word likelihoods utilize Laplace additive smoothing ($\alpha = 1$):

$$P(w_j \mid c) = \frac{N_{c, j} + \alpha}{N_c + \alpha \cdot |V|}$$

---

## Experimental Results

Evaluated on an independent holdout test partition of 1,115 samples (80/20 train-test split):

| Class | Precision | Recall | F1-Score | Support |
| :--- | :--- | :--- | :--- | :--- |
| **Ham (0)** | 0.99 | 1.00 | 1.00 | 970 |
| **Spam (1)** | 0.97 | 0.97 | 0.97 | 145 |
| **Overall Accuracy** | - | - | **99.19%** | 1,115 |

### Confusion Matrix

```text
                   Predicted Ham    Predicted Spam
Actual Ham (970)        966                4
Actual Spam (145)         5              140
```

- **False Positive Rate:** 0.41% (only 4 legitimate communications intercepted).
- **False Negative Rate:** 3.45% (intercepts 96.55% of all spam transmissions).

---

## Execution Instructions

### Prerequisites
Ensure dependencies and NLTK stopwords are present:

```bash
pip install numpy pandas scikit-learn nltk jupyter
python3 -c "import nltk; nltk.download('stopwords')"
```

### Running the Notebook
From the repository root or the `MultinomialNB` directory:

```bash
cd MultinomialNB
jupyter notebook main.ipynb
```

Execute all cells sequentially to run data ingestion, text preprocessing, model fitting, and evaluation benchmarks.

---

## Technology Stack

- **Language:** Python 3
- **Data Engineering:** Pandas, NumPy
- **Natural Language Processing:** NLTK, Regular Expressions (`re`)
- **Machine Learning:** Scikit-learn (`CountVectorizer`, `MultinomialNB`, `LabelEncoder`, `metrics`)

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
