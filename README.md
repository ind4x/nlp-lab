# Natural language processing laboratory portfolio

This repository contains practical laboratory implementations, mathematical
derivations, and academic reference materials completed as part of the Natural
Language Processing (NLP) and Artificial Intelligence curriculum in the
Engineering Cycle at Mundiapolis University.

The portfolio covers foundational and applied topics in machine learning: raw
text preprocessing, vector space modeling, probabilistic text classification,
and from-scratch neural network implementation using analytical calculus and
vectorized matrix operations.

## Laboratory overview

| Laboratory | Domain | Core methodology | Primary results | Reference |
| :--- | :--- | :--- | :--- | :--- |
| [MultinomialNB](./MultinomialNB) | Text classification | Regex sanitation, stopword filtering, CountVectorizer, Multinomial Naive Bayes | Accuracy: 99.19%<br>Spam F1: 0.97 | [Notebook](./MultinomialNB/main.ipynb) |
| [NeuralNetwork](./NeuralNetwork) | Deep learning foundations | Multi-Layer Perceptron (2-2-1), analytical calculus, backpropagation, batch gradient descent | Exact convergence on XOR truth table | [Notebook](./NeuralNetwork/main.ipynb) |
| [resources](./resources) | Optimization theory | Multiple linear regression, step-by-step MSE gradient derivation | Theoretical verification baseline | [Directory](./resources) |

## Repository structure

```text
.
├── MultinomialNB/
│   ├── README.md              # Pipeline documentation, metrics, and confusion matrix
│   ├── SpamCollection.txt     # SMS Spam Collection dataset (5,574 labeled samples)
│   └── main.ipynb             # Data cleaning, vectorization, training, and evaluation
├── NeuralNetwork/
│   ├── README.md              # Mathematical formulation and analytical derivations
│   └── main.ipynb             # NumPy MLP implementation and XOR convergence verification
├── resources/
│   ├── README.md              # Overview of theoretical problem sets
│   ├── Exercice_Regression_Descente_Gradient.pdf # Gradient descent problem formulation
│   └── ex1.png                # Discrete manual gradient update baseline
└── README.md                  # Main portfolio index and execution guide
```

## Setup and environment

### Prerequisites

- Python 3.10 or higher
- Git

### Installation

1. Clone the repository:
   ```bash
   git clone git@github.com:ind4x/NLP-LAB.git
   cd NLP-LAB
   ```

2. Create and activate a Python virtual environment:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. Install required runtime dependencies:
   ```bash
   pip install numpy pandas scikit-learn nltk jupyter
   ```

4. Download NLTK linguistic corpora:
   ```bash
   python3 -c "import nltk; nltk.download('stopwords')"
   ```

### Running the notebooks

Launch Jupyter Lab or Jupyter Notebook from the root directory:

```bash
jupyter notebook
```

Navigate to:
- `MultinomialNB/main.ipynb` to execute the spam classification pipeline.
- `NeuralNetwork/main.ipynb` to execute the manual analytical derivations and
  XOR network training.

Alternatively, open the repository in VS Code with the Jupyter extension
installed and select the active `.venv` Python interpreter.

## Technology stack

- **Programming language:** Python 3
- **Numerical computing:** NumPy
- **Data analysis:** Pandas
- **Natural language processing:** NLTK, Regular Expressions (`re`)
- **Machine learning:** Scikit-learn (`CountVectorizer`, `MultinomialNB`, `LabelEncoder`, `train_test_split`, `metrics`)
- **Interactive development:** Jupyter Notebook, VS Code

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
