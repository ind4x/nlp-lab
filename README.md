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

## Module details

### 1. SMS and email spam classification

The text classification laboratory implements an end-to-end supervised NLP
pipeline to detect spam messages within the SMS Spam Collection dataset (5,574
samples).

#### Pipeline workflow
1. **Data cleaning and normalization:** Converts raw text to lowercase and strips
   non-alphabetic characters using regular expressions (`[^a-z]`).
2. **Stopword removal:** Filters tokens against NLTK's English stopword lexicon
   to eliminate non-discriminative syntax.
3. **Feature extraction:** Transforms text into numerical feature vectors using
   Scikit-learn's `CountVectorizer`, generating a bag-of-words token matrix.
4. **Classification:** Trains a `MultinomialNB` classifier applying Laplace
   smoothing ($\alpha = 1.0$) to avoid zero-probability estimates on unseen
   vocabulary.

#### Performance metrics (Test partition: 1,115 samples)

| Metric | Class: Ham (0) | Class: Spam (1) | Overall |
| :--- | :--- | :--- | :--- |
| Precision | 0.99 | 0.97 | - |
| Recall | 1.00 | 0.97 | - |
| F1-score | 1.00 | 0.97 | - |
| Accuracy | - | - | **99.19%** |

#### Confusion matrix

```text
                   Predicted Ham    Predicted Spam
Actual Ham (970)        966                4
Actual Spam (145)         5              140
```

The model achieves a false-positive rate of 0.41% (4 misclassified legitimate
messages out of 970), preserving legitimate communication while intercepting 96.55%
of spam transmissions.

### 2. Multi-layer perceptron for non-linear XOR classification

The neural network laboratory constructs a two-layer feedforward neural network
from first principles in NumPy without reliance on automated differentiation
engines or high-level frameworks (such as PyTorch or TensorFlow).

#### Mathematical architecture
- **Input layer:** 2 features ($x \in \mathbb{R}^{1 \times 2}$) representing the
  binary inputs $x_1, x_2$.
- **Hidden layer:** 2 neurons with logistic sigmoid activation $\sigma(z) =
  \frac{1}{1 + e^{-z}}$, parameterized by $W^{(1)} \in \mathbb{R}^{2 \times 2}$
  and $b^{(1)} \in \mathbb{R}^{1 \times 2}$.
- **Output layer:** 1 neuron with logistic sigmoid activation, parameterized by
  $W^{(2)} \in \mathbb{R}^{2 \times 1}$ and $b^{(2)} \in \mathbb{R}^{1 \times 1}$.
- **Objective function:** Half Mean Squared Error:
  $$L = \frac{1}{2}(\hat{y} - y)^2$$

#### Backpropagation and gradient derivation
Error deltas and parameter gradients are computed via the chain rule:
- Output delta: $\delta^{(2)} = (\hat{y} - y) \odot \sigma'(z^{(2)})$
- Hidden delta: $\delta^{(1)} = (\delta^{(2)} (W^{(2)})^T) \odot \sigma'(z^{(1)})$
- Weight gradients: $\nabla_{W^{(2)}} L = (a^{(1)})^T \delta^{(2)}$, $\nabla_{W^{(1)}} L = x^T \delta^{(1)}$
- Bias gradients: $\nabla_{b^{(2)}} L = \delta^{(2)}$, $\nabla_{b^{(1)}} L = \delta^{(1)}$

#### XOR convergence results

Trained over 10,000 epochs with learning rate $\eta = 1.0$, the modular
`TwoLayerNeuralNetwork` successfully resolves the non-linearly separable XOR
decision boundary:

| $x_1$ | $x_2$ | Target $y$ | Predicted $\hat{y}$ | Decision ($\hat{y} \ge 0.5$) |
| :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0.0526 | 0 |
| 0 | 1 | 1 | 0.9490 | 1 |
| 1 | 0 | 1 | 0.9490 | 1 |
| 1 | 1 | 0 | 0.0566 | 0 |

### 3. Theoretical coursework resources

The `resources` directory includes academic exercises and reference material
covering optimization foundations:
- `Exercice_Regression_Descente_Gradient.pdf`: Multiple linear regression
  problem analyzing student performance factors (study hours, attendance,
  sleep, past grades) via gradient descent.
- `ex1.png`: Step-by-step manual calculation verifying gradient descent weight
  and bias updates across discrete training iterations.

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
