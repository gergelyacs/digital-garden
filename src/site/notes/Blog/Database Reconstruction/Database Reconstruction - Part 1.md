---
{"dg-publish":true,"dg-path":"Database Reconstruction/Database Reconstruction - Part 1.md","permalink":"/database-reconstruction/database-reconstruction-part-1/","created":"2024-12-29T08:42:00.368+01:00","updated":"2025-01-04T11:17:43.121+01:00"}
---

One common challenge is convincing people that aggregate information can still qualify as personal data under the GDPR. By “aggregate information,” I refer to statistical summaries such as sums, medians, and means derived from a confidential dataset, or even the parameters of a trained machine learning model.

In this post, I start with the simplest case: demonstrating how sensitive personal data can be reconstructed from the results of a series of SUM queries performed on a confidential dataset. In a follow-up post, I’ll explore how an attacker can strategically minimize the number of queries needed to reconstruct every record in the dataset.

For the sake of illustration, consider the following hospital dataset.

| <span style="color:green;">**#**</span> | <span style="color:red;">**Name**</span>       | <span style="color:green;">**ZIP**</span> | <span style="color:green;">**Gender**</span>     | <span style="color:red;">**Blood sugar**</span> |
| --------------------------------------- | ---------------------------------------------- | ----------------------------------------- | ------------------------------------------------ | ----------------------------------------------- |
| <span style="color:green;">1</span>     | <span style="color:red;">John Smith</span>     | <span style="color:green;">32453</span>   | <span style="color:green;">Male</span>           | <span style="color:red;">4.3</span>             |
| <span style="color:green;">2</span>     | <span style="color:red;">Jeremy Doe</span>     | <span style="color:green;">43813</span>   | <span style="color:green;">Male</span>           | <span style="color:red;">5.2</span>             |
| <span style="color:green;">3</span>     | <span style="color:red;">Susan Atkinson</span> | <span style="color:green;">43765</span>   | <span style="color:green;">Female</span><br><br> | <span style="color:red;">6.1</span>             |
| <span style="color:green;">4</span>     | <span style="color:red;">Eve Brown</span>      | <span style="color:green;">32187</span>   | <span style="color:green;">Female</span>         | <span style="color:red;">3.2</span>             |
| <span style="color:green;">5</span>     | <span style="color:red;">Joseph Sky</span>     | <span style="color:green;">33745</span>   | <span style="color:green;">Male</span>           | <span style="color:red;">7.1</span>             |
| <span style="color:green;">6</span>     | <span style="color:red;">Alison Moon</span>    | <span style="color:green;">22983</span>   | <span style="color:green;">Female</span>         | <span style="color:red;">6.2</span>             |

Here, `ZIP` and `Gender` (in green) are considered public attributes, while `Name` and `Blood Sugar` (in red) are private. We show how statistical queries (such as sums or averages) over `Blood Sugar` can potentially expose an individual's blood sugar value - even when each query is computed across multiple records.

Following SQL syntax, a query consists of a `WHERE` clause, which filters records from the dataset based on public attributes, and an aggregate function that summarizes a private attribute over the selected records. For example, the query `SELECT SUM("Blood sugar") WHERE Gender = "Male"` uses the predicate `Gender = "Male"` and the aggregate function `SUM`, returning the result 16.6 (the sum of blood sugar values for males: $4.3 + 5.2 + 7.1$). 

It is not hard to see that answering the following queries reveal the blood sugar level of the second record (Jeremy Doe), even though each query covers at least two records.

1. Query 1: `SELECT SUM("Blood sugar") FROM Dataset`; Result: 32.1
2. Query 2: `SELECT SUM("Blood sugar") FROM Dataset WHERE Gender = "Female"`; Result: 15.5
3. Query 3: `SELECT SUM("Blood sugar") FROM Dataset WHERE ZIP > 32000 and ZIP < 35000 AND Gender = "Male"`; Result: 11.4

To see why, notice that Query 1 and 2 reveal the total blood sugar level of all males (subtract the result of Query 2 from that of Query 1), and Query 3 selects only Record 1 and 5. Therefore, taking the difference of the result of Query 3 and the total blood sugar of all males, we obtain
the exact blood sugar level of Record 2.

To make it more formal, let $x_i$ denote the unknown (private) Blood sugar value of record $i$. Each query along with its result can be represented by a linear equation as follows:

  1. Query 1: $x_1 + x_2 + x_3 + x_4 + x_5 + x_6 = 32.1$
  2. Query 2: $x_3 + x_4 + x_6 = 15.5$
  3. Query 3: $x_1 + x_5 = 11.4$
  
Or, with matrix notation: 
$$
\mathbf{A}\mathbf{x} = \mathbf{b}
$$
where 
$$
\mathbf{A} = \begin{bmatrix}
1 & 1 & 1 & 1 & 1 & 1 \\
0 & 0 & 1 & 1 & 0 & 1 \\
1 & 0 & 0 & 0 & 1 & 0
\end{bmatrix}
$$
$$
\mathbf{x} = (x_1, x_2, \ldots, x_6)^\mathsf{T}
$$
$$
\mathbf{b} = (32.1, 15.5, 11.4)^\mathsf{T}
$$

The task is to check if *any* $x_i$ can be unambiguously determined, or, in the worst case, whether the entire [system of equations](https://en.wikipedia.org/wiki/System_of_linear_equations) has a unique solution (i.e., all $x_i$ can be uniquely identified). If that is the case, the queries specified by matrix $\mathbf{A}$ cannot be answered without revealing individual private attribute values.
> [!FAQ]- Why?
> Seemingly, solving this linear system for any unknown, without associating it with a specific individual (e.g., Jeremy Doe), should not imply a privacy breach under GDPR. For example, according to GDPR, information is considered personal only if it can be linked to an identified or identifiable person. If the attacker only has access to public attributes (marked in blue), they likely cannot link the value of  $x_2$  to Jeremy Doe. However, the attacker may have additional background knowledge that could help identify Jeremy, even without access to the “Name” attribute. For example, they could know from another source (such as Facebook) that Jeremy visited a hospital and that his ZIP code starts with `438**` If this additional information is readily accessible, it could enable the attacker to single out Jeremy’s record. This post aims to demonstrate that query results themselves - such as the outcomes from Query 1, 2, and  3 - can be considered personal or sensitive data under GDPR, especially when combined with background knowledge that makes it possible to identify an individual.
## Which records are exposed?

Given a linear system of equations $\mathbf{A}\mathbf{x} = \mathbf{b}$, where $\mathbf{b} = \{b_1, \ldots, b_m\}$, $\mathbf{A} \in \mathbb{R}^{m,n}$, and row $i$ of $\mathbf{A}$ corresponds to query $q_i$. In general, an unknown $x_i$ can be unambiguously determined if the following conditions are met:

1. The system must be consistent, meaning that the vector $\mathbf{b}$ is a linear combination of the columns of $\mathbf{A}$. In other words, $\mathbf{b}$ must lie within the column space of $\mathbf{A}$. This is verified by checking if $\text{rank}(\mathbf{A}) = \text{rank}([\mathbf{A} \, | \, \mathbf{b}])$, where $[\mathbf{A} \, | \, \mathbf{b}]$ is the augmented matrix formed by appending $\mathbf{b}$ as a column to $\mathbf{A}$.

2. The $i$th column of $\mathbf{A}$ must be **linearly independent** of the remaining columns. To check this, remove the $i$th column from $\mathbf{A}$ and compute the rank of the resulting matrix. If the rank decreases, it confirms that the $i$th column is linearly independent.
  
In the above example, each query was answered faithfully, hence the linear system is consistent. To illustrate the second condition, let's reformulate $\mathbf{A}\mathbf{x} = \mathbf{b}$ as:
$$

x_1\cdot

\begin{bmatrix}

1 \\

0 \\

1

\end{bmatrix}

+

x_2\cdot

\begin{bmatrix}

1 \\

0 \\

0

\end{bmatrix}

+

x_3\cdot

\begin{bmatrix}

1 \\

1 \\

0

\end{bmatrix}

+

x_4\cdot

\begin{bmatrix}

1 \\

1 \\

0

\end{bmatrix}

+

x_5\cdot

\begin{bmatrix}

1 \\

0 \\

1

\end{bmatrix}

+

x_6\cdot

\begin{bmatrix}

1 \\

1 \\

0

\end{bmatrix}

=

\begin{bmatrix}

32.1 \\

15.5 \\

11.4

\end{bmatrix}

$$
This expression shows that vector $\mathbf{b}$ is a linear combination of the columns of the matrix $\mathbf{A}$, weighted by the unknowns $x_1, \ldots, x_6$. Here, the contribution of the second column $(1, 0, 0)^\mathsf{T}$ to the linear combination $\mathbf{b}$ is unique and cannot be replicated by any other column or combination of columns from the matrix $\mathbf{A}$. Indeed, adding this column to matrix
$$\begin{bmatrix}

1 & 1 & 1 & 1 & 1 \\

0 & 1 & 1 & 0 & 1 \\

1 & 0 & 0 & 1 & 0

\end{bmatrix}$$
increases its rank by one:

``` python
import numpy as np
from numpy.linalg import matrix_rank

# Define the matrix A and vector b
A = np.array([
[1, 1, 1, 1, 1, 1],
[0, 0, 1, 1, 0, 1],
[1, 0, 0, 0, 1, 0]
])

b = np.array([32.1, 15.5, 11.4])

for col in range(A.shape[1]):
	print (f"Unknown x_{col+1} can be uniquely determined:",
	matrix_rank(A) != matrix_rank(np.delete(A, col, axis=1)))
```

which produces:

```
Unknown x_1 can be uniquely determined: False
Unknown x_2 can be uniquely determined: True
Unknown x_3 can be uniquely determined: False
Unknown x_4 can be uniquely determined: False
Unknown x_5 can be uniquely determined: False
Unknown x_6 can be uniquely determined: False
```

However, this uniqueness does not apply to the other columns which appear multiple times in the matrix. After multiplying out these (linearly dependent) columns we have:
$$

(x_1 + x_5)\cdot

\begin{bmatrix}

1 \\

0 \\

1

\end{bmatrix}

+

x_2\cdot

\begin{bmatrix}

1 \\

0 \\

0

\end{bmatrix}

+

(x_3 + x_4 + x_6)\cdot

\begin{bmatrix}

1 \\

1 \\

0

\end{bmatrix}

=

\begin{bmatrix}

32.1 \\

15.5 \\

11.4

\end{bmatrix}

$$
Since all the three column vectors are linearly indepedent in this reformulated version, we can uniquely determine their weights: $x_2$, the sum of $x_1$ and $x_5$, and the sum of $x_3$, $x_4$, and $x_6$. However, we cannot tell how these sums are shared among their respective variables. This implies that the original linear system as a whole still has an infinite number of solutions. 

In general, if $\text{rank}([\mathbf{A}|\mathbf{b}]) = \text{rank}(\mathbf{A}) < n$, as in the example above, the system is underdetermined because there are fewer linearly independent queries (rows) than unknowns. As a result, the solution space typically has infinitely many solutions. Imposing constraints, such as requiring $\mathbf{x}$ to be an integer or enforcing sparsity (at most $\text{rank}(\mathbf{A})$ non-zero entries in $\mathbf{x}$) can reduce the solution space to a finite set.

If $\text{rank}([\mathbf{A}|\mathbf{b}]) = \text{rank}(\mathbf{A}) = n$, meaning $\mathbf{A}$ is square and full rank, the system has a unique solution given by $\mathbf{x} = \mathbf{A}^{-1}\mathbf{b}$. In that case, all columns are linearly independent and therefore *every* unknown $x_i$ can be unambiguously determined. It may happen that we have more rows than columns because some rows/queries are linearly dependent, but the number of linearly independent queries is at most $n$.
## Solving the equations

There are a couple of techniques to solve (or approximate) a linear system; any matrix decomposition technique (e.g., QR or LU decomposition) can be applied on the augmented matrix $\mathbf{A}^*$. Here, we rather use a more versatile optimization technique which provides an approximation of $\mathbf{x}$ irrespective of the rank of $[\mathbf{A}|\mathbf{b}]$:

$$
\mathbf{x}' = \arg\min_{\mathbf{x} \in \mathbb{R}^n}  ||\mathbf{A}\mathbf{x} - \mathbf{b}||_2
$$

where $||\cdot ||_2$ denotes the Euclidean distance ($L_2$-norm).

The solution of this optimization problem has a closed form $\mathbf{x}' = \mathbf{\hat{A}} \mathbf{b}$, where $\mathbf{\hat{A}}$ is the pseudo-inverse (or [Moore-Penrose inverse](https://en.wikipedia.org/wiki/Moore–Penrose_inverse)) of $\mathbf{A}$. If $\mathbf{A}$ is invertible, then $\mathbf{A}^{-1} = \mathbf{\hat{A}}$. The reconstruction error $||\mathbf{x}' - \mathbf{x}||_2$ depends on $\text{rank}(\mathbf{A})$ (the number of linearly independent  rows/columns). Exact reconstruction $(\mathbf{x}' = \mathbf{x})$ is only guaranteed if $\mathbf{A}$ has full rank and $\text{rank}(\mathbf{A}) = \text{rank}([\mathbf{A}|\mathbf{b}])$, as discussed earlier. This optimization technique is widely known as [Ordinary Least Squares (OLS)](https://en.wikipedia.org/wiki/Ordinary_least_squares#:~:text=In%20statistics%2C%20ordinary%20least%20squares,of%20the%20squares%20of%20the) or linear regression in machine learning terminology. OLS and pseudo-inverse computations are supported by many linear algebra libraries and machine learning frameworks, including [scikit-learn](hhttps://scikit-learn.org/1.5/modules/generated/sklearn.linear_model.LinearRegression.html), [torch](https://pytorch.org/docs/stable/generated/torch.linalg.lstsq.html), [numpy](https://numpy.org/doc/2.1/reference/generated/numpy.linalg.lstsq.html).

For example, in numpy:

``` python
# Solve the system using NumPy's least-squares method (since A is not square)
x_prime = np.linalg.lstsq(A, b, rcond=None)[0]

print ("Solution:", x_prime)
```

and the output:

``` python
Solution: [5.7 5.2 5.16666667 5.16666667 5.7 5.16666667]
```

where Jeremy Doe's blood sugar level ($x_2$) is accurately computed. Notice that the sums $x_1 + x_5 = 11.4$ and $x_3+x_4+x_6 = 15.5$ are correct. OLS finds the solution with the smallest $L_2$-norm, which occurs when the sums are evenly distributed among their respective variables.

### Incorporating background knowledge as constraints

The above solution can be made more accurate if we incorporate prior knowledge about the unknowns as additional constraints into the optimization problem. As long as these constraints are convex, efficient convex solvers can be used to obtain a better approximation of $\mathbf{x}$. For example, knowing that John Smith’s blood sugar level is less than 5 could improve the accuracy of the approximation for Joseph Sky’s blood sugar level:
``` python
import cvxpy as cvx

x = cvx.Variable(A.shape[1])
objective = cvx.Minimize(cvx.sum_squares(A @ x - b))
# First record is smaller than 5
constraints = [x[0] <= 5]
  
prob = cvx.Problem(objective, constraints)
prob.solve()

print ("Solution:", x.value)
```
and the output is:
``` python
Solution: [1.15961346e-04 5.20000027e+00 5.16666694e+00 5.16666694e+00 1.13998846e+01 5.16666694e+00]
```

This is quite dummy as no one has a blood sugar level of less than 0.01. Let's add another constraint that all values should be larger than 3:

``` python
objective = cvx.Minimize(cvx.sum_squares(A @ x - b))
# Constraint 1: First record is less than 5
# Constraint 2: All records are larger than 3
constraints = [x[0] <= 5, x >= 3]
  
prob = cvx.Problem(objective, constraints)
prob.solve()

print ("Solution:", x.value)
```
The output is:
``` python
Solution: [4.06728662 5.20000018 5.1666669 5.1666669 7.3327138 5.1666669 ]
```
This looks much more realistic! This approximation is closer to the true solution $\mathbf{x}$ and suggests that Joseph Sky may have hyperglycemia.

The private attribute $\mathbf{x}$ may sometimes be binary in practice, such as indicating whether a patient has AIDS or not. Such an *integer constraint* turns the problem into an instance of Integer Linear Programming (ILP) which is NP-complete in general. While [cvxpy](https://www.cvxpy.org/tutorial/solvers/index.html) supports mixed integer linear programming (MILP) solvers, a more practical  and efficient approach is to round each element of $\mathbf{x}'$ (the OLS solution) to the nearest integer, obtaining a final approximation of the original vector $\mathbf{x}$. 

### Handling Many Queries and Records

Although convex optimization, including OLS, have polynomial time complexity in the record number $n$ and query number $m$, they can become quite slow and memory-intensive for large datasets with many queries as they operate on the entire matrix $\mathbf{A}$. Iterative methods, on the other hand, generate a sequence of approximate solutions that converge progressively closer to the exact solution with each iteration. These methods are typically less computationally demanding, though their convergence rate depends on the properties of matrix $\mathbf{A}$. Common iterative methods include the [Jacobi method](https://en.wikipedia.org/wiki/Jacobi_method), the [Conjugate Gradient method](https://en.wikipedia.org/wiki/Conjugate_gradient_method), or the [L-BFGS](https://en.wikipedia.org/wiki/Limited-memory_BFGS). However, these algorithms may still be inefficient when dealing with a large number of equations and not only unknowns.

To address this, we present a solution based on [stochastic gradient descent (SGD)](https://en.wikipedia.org/wiki/Gradient_descent), which uses only one row of the matrix $\mathbf{A}$ per iteration. This approach is particularly appealing in online scenarios where queries must be audited in real-time, or where storing the entire query set is impractical due to legal constraints or the sheer number of queries.

Our objective function is $R(\mathbf{x}) = ||\mathbf{b}-\mathbf{A}\mathbf{x}||_2^2$, where the $j$th equation (query) is defined as  $r_i(\mathbf{x}) = ||b_j - A_j \mathbf{x} ||_2^2$, with $A_j$ being the $j$th row of matrix. Since $R(\mathbf{x}) = \sum_{j=1}^m r_j(\mathbf{x})$, we can write
$$
\begin{align}
\arg\min_{\mathbf{x}} R(\mathbf{x}) &= \arg\min_{\mathbf{x}} \sum_{j=1}^m(1/m) \cdot r_j(\mathbf{x})\\
&= \arg\min_{\mathbf{x}} \mathbb{E}_{j\sim \mathcal{U}(1,m)}[ r_j(\mathbf{x})]
\end{align}
$$
where $\mathcal{U}(1,m)$ represents the uniform distribution over integers in $[1, m]$. To minimize $\mathbb{E}_{j\sim \mathcal{U}(1,m)}[ r_j(\mathbf{x})]$, we can use gradient descent with the following update rule:
$$
\begin{align}
x^{(i+1)} &= x^{(i)} - \eta \cdot \nabla_\mathbf{x} \mathbb{E}_{j\sim \mathcal{U}(1,m)}[ r_j(\mathbf{x})]\\
&= x^{(i)} - \eta \cdot \mathbb{E}_{j\sim \mathcal{U}(1,m)}[ \nabla_\mathbf{x} r_j(\mathbf{x})]
\end{align}
$$
where $\eta$ is the learning rate. This approach converges to the global minimum of $R(\mathbf{x})$, as $r_j(\mathbf{x})$ is convex. However, this approach still requires iterating over all queries in each iteration. To overcome this, we apply *stochastic gradient descent* to approximate the expected value through sampling. In each iteration $i$, a query (or equation) $j$ is randomly selected, and the gradient's expected value over all queries is estimated using the gradient of this single query. This is gradient is given by:
$$
\nabla_\mathbf{x} r_j(\mathbf{x})=-2(b_j - A_j x^{(i)}) A_j^{\mathsf{T}} 
$$
If we set the step size  $\eta = \frac{1}{2||A_j||_2^2}$, this iterative approach becomes the randomized version of [Kaczmarz's method](https://en.wikipedia.org/wiki/Kaczmarz_method) :
$$
x^{(i+1)} = x^{(i)} + \frac{(b_j - A_j x^{(i)})A_j^{\mathsf{T}}}{||A_j||_2^2}
$$

Kaczmarz’s method has an intuitive geometric interpretation: starting from a random location $x^{(0)}$, a better solution $x^{(i+1)}$ is found by projecting $x^{(i)}$ onto the hyperplane defined by the $j$th equation, $\langle A_j,x \rangle = b_j$. 

> [!FAQ]- Why?
> Let $x_p$ be the projection of $x^{(i)}$ onto hyperplane $A_j\mathbf{x} = b_j$ and  $x'$  be any reference point that lies on this hyperplane.  Since vector $A_j$ is orthogonal to this hyperplane, $x_p$ can be described as the result of shifting $x^{(i)}$ by the projection of vector  $(x^{(i)} - x')$ to vector $A_j$. The vector  $(x^{(i)} - x')$  represents the displacement from a reference point  $x'$ to the point $x^{(i)}$. 
> > [!FAQ]- How does it look like in two dimenions?
> > <style> .container {font-family: sans-serif; text-align: center;} .button-wrapper button {z-index: 1;height: 40px; width: 100px; margin: 10px;padding: 5px;} .excalidraw .App-menu_top .buttonList { display: flex;} .excalidraw-wrapper { height: 800px; margin: 50px; position: relative;} :root[dir="ltr"] .excalidraw .layer-ui__wrapper .zen-mode-transition.App-menu_bottom--transition-left {transform: none;} </style><script src="https://cdn.jsdelivr.net/npm/react@17/umd/react.production.min.js"></script><script src="https://cdn.jsdelivr.net/npm/react-dom@17/umd/react-dom.production.min.js"></script><script type="text/javascript" src="https://cdn.jsdelivr.net/npm/@excalidraw/excalidraw@0/dist/excalidraw.production.min.js"></script><div id="Projectionexcalidraw.md1"></div><script>(function(){const InitialData={"type":"excalidraw","version":2,"source":"https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/2.7.4","elements":[{"id":"yDc9n6BIpzqE_nHdt59l_","type":"rectangle","x":-241.41141618363827,"y":44.578335981234204,"width":1239.4953544335258,"height":1131.4544999884572,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#ffec99","fillStyle":"hachure","strokeWidth":0.5,"strokeStyle":"solid","roughness":2,"opacity":90,"groupIds":[],"frameId":null,"index":"a0","roundness":null,"seed":1708700751,"version":323,"versionNonce":1370014753,"isDeleted":false,"boundElements":[],"updated":1735980911971,"link":null,"locked":false},{"id":"V7X7XdRtdS1OZMbE9s4r4","type":"line","x":361.6717032895871,"y":178.8719201496103,"width":364.46946376351525,"height":913.5809986760495,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a1","roundness":{"type":2},"seed":1486604962,"version":411,"versionNonce":1458510511,"isDeleted":false,"boundElements":[],"updated":1735980507952,"link":null,"locked":false,"points":[[0,0],[-364.46946376351525,913.5809986760495]],"lastCommittedPoint":null,"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null},{"id":"JAdc_aG4LekQXekQtyD1z","type":"ellipse","x":275.97877578966813,"y":315.80560483647844,"width":35.93145252618887,"height":42.46444389458684,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#1e1e1e","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a2","roundness":{"type":2},"seed":1982851390,"version":423,"versionNonce":656960449,"isDeleted":false,"boundElements":[{"id":"MVRFi08fnvhG7wDQxibUV","type":"arrow"},{"id":"PJ3jwZszquRRNk-qFTI9b","type":"arrow"}],"updated":1735980507952,"link":null,"locked":false},{"id":"2Y5ELn9B4FuxEhogea4an","type":"ellipse","x":428.1226485798486,"y":783.1440759163246,"width":35.93145252618887,"height":42.46444389458684,"angle":0,"strokeColor":"#e8590c","backgroundColor":"#e8590c","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a3","roundness":{"type":2},"seed":1906737342,"version":571,"versionNonce":565069071,"isDeleted":false,"boundElements":[{"id":"MVRFi08fnvhG7wDQxibUV","type":"arrow"},{"id":"mTQu355I8xTi5F3eUoLad","type":"arrow"}],"updated":1735980507952,"link":null,"locked":false},{"id":"jRcNvJCViNnQGOVnsCZmQ","type":"ellipse","x":128.7071678719663,"y":696.089046688833,"width":35.93145252618887,"height":42.46444389458684,"angle":0,"strokeColor":"#e8590c","backgroundColor":"#e8590c","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a3G","roundness":{"type":2},"seed":1553992830,"version":638,"versionNonce":71824161,"isDeleted":false,"boundElements":[{"id":"MVRFi08fnvhG7wDQxibUV","type":"arrow"},{"id":"mTQu355I8xTi5F3eUoLad","type":"arrow"}],"updated":1735980507952,"link":null,"locked":false},{"id":"MVRFi08fnvhG7wDQxibUV","type":"arrow","x":305.90176141548227,"y":363.1077274156024,"width":132.89062470781107,"height":417.9624486777933,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#1e1e1e","fillStyle":"solid","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a5","roundness":{"type":2},"seed":1980125538,"version":1093,"versionNonce":1191675919,"isDeleted":false,"boundElements":[],"updated":1735980508185,"link":null,"locked":false,"points":[[0,0],[132.89062470781107,417.9624486777933]],"lastCommittedPoint":null,"startBinding":{"elementId":"JAdc_aG4LekQXekQtyD1z","focus":-0.19113860434705363,"gap":8.105260065310397,"fixedPoint":null},"endBinding":{"elementId":"2Y5ELn9B4FuxEhogea4an","focus":0.005948400577534531,"gap":3.542825877264722,"fixedPoint":null},"startArrowhead":null,"endArrowhead":"triangle","elbowed":false},{"id":"mTQu355I8xTi5F3eUoLad","type":"arrow","x":427.12290779781296,"y":804.2600609172534,"width":257.95687978759196,"height":75.35364128455956,"angle":0,"strokeColor":"#e8590c","backgroundColor":"#1e1e1e","fillStyle":"solid","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":50,"groupIds":[],"frameId":null,"index":"a7","roundness":{"type":2},"seed":149303650,"version":1049,"versionNonce":2138304463,"isDeleted":false,"boundElements":[],"updated":1735980508185,"link":null,"locked":false,"points":[[0,0],[-257.95687978759196,-75.35364128455956]],"lastCommittedPoint":null,"startBinding":{"elementId":"2Y5ELn9B4FuxEhogea4an","focus":-0.24799279284168158,"gap":1,"fixedPoint":null},"endBinding":{"elementId":"jRcNvJCViNnQGOVnsCZmQ","focus":0.2292749226734886,"gap":6.745527606016125,"fixedPoint":null},"startArrowhead":null,"endArrowhead":"triangle","elbowed":false},{"id":"igzT0ryvd6CmZ6iWV5Kby","type":"line","x":310.8730460721682,"y":339.1061694229296,"width":308.6097332539094,"height":114.90011369900837,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#1e1e1e","fillStyle":"solid","strokeWidth":1,"strokeStyle":"dashed","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a8","roundness":{"type":2},"seed":1439145186,"version":409,"versionNonce":1446394799,"isDeleted":false,"boundElements":[],"updated":1735980507952,"link":null,"locked":false,"points":[[0,0],[308.6097332539094,114.90011369900837]],"lastCommittedPoint":null,"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null},{"id":"-sMwhBXSZRoIw70PbQXBg","type":"line","x":457.97132364421964,"y":779.681089189847,"width":132.01811732391224,"height":363.57370193038133,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"#1e1e1e","fillStyle":"solid","strokeWidth":1,"strokeStyle":"dashed","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"a9","roundness":{"type":2},"seed":356322366,"version":492,"versionNonce":2082462401,"isDeleted":false,"boundElements":[],"updated":1735980507952,"link":null,"locked":false,"points":[[0,0],[132.01811732391224,-363.57370193038133]],"lastCommittedPoint":null,"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null},{"id":"CyQKvnJs","type":"image","x":469.36814128132585,"y":813.575494369808,"width":89.47818537006093,"height":59.65212358004062,"angle":0,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":74034,"version":520,"versionNonce":320355233,"updated":1735980608733,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"c8f7dcca926c182b82e86ce92a8a4293c4b9d01e","scale":[1,1],"index":"aA","frameId":null,"status":"pending","crop":null},{"id":"GuOcUup6TtEt6oyNSoIyr","type":"image","x":-24.273402474120317,"y":685.2960843661969,"width":132.64947066852764,"height":55.85240870253796,"angle":0,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":1188642049,"version":528,"versionNonce":1053882415,"updated":1735980604366,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"4d1d5f7e5f07258cf75ffc1a89cebcfe697b47db","scale":[1,1],"index":"aB","frameId":null,"status":"pending","crop":null},{"id":"PyKnG0liXAyJmGEOXYey_","type":"image","x":-162.50262345065553,"y":1111.2981505645928,"width":259.2710908867745,"height":54.41492030956996,"angle":0,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":332362465,"version":589,"versionNonce":62688769,"updated":1735980600925,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"517bf499393e06d4b005584757ea94f2f96d5702","scale":[1,1],"index":"aBV","frameId":null,"status":"pending","crop":null},{"id":"Yp7jMsKq9iRBvsIKSXEP9","type":"image","x":179.73153222214347,"y":281.96340971723885,"width":81.25721778357745,"height":50.302087199357466,"angle":0,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":444097327,"version":607,"versionNonce":1107034063,"updated":1735980613565,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"1c34a110bfa091e9906b2570f42933f3d0da0499","scale":[1,1],"index":"aC","frameId":null,"status":"pending","crop":null},{"id":"CBSHHTdi3YXCgMFPBsXWw","type":"image","x":502.61471889624175,"y":81.94153055082327,"width":444.87288328265487,"height":182.51195211596098,"angle":6.259163604864684,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":1021631553,"version":1343,"versionNonce":1634644417,"updated":1735980629764,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"d59c3f46a403e09aac505562ad2897f9253d32f9","scale":[1,1],"index":"aD","frameId":null,"status":"pending","crop":null},{"id":"a3qh4xf_5WONVs0HOaMWc","type":"line","x":352.0212809012071,"y":251.26565258354532,"width":258.1831627141313,"height":122.00390042871932,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"aI","roundness":{"type":2},"seed":758793505,"version":844,"versionNonce":1265198689,"isDeleted":false,"boundElements":[],"updated":1735980507952,"link":null,"locked":false,"points":[[0,0],[37.19785241793787,-22.470376514269333],[121.46509033468266,1.277179482160957],[152.57638428294723,-26.767553313623385],[152.27039336534617,18.45258272663799],[255.07668970283362,55.903210250420216],[258.1831627141313,95.23634711509594]],"lastCommittedPoint":null,"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null},{"id":"Shj6GTUefu0xIXjUHSIUM","type":"image","x":332.6353638239451,"y":391.34291199340095,"width":79.88104873657984,"height":31.064852286447714,"angle":6.1154863955710885,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":1302426881,"version":560,"versionNonce":118451247,"updated":1735980507952,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"901929de36ee4b348950f06127e60e71d9a0ca67","scale":[1,1],"index":"aJ","frameId":null,"status":"pending","crop":null},{"id":"GEOl8qYX7iJWkDCENvu_Q","type":"line","x":355.5800883124368,"y":482.1557558429558,"width":85.84375634003722,"height":80.02327692914746,"angle":0,"strokeColor":"#1e1e1e","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"aL","roundness":{"type":2},"seed":256358241,"version":314,"versionNonce":1158672961,"isDeleted":false,"boundElements":[],"updated":1735980507952,"link":null,"locked":false,"points":[[0,0],[74.02984613004837,-12.711927033383272],[85.84375634003722,-80.02327692914746]],"lastCommittedPoint":null,"startBinding":null,"endBinding":null,"startArrowhead":null,"endArrowhead":null},{"id":"PJ3jwZszquRRNk-qFTI9b","type":"arrow","x":314.10345505820214,"y":344.83838344207317,"width":178.9330750218078,"height":64.099067400124,"angle":0,"strokeColor":"#2f9e44","backgroundColor":"transparent","fillStyle":"solid","strokeWidth":2,"strokeStyle":"solid","roughness":1,"opacity":100,"groupIds":[],"frameId":null,"index":"aM","roundness":{"type":2},"seed":1167535727,"version":584,"versionNonce":806989359,"isDeleted":false,"boundElements":[],"updated":1735980508185,"link":null,"locked":false,"points":[[0,0],[178.9330750218078,64.099067400124]],"lastCommittedPoint":null,"startBinding":{"elementId":"JAdc_aG4LekQXekQtyD1z","focus":0.026098628425716898,"gap":3.2970350625324265,"fixedPoint":null},"endBinding":null,"startArrowhead":null,"endArrowhead":"arrow","elbowed":false},{"id":"Or2-KYnM8fI5ktG5sMPgg","type":"image","x":369.441370942375,"y":265.4343474286712,"width":59.98212041959367,"height":103.30254072263354,"angle":0,"strokeColor":"#000000","backgroundColor":"transparent","fillStyle":"hachure","strokeWidth":1,"strokeStyle":"solid","roughness":1,"opacity":100,"roundness":null,"seed":1615643247,"version":604,"versionNonce":1529938639,"updated":1735980618666,"isDeleted":false,"groupIds":[],"boundElements":[],"link":null,"locked":false,"fileId":"0afd6ea753fea2b01fccef31ad72e94e7ccf0253","scale":[1,1],"index":"aO","frameId":null,"status":"pending","crop":null}],"appState":{"theme":"light","viewBackgroundColor":"#ffffff","currentItemStrokeColor":"#1e1e1e","currentItemBackgroundColor":"#ffec99","currentItemFillStyle":"hachure","currentItemStrokeWidth":0.5,"currentItemStrokeStyle":"solid","currentItemRoughness":2,"currentItemOpacity":90,"currentItemFontFamily":5,"currentItemFontSize":20,"currentItemTextAlign":"left","currentItemStartArrowhead":null,"currentItemEndArrowhead":"arrow","currentItemArrowType":"round","scrollX":761.5467100646165,"scrollY":-202.97129148571696,"zoom":{"value":0.726829},"currentItemRoundness":"sharp","gridSize":20,"gridStep":5,"gridModeEnabled":true,"gridColor":{"Bold":"rgba(217, 217, 217, 0.5)","Regular":"rgba(230, 230, 230, 0.5)"},"currentStrokeOptions":null,"frameRendering":{"enabled":true,"clip":true,"name":true,"outline":true},"objectsSnapModeEnabled":false,"activeTool":{"type":"selection","customType":null,"locked":false,"lastActiveTool":null}},"files":{}};InitialData.scrollToContent=true;App=()=>{const e=React.useRef(null),t=React.useRef(null),[n,i]=React.useState({width:void 0,height:void 0});return React.useEffect(()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height});const e=()=>{i({width:t.current.getBoundingClientRect().width,height:t.current.getBoundingClientRect().height})};return window.addEventListener("resize",e),()=>window.removeEventListener("resize",e)},[t]),React.createElement(React.Fragment,null,React.createElement("div",{className:"excalidraw-wrapper",ref:t},React.createElement(ExcalidrawLib.Excalidraw,{ref:e,width:n.width,height:n.height,initialData:InitialData,viewModeEnabled:!0,zenModeEnabled:!0,gridModeEnabled:!1})))},excalidrawWrapper=document.getElementById("Projectionexcalidraw.md1");ReactDOM.render(React.createElement(App),excalidrawWrapper);})();</script>
> 
> The [projection](https://en.wikipedia.org/wiki/Vector_projection)of this displacement vector onto $A_j$ is given by
>  $$
>  \left\langle \frac{A_j}{||A_j||_2}, x^{(i)} - x' \right\rangle 
>  \frac{A_j^{\mathsf{T}}}{||A_j||_2}
> $$
>  and hence
>  $$
>  \begin{align}
>  x_p &= x^{(i)} - \left\langle \frac{A_j}{||A_j||_2}, x^{(i)} - x' \right\rangle \frac{A_j^{\mathsf{T}}}{||A_j||_2} \\
>  &= x^{(i)} -  \left\langle A_j, x^{(i)} - x' \right\rangle \frac{A_j^{\mathsf{T}}}{||A_j||_2^2} \\
>  &= x^{(i)} - \left(\langle A_j, x^{(i)}\rangle - \langle A_j, x'\rangle  \right) \frac{A_j^{\mathsf{T}}}{||A_j||_2^2}  \\
>  &=
>  x^{(i)} + \left(b_j - \langle A_j, x^{(i)}\rangle\right) \frac{A_j^{\mathsf{T}}}{||A_j||_2^2} 
>  \end{align}$$
>  for any $x'$ such that $\langle A_j, x' \rangle = b$.

The steps of the process in 2D are shown in the figure below, where each equation is a line, and their single intersection point in the center is the unique solution to the whole system of equations. For consistent systems, $x^{(i)}$ gradually converges to this unique solution.

![Kaczmarz.png](/img/user/Blog/Database%20Reconstruction/Kaczmarz.png)

This convergence can be accelerated by selecting row $j$ with probability proportional to its squared norm $||A_j||_2^2$. There are other [variants](https://link.springer.com/article/10.1007/s11075-024-01945-2) of Kaczmarz's method that have been adapted even for inconsistent systems.

Adding constraints is also possible. For example, box constraints  $\mathbf{\ell} \leq \mathbf{x} \leq \mathbf{u}$ can be incorporated by introducing a change of variables as follows:
$$
\arg\min_{\mathbf{x}: \mathbf{\ell} \leq \mathbf{x} \leq \mathbf{u}} R(\mathbf{x}) = \arg\min_{\mathbf{z}} R\left(\mathbf{\ell} + (\mathbf{u}- \mathbf{\ell}) \odot \text{sigmoid}(\mathbf{z})\right)
$$
where $\odot$ denotes the element-wise product between vectors, and $\text{sigmoid}(z) = 1/(1+\exp(-z))$ is applied element-wise to map each entry of $\mathbf{z}$ into the range $(0,1)$. This transformation ensures that the objective function remains differentiable, allowing gradient descent to be applied as in the unconstrained case.  Other techniques, like [barrier functions](https://en.wikipedia.org/wiki/Barrier_function), can also handle constraints.

Continuing our hospital example, suppose the blood sugar levels have lower bounds $\mathbf{\ell} = (3, 3, 3, 3, 3, 3)$  and upper bounds $\mathbf{u} = (5, 10, 10, 10, 10, 10)$ for our six records. Using [PyTorch's automatic differentation framework](https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html), we can efficiently compute gradients and perform optimization under the given box constraints:

``` python
import torch

A_t = torch.tensor(A).float()
b_t = torch.tensor(b)
# compute gradient wrt x_t
x_t = torch.zeros(A.shape[1], requires_grad=True)

# Box constraints (l: min blood sugar, u: max blood sugar)
l = torch.tensor([3, 3, 3, 3, 3, 3])
u = torch.tensor([5, 10, 10, 10, 10, 10])
z = lambda x_t: l + (u - l) * torch.sigmoid(x_t)

learning_rate = 0.01

# Do 1000 SGD iterations
for i in range(1000):
	# select one row from A
	row = np.random.randint(0, A_t.shape[0])
	
	# objective function
	residual = torch.sum((A_t[row] @ z(x_t) - b[row])**2)
	# compute gradient
	residual.backward()
	
	with torch.no_grad():
		# gradient descent
		x_t -= learning_rate * x_t.grad
		x_t.grad.zero_()

print ("Solution:", z(x_t).detach().numpy())
```
that gives:
``` python
Solution: [4.0698786 5.2092214 5.165333 5.165333 7.3272643 5.165333 ]
```
This is close to the exact solution computed by cvxpy [[Blog/Database Reconstruction/Database Reconstruction - Part 1#Incorporating background knowledge as constraints\|above]]!

# Beyond linear queries

SUM and AVG are linear queries can be efficiently audited using tools from linear algebra. However, auditing non-linear queries such as MEDIAN, MAX, and MIN is much more challenging. In fact, [checking disclosure](https://theory.stanford.edu/~nmishra/Papers/surveyQueryAuditingTechniquesDataPrivacy.pdf) for such queries may not even be feasible in polynomial time of the dataset size $n$. In these cases, we can resort to [heuristics such as SAT solvers](https://dl.acm.org/doi/10.1145/3287287).





