> [!NOTE]
> This document is a brief explanation of the math behind the solver. It is NOT meant to be a detailed proof. If you wish to read more on the math behind how the solver works, please refer to the resources listed in the [main markdown](https://github.com/Implycitt/LightsOut/blob/main/public/math.md).

## Setup
---

Since the Lights Out puzzle is a grid of buttons that can only be in an on or off state, we can represent the board as a matrix with 1's for an on state and 0's as an off state; meaning we are only dealing with two values: 0 and 1. Before even beginning to attempt to solve, we need to be able to define our operations that will work similar to a modulo.

A Galois Field allows us to redefine operations on elements contained in a finite set. In our case, the finite set will contain 1 and 0. The operations performed on elements in the set must satisfy the field axioms. The field is closed meaning that all operations performed on an element contained within the field must map to another element inside the set. In our case, with a Binary Galois Field, these are the results of the addition of all elements inside the set:

$$ 1 + 1 = 0 $$
$$ 0 + 1 = 1 $$
$$ 1 + 0 = 1 $$
$$ 0 + 0 = 0 $$

We can then define, $A$, an adjacency matrix, which is an $n^2 \times n^2$ matrix, for the board represented as a graph. If we take each light, or node and represent it as a vertex, we can draw an edge between that vertex and the other vertices that it will affect when pressed. For now we will look at a smaller $3\times3$ example, however the construction of the adjacency matrix can be generalized to any $n \in \mathbb{Z}$

```math
A = \begin{bmatrix}
        Z &I & 0\\
        I &Z & I\\
        0 &I & Z
        \end{bmatrix}

Z = \begin{bmatrix}
        1 &1 & 0\\
        1 &1 & 1\\
        0 &1 & 1 
        \end{bmatrix}
```

The matrix $Z$ is the adjacency matrix for the line graph of nodes in the same row. Since pressing the node also affects its own state, the diagonal of the adjacency matrix gets filled with ones. The matrix $I$ is simply the identity matrix and is necessary to be able to represent vertical connections. The $0$ represents an $n\times n$ matrix filled with zeros - the zero matrix; these nodes are unreachable from the node we are representing.

The best way of thinking of our matrix $A$ is that each column, $i$, is a flattened grid matrix where each 1 represents the nodes that will be affected when the $i$-th node is toggled which is also a 1.

## Config
---

Given a configuration, $\vec{b}$, we first have to check if that configuration is valid or has a solution. In order for our configuration vector to be valid, it has to belong to the orthogonal complements of the null space of matrix $A$; if it does not belong to this null space then we cannot get from the zero matrix to our configuration $\vec{b}$.

From there, we can set up our equation:

$$
    A\vec{x} = \vec{b}
$$

$A$ is our adjacency matrix that we constructed. $\vec{x}$ is a vector of length $n^2$ representing the strategy, or sequence, to get to the configuration vector $\vec{b}$ from the zero matrix.

Finally, to attain our solution, we take the inverse of our adjacency matrix and dot it with the configuration vector:

$$
    \vec{x} = A^{-1}\cdot\vec{b}
$$

There are also additional winning configurations which involve the addition of the null vectors from $A$. These can be computed and the vector with the smallest sum of components can be used as the most efficient solution to the given configuration.

