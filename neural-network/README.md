# Multi-Layer Perceptron for Non-Linear XOR Classification

This laboratory constructs a foundational Multi-Layer Perceptron (MLP) from first
principles using pure NumPy. The implementation resolves the non-linearly separable
Exclusive OR (XOR) problem through manual analytical derivations, vectorized matrix
forward passes, chain-rule backward error propagation, and batch gradient descent.

---

## Lab Structure

```text
NeuralNetwork/
├── README.md                          # Technical specification, mathematical derivations, and architecture
└── main.ipynb                         # Vectorized implementation, analytical ground truth check, and XOR training
```

### File Responsibilities

- **`main.ipynb`**: Contains two complementary implementations:
  1. *Analytical Step-by-Step Validation*: Hand-derivation verification for single sample $x=[1, 0]$ with target $y=1$.
  2. *Modular Vectorized Engine (`TwoLayerNeuralNetwork`)*: Production-ready object-oriented network implementation handling batch matrix inputs, sigmoid activation, mean squared error loss, analytical gradient updates, and dynamic loss convergence tracking.

---

## Network Architecture & Mathematical Derivations

### Structural Topology (2-2-1 MLP)

```text
  Input Layer                  Hidden Layer                   Output Layer
 (2 Features)                  (2 Neurons)                     (1 Neuron)

    x_1 o-------------------------> (h_1) o------------------------\
         \                      /          \                        \
          \                    /            \                        v
           x                  x              ---------------------> ( y_hat ) ---> Loss
          /                    \            /
         /                      \          /
    x_2 o-------------------------> (h_2) o------------------------/

               W^(1) in R^{2x2}                    W^(2) in R^{2x1}
               b^(1) in R^{1x2}                    b^(2) in R^{1x1}
```

### Forward Propagation
For an input vector $x \in \mathbb{R}^{1 \times 2}$:
1. **Hidden Pre-Activation:**
   $$z^{(1)} = x W^{(1)} + b^{(1)} \quad \in \mathbb{R}^{1 \times 2}$$
2. **Hidden Activation (Logistic Sigmoid):**
   $$a^{(1)} = \sigma(z^{(1)}) = \frac{1}{1 + e^{-z^{(1)}}}$$
3. **Output Pre-Activation:**
   $$z^{(2)} = a^{(1)} W^{(2)} + b^{(2)} \quad \in \mathbb{R}^{1 \times 1}$$
4. **Prediction:**
   $$\hat{y} = \sigma(z^{(2)}) \quad \in [0, 1]$$
5. **Loss (Half Mean Squared Error):**
   $$L = \frac{1}{2} (\hat{y} - y)^2$$

### Backward Propagation & Parameter Gradients
Applying the multivariable chain rule:
- **Output Error Delta:**
  $$\delta^{(2)} = \frac{\partial L}{\partial z^{(2)}} = (\hat{y} - y) \odot \sigma'(z^{(2)}) = (\hat{y} - y) \cdot \hat{y}(1 - \hat{y})$$
- **Hidden Error Delta:**
  $$\delta^{(1)} = \frac{\partial L}{\partial z^{(1)}} = \left(\delta^{(2)} (W^{(2)})^T\right) \odot \sigma'(z^{(1)}) = \left(\delta^{(2)} (W^{(2)})^T\right) \odot a^{(1)}(1 - a^{(1)})$$
- **Weight & Bias Gradients:**
  $$\nabla_{W^{(2)}} L = (a^{(1)})^T \delta^{(2)}, \quad \nabla_{b^{(2)}} L = \delta^{(2)}$$
  $$\nabla_{W^{(1)}} L = x^T \delta^{(1)}, \quad \nabla_{b^{(1)}} L = \delta^{(1)}$$
- **Gradient Descent Parameter Updates:**
  $$\theta \leftarrow \theta - \eta \cdot \nabla_{\theta} L \quad \text{for } \theta \in \{W^{(1)}, b^{(1)}, W^{(2)}, b^{(2)}\}$$

---

## XOR Convergence Verification

Trained over 10,000 iterations with learning rate $\eta = 1.0$:

| $x_1$ | $x_2$ | Target $y$ | Predicted $\hat{y}$ | Rounded Decision ($\hat{y} \ge 0.5$) | Status |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 0 | 0 | 0 | 0.0526 | **0** | Correct |
| 0 | 1 | 1 | 0.9490 | **1** | Correct |
| 1 | 0 | 1 | 0.9490 | **1** | Correct |
| 1 | 1 | 0 | 0.0566 | **0** | Correct |

The decision boundaries successfully isolate the non-linear XOR operator without requiring specialized deep learning frameworks.

---

## Execution Instructions

### Prerequisites
- Python 3.10+
- NumPy, Pandas, Jupyter

```bash
pip install numpy pandas jupyter
```

### Running the Notebook
From the repository root or the `NeuralNetwork` directory:

```bash
cd NeuralNetwork
jupyter notebook main.ipynb
```

Execute all cells to verify the analytical calculus derivations against Python numerical calculations, followed by the complete XOR convergence benchmark.

---

## Technology Stack

- **Language:** Python 3
- **Numerical Computing:** NumPy (vectorized multidimensional array calculus)
- **Data Structuring:** Pandas
- **Interactive Development:** Jupyter Notebook, VS Code

---

## Author

- **Author:** Youssef Fellah ([ind4x_](https://github.com/ind4x))
- **Institution:** Mundiapolis University, Engineering Cycle
