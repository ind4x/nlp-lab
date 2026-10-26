# Natural Language Processing Laboratory Portfolio

This repository hosts practical implementations, machine learning algorithms, and
analytical architectures developed for the Natural Language Processing (NLP) and
Artificial Intelligence curriculum at Mundiapolis University.

---

## Laboratory Overview

| Laboratory | Domain & Topic | Core Methodology | Primary Results | Reference |
| :--- | :--- | :--- | :--- | :--- |
| **[MultinomialNB](./MultinomialNB)** | Supervised Text & Spam Classification | Regex sanitization, stopword filtering, CountVectorizer sparse modeling, Multinomial Naive Bayes | **Accuracy: 99.19%**<br>Spam F1: 0.97 | [Notebook](./MultinomialNB/main.ipynb) |
| **[NeuralNetwork](./NeuralNetwork)** | Deep Learning Foundations from Scratch | 2-2-1 Multi-Layer Perceptron, vectorized NumPy calculus, analytical backpropagation via chain rule | **Exact XOR Separation**<br>(MSE $\to$ 0) | [Notebook](./NeuralNetwork/main.ipynb) |
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
   pip install numpy pandas scikit-learn nltk jupyter
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
- `MultinomialNB/main.ipynb` to execute the spam classification pipeline and review metrics.
- `NeuralNetwork/main.ipynb` to verify analytical derivations and run XOR network training.

---

## Technology Stack

- **Programming Language:** Python 3
- **Numerical Computing:** NumPy
- **Data Engineering:** Pandas
- **Natural Language Processing:** NLTK, Regular Expressions (`re`)
- **Machine Learning:** Scikit-learn (`CountVectorizer`, `MultinomialNB`, `LabelEncoder`, `metrics`)
- **Interactive Development:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
