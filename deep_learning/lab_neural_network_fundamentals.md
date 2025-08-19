# Lab 3: Neural Network Fundamentals

## Objectives
*   Understand the architecture of a basic neural network.
*   Define and explain the components: layers, neurons, and activation functions.
*   Describe and implement the feedforward process in a neural network.

## Prerequisites
*   Familiarity with the Linux command line.
*   Basic understanding of Python programming.
*   Conceptual knowledge of algebra.

## Tools Required
*   Ubuntu Linux
*   Python 3.x
*   NumPy
*   Matplotlib
*   Jupyter Notebook

---

### System Setup and Installation

Before starting the lab, you need to install the required tools. Open your terminal and run the following commands one by one.

**1. Update Package Lists**

First, update your package manager's list of available packages.
```bash
sudo apt-get update
```

**2. Install Python and Pip**

Most Ubuntu systems come with Python pre-installed. You can verify this by running `python3 --version`. If it's not installed, or if you need the package manager `pip`, run the following command.
```bash
sudo apt-get install -y python3-pip
```

**3. Install Python Libraries**

Now, use `pip` to install NumPy, Matplotlib, and Jupyter Notebook.
```bash
pip3 install numpy matplotlib jupyter
```

**4. Start Jupyter Notebook**

Once the installation is complete, you can start the Jupyter Notebook server.
```bash
jupyter notebook
```

This will open a new tab in your web browser. From the file navigator, click on "New" and select "Python 3 (ipykernel)" to create a new notebook to work in.

---

## Lab Tasks

### Task 1: Diagram a Basic Neural Network

#### 1.1 Create a Basic Diagram

**Objective:** Illustrate the structure of a simple neural network with an input layer, a hidden layer, and an output layer.

**Explanation:**

*   **Neural Network:** A computational model inspired by the human brain, consisting of interconnected nodes called neurons that process and transmit information.
*   **Layers:** Neurons are organized into layers. Every neural network has an **input layer**, one or more **hidden layers**, and an **output layer**.
    *   The **input layer** receives the initial data.
    *   **Hidden layers** perform intermediate computations and transformations. The "deep" in "deep learning" refers to having multiple hidden layers.
    *   The **output layer** produces the final result or prediction.

**Example:**

Let's visualize a simple network with 3 input nodes, 4 hidden nodes, and 2 output nodes. The lines represent connections between neurons, each having an associated weight that determines the strength of the connection.

**ASCII Art Representation:**

```
  Input Layer          Hidden Layer         Output Layer
  (3 neurons)          (4 neurons)          (2 neurons)

    [I1] -------------- [H1] \
      \  \            /   /   \
       \  -----------    /      ---- [O1]
        \           \  /      /
    [I2] ------------- [H2]   /
         \          /  \     /
          ---------    /----
         /          \ /       \
    [I3] ------------- [H3]       ---- [O2]
        \           /  \     /
         -----------    \   /
                      \  \ /
                       - [H4]
```

---

### Task 2: Define Layers, Neurons, and Activation Functions

#### 2.1 Define Layers

**Objective:** Describe the different types of layers and represent them using code.

In a neural network, data is processed layer by layer. We can represent the data and the layers using matrices (or arrays) with NumPy.

**Example Code:**

Copy the following code into a cell in your Jupyter Notebook and run it.

```python
import numpy as np

# The input layer receives data. Let's imagine we have a single data sample
# with 3 features: [temperature, humidity, pressure].
# We represent this as a NumPy array.
input_features = np.array([[25, 60, 1013]]) # A 1x3 matrix (1 sample, 3 features)

# We define the size (number of neurons) for our hidden and output layers.
hidden_layer_size = 4
output_layer_size = 2

print("Input Features (1x3 Matrix):")
print(input_features)
print(f"\nHidden Layer will have {hidden_layer_size} neurons.")
print(f"Output Layer will have {output_layer_size} neurons.")
```

#### 2.2 Neurons

**Objective:** Explain the role of a neuron.

**Explanation:**

A neuron is the fundamental processing unit of a neural network. It performs two main steps:
1.  **Weighted Sum:** It calculates a weighted sum of all its inputs. Each input connection has a "weight," which is a number that the input value is multiplied by. A "bias" term is also added.
2.  **Activation:** It passes the result of the weighted sum through an **activation function**. This function introduces non-linearity, allowing the network to learn complex patterns.

**Snippet:**

Let's simulate how a single neuron processes input. The `np.dot()` function performs a dot product, which is a way to calculate the weighted sum efficiently for all neurons in a layer at once.

```python
# A neuron takes inputs, processes them, and generates an output.
# Let's define an activation function first. The sigmoid is a popular choice.
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

# Each connection has a weight. We'll initialize them randomly for now.
# Weights from 3 input neurons to 4 hidden neurons form a 3x4 matrix.
weights_input_to_hidden = np.random.rand(3, hidden_layer_size)

# 1. Calculate the weighted sum for the hidden layer
hidden_layer_input = np.dot(input_features, weights_input_to_hidden)

# 2. Apply the activation function
hidden_layer_output = sigmoid(hidden_layer_input)

print("Random Weights (Input -> Hidden):")
print(weights_input_to_hidden)
print("\nHidden Layer Input (Weighted Sum):")
print(hidden_layer_input)
print("\nHidden Layer Output (After Activation):")
print(hidden_layer_output)
```

#### 2.3 Activation Functions

**Objective:** Explore common activation functions.

**Key Concepts:**

Activation functions decide whether a neuron should be "activated" or not. They add non-linear properties to the network, which is crucial for learning complex data patterns.

*   **Sigmoid:** Maps any value to a range between 0 and 1. Useful for output layers in binary classification tasks.
*   **ReLU (Rectified Linear Unit):** Returns `0` for any negative input and the input value itself for any positive input. It is the most common activation function for hidden layers due to its efficiency.
*   **Tanh (Hyperbolic Tangent):** Similar to sigmoid, but maps values to a range between -1 and 1.

**Snippet:**

Let's define and visualize these functions.

```python
import matplotlib.pyplot as plt

# We already defined sigmoid. Let's define ReLU.
def relu(x):
    return np.maximum(0, x)

# Let's see the output of ReLU on the same hidden layer input.
relu_output = relu(hidden_layer_input)

print("Output from Sigmoid activation:")
print(hidden_layer_output)
print("\nOutput from ReLU activation:")
print(relu_output)

# Visualize the activation functions
x = np.linspace(-10, 10, 100)
fig, ax = plt.subplots(1, 2, figsize=(12, 4))

ax[0].plot(x, sigmoid(x))
ax[0].set_title('Sigmoid Activation Function')
ax[0].grid(True)

ax[1].plot(x, relu(x))
ax[1].set_title('ReLU Activation Function')
ax[1].grid(True)

plt.show()
```

---

### Task 3: Explain and Implement the Feedforward Process

#### 3.1 Define the Feedforward Process

**Objective:** Describe how data flows through the neural network.

**Explanation:**

The **feedforward process** is the journey of data from the input layer to the output layer. The term "feedforward" means the data moves in only one direction—forward—through the network's layers. There are no loops.

The process is as follows:
1.  The input layer receives the data.
2.  The data is passed to the first hidden layer. Each neuron in this layer calculates its weighted sum and applies an activation function.
3.  The output of the first hidden layer becomes the input for the next hidden layer (if any).
4.  This continues until the data reaches the output layer, which produces the network's final prediction.

#### 3.2 Implement the Feedforward Process

**Objective:** Implement a complete, basic feedforward pass using Python and NumPy.

**Code Example:**

This code combines everything we've learned into a single function that simulates a full pass of data through our 3-4-2 network.

```python
# We'll use our previously defined sigmoid function
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

# Full feedforward function
def feedforward(input_data, weights_input_hidden, weights_hidden_output):
    # Step 1: Calculate output of the hidden layer
    hidden_input = np.dot(input_data, weights_input_hidden)
    hidden_output = sigmoid(hidden_input)
    
    # Step 2: Calculate the output of the final layer
    final_input = np.dot(hidden_output, weights_hidden_output)
    final_output = sigmoid(final_input)
    
    return final_output

# --- Setup ---
# Define layer sizes
input_layer_size = 3
hidden_layer_size = 4
output_layer_size = 2

# Define input data (1 sample, 3 features)
input_features = np.array([[25, 60, 1013]])

# Initialize weights with random values for demonstration
# Weights for connections from input layer (3) to hidden layer (4)
weights_input_hidden = np.random.rand(input_layer_size, hidden_layer_size)

# Weights for connections from hidden layer (4) to output layer (2)
weights_hidden_output = np.random.rand(hidden_layer_size, output_layer_size)


# --- Execution ---
# Perform the feedforward pass
output = feedforward(input_features, weights_input_hidden, weights_hidden_output)

print("--- Feedforward Process ---")
print("Input Features:\n", input_features)
print("\nWeights (Input -> Hidden):\n", weights_input_hidden)
print("\nWeights (Hidden -> Output):\n", weights_hidden_output)
print("\nFinal Feedforward Output:\n", output)

```
The output is a `1x2` matrix, representing the two values produced by the two neurons in the output layer. In a real-world scenario, these values would represent a prediction, such as the probability of two different outcomes.

---

## Conclusion

**Recap:**

In this lab, you have:
*   Visualized the basic three-layer architecture of a neural network.
*   Defined and represented layers, neurons, and activation functions using NumPy.
*   Implemented a simple feedforward process to understand how data flows through a network to generate a prediction.

This feedforward pass is the core mechanism for making predictions with a neural network. The next logical step, which is beyond the scope of this lab, is "backpropagation," the process by which a network learns by adjusting its weights based on the error in its predictions.

**Further Reading:**

To deepen your understanding, research real-world neural network applications such as image recognition (e.g., Convolutional Neural Networks - CNNs) and natural language processing (e.g., Recurrent Neural Networks - RNNs).
