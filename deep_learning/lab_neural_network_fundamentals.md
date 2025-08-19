# Lab 3: Neural Network Fundamentals (with Virtual Environment)

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

### Part 1: System and Environment Setup

Before starting the lab, we will set up our system and a dedicated virtual environment for our project.

**1. Update Package Lists**

First, open your terminal and update your package manager's list of available packages.
```bash
sudo apt-get update
```

**2. Install Core Python Tools**

Most Ubuntu systems have Python pre-installed. These commands ensure you have `pip` (the Python package installer) and `venv` (the tool for creating virtual environments).
```bash
sudo apt-get install -y python3-pip python3-venv
```

**3. Create a Project Directory**

It's good practice to have a dedicated folder for your project.
```bash
mkdir ~/neural_network_lab
cd ~/neural_network_lab
```

**4. Create a Virtual Environment**

Inside your project directory, create a virtual environment. We will name it `venv`.
```bash
python3 -m venv venv
```
This command creates a `venv` directory containing a private copy of the Python interpreter and its libraries.

**5. Activate the Virtual Environment**

To use the virtual environment, you must activate it.
```bash
source venv/bin/activate
```
You will know it's active because your terminal prompt will change to show `(venv)` at the beginning, like this:
`(venv) user@hostname:~/neural_network_lab$`

**Important:** From this point on, all commands should be run with the virtual environment active. Any packages you install will be placed inside the `venv` folder, not on your global system.

**6. Install Python Libraries into the Virtual Environment**

Now, use `pip` to install the required libraries. Notice we use `pip` instead of `pip3`—inside the activated environment, `pip` points to the correct version.
```bash
pip install numpy matplotlib jupyter
```

**7. Start Jupyter Notebook**

With the libraries installed in your environment, you can now start the Jupyter Notebook server.
```bash
jupyter notebook
```
This will open a new tab in your web browser, running from your `neural_network_lab` directory. From the file navigator, click on **New** and select **Python 3 (ipykernel)** to create a new notebook to work in.

---

### Part 2: Lab Tasks

*Now, you can proceed with the lab tasks inside the Jupyter Notebook you just created. Copy and paste the code from each task into a cell in the notebook and run it.*

#### Task 1: Diagram a Basic Neural Network

##### 1.1 Create a Basic Diagram

**Objective:** Illustrate the structure of a simple neural network.

**Explanation:**

*   **Neural Network:** A computational model inspired by the human brain, consisting of interconnected nodes called neurons.
*   **Layers:** Neurons are organized into an **input layer** (receives data), one or more **hidden layers** (performs computations), and an **output layer** (produces the result).

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

#### Task 2: Define Layers, Neurons, and Activation Functions

##### 2.1 Define Layers

**Objective:** Describe the different types of layers and represent them using code.

```python
import numpy as np

# Input data with 3 features (e.g., temperature, humidity, pressure).
# We represent this as a 1x3 matrix (1 sample, 3 features).
input_features = np.array([[25, 60, 1013]])

# Define the size (number of neurons) for our hidden and output layers.
hidden_layer_size = 4
output_layer_size = 2

print("Input Features (1x3 Matrix):")
print(input_features)
print(f"\nHidden Layer will have {hidden_layer_size} neurons.")
print(f"Output Layer will have {output_layer_size} neurons.")
```

##### 2.2 Neurons

**Objective:** Explain the role of a neuron.

**Explanation:** A neuron takes multiple weighted inputs, adds a bias, and then passes the result through an **activation function** to produce an output.

```python
# The sigmoid activation function maps any value to a range between 0 and 1.
def sigmoid(x):
    return 1 / (1 + np.exp(-x))

# Weights are initialized randomly. A 3x4 matrix connects the 3 input neurons
# to the 4 hidden neurons.
weights_input_to_hidden = np.random.rand(3, hidden_layer_size)

# 1. Calculate the weighted sum for the hidden layer using a dot product.
hidden_layer_input = np.dot(input_features, weights_input_to_hidden)

# 2. Apply the activation function.
hidden_layer_output = sigmoid(hidden_layer_input)

print("Random Weights (Input -> Hidden):")
print(weights_input_to_hidden)
print("\nHidden Layer Input (Weighted Sum):")
print(hidden_layer_input)
print("\nHidden Layer Output (After Activation):")
print(hidden_layer_output)
```

##### 2.3 Activation Functions

**Objective:** Explore common activation functions.

**Key Concepts:**
*   **Sigmoid:** Maps values to (0, 1).
*   **ReLU (Rectified Linear Unit):** Returns `max(0, x)`. It is computationally efficient and the most common choice for hidden layers.
*   **Tanh (Hyperbolic Tangent):** Maps values to (-1, 1).

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
x_vals = np.linspace(-10, 10, 100)
fig, ax = plt.subplots(1, 2, figsize=(12, 4))

ax[0].plot(x_vals, sigmoid(x_vals))
ax[0].set_title('Sigmoid Activation Function')
ax[0].grid(True)

ax[1].plot(x_vals, relu(x_vals))
ax[1].set_title('ReLU Activation Function')
ax[1].grid(True)

plt.show()
```

---

#### Task 3: Explain and Implement the Feedforward Process

##### 3.1 Define the Feedforward Process

**Objective:** Describe how data flows through the network.

**Explanation:** The **feedforward process** is the one-way flow of data from the input layer, through the hidden layers, and finally to the output layer to produce a prediction. Each layer's output is the next layer's input.

##### 3.2 Implement the Feedforward Process

**Objective:** Implement a complete, basic feedforward pass using Python.

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
input_layer_size = 3
hidden_layer_size = 4
output_layer_size = 2
input_features = np.array([[25, 60, 1013]])

# Initialize weights randomly for demonstration
weights_input_hidden = np.random.rand(input_layer_size, hidden_layer_size)
weights_hidden_output = np.random.rand(hidden_layer_size, output_layer_size)

# --- Execution ---
output = feedforward(input_features, weights_input_hidden, weights_hidden_output)

print("--- Feedforward Process ---")
print("Input Features:\n", input_features)
print("\nFinal Feedforward Output:\n", output)
```

---

### Part 3: Concluding Your Session

#### 1. Shut Down Jupyter

In the terminal where you ran `jupyter notebook`, press `Ctrl + C` and then confirm with `y` to shut down the server.

#### 2. Deactivate the Virtual Environment

Once you are done with your work, you can deactivate the environment by simply typing:
```bash
deactivate
```

Your terminal prompt will return to normal. To work on the project again, just `cd` into the directory and run `source venv/bin/activate`.

---

## Conclusion

**Recap:**

In this lab, you have:
*   Set up a professional Python project environment.
*   Visualized the basic three-layer architecture of a neural network.
*   Defined and represented layers, neurons, and activation functions using NumPy.
*   Implemented a simple feedforward process to understand how data flows through a network to generate a prediction.
