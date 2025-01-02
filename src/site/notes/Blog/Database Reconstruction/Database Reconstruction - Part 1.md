---
{"dg-publish":true,"permalink":"/blog/database-reconstruction/database-reconstruction-part-1/","created":"2024-12-29T08:42:00.368+01:00","updated":"2025-01-03T00:23:02.271+01:00"}
---

# Database Reconstruction - Part 1

One common challenge is convincing people that aggregate information can still qualify as personal data under the GDPR. By “aggregate information,” we refer to statistical summaries such as sums, medians, and means derived from a confidential dataset, or even the parameters of a trained machine learning model.

In this post, we start with the simplest case: demonstrating how sensitive personal data can be reconstructed from the results of a series of SUM queries performed on a confidential dataset. In a follow-up post, we’ll explore how an attacker can strategically minimize the number of queries needed to reconstruct every record in the dataset.

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

The task is to check if *any* $x_i$ can be unambiguously determined, or, in the worst case, whether the entire [system of equations](https://en.wikipedia.org/wiki/System_of_linear_equations) has a unique solution (i.e., all $x_i$ can be uniquely identified). If that is the case, the queries specified by matrix $\mathbf{A}$ cannot be answered without revealing individual private attribute values[^1].
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
where $\mathcal{U}(1,m)$ represents the uniform distribution over integers in $[1, m]$. To minimize $\mathbb{E}_{j\sim \mathcal{U}(1,m)}[ r_j(\mathbf{x})]$, we can use stochastic gradient descent: in each iteration $i$,  we randomly select a row/query $j$ and update the current estimate $\mathbf{x}^{(i)}$ as follows:
$$
x^{(i+1)} = x^{(i)} - \eta \nabla_\mathbf{x} r_j(\mathbf{x})
$$
where the gradient of $r_j(\mathbf{x})$ is given by:
$$
\nabla_\mathbf{x} r_j(\mathbf{x})=-2(b_j - A_j x^{(i)}) A_j^{\mathsf{T}} 
$$

If we set the step size  $\eta = \frac{1}{2||A_j||_2^2}$, this update becomes the randomized version of [Kaczmarz's method](https://en.wikipedia.org/wiki/Kaczmarz_method) :
$$
x^{(i+1)} = x^{(i)} + \frac{(b_j - A_j x^{(i)})A_j^{\mathsf{T}}}{||A_j||_2^2}
$$

Kaczmarz’s method has an intuitive geometric interpretation: starting from a random location $x^{(0)}$, a better solution $x^{(i+1)}$ is found by projecting $x^{(i)}$ onto the hyperplane defined by the $j$th equation, $\langle A_j,x \rangle = b_j$ [^2]. The steps of the process in 2D are shown in the figure below, where each equation is a line, and their single intersection point in the center is the unique solution to the whole system of equations. For consistent systems, $x^{(i)}$ gradually converges to this unique solution.

![Pasted image 20250102094456.png](/img/user/Blog/Database%20Reconstruction/Pasted%20image%2020250102094456.png)

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
This is close to the exact solution computed by cvxpy above!

# Beyond linear queries

SUM and AVG are linear queries can be efficiently audited using tools from linear algebra. However, auditing non-linear queries such as MEDIAN, MAX, and MIN is much more challenging. In fact, [checking disclosure](https://theory.stanford.edu/~nmishra/Papers/surveyQueryAuditingTechniquesDataPrivacy.pdf) for such queries may not even be feasible in polynomial time of the dataset size $n$. In these cases, we can resort to [heuristics such as SAT solvers](https://dl.acm.org/doi/10.1145/3287287).

---
[^1]: Seemingly, solving this linear system for any unknown, without associating it with a specific individual (e.g., Jeremy Doe), should not imply a privacy breach under GDPR. For example, according to GDPR, information is considered personal only if it can be linked to an identified or identifiable person. If the attacker only has access to public attributes (marked in blue), they likely cannot link the value of  $x_2$  to Jeremy Doe. However, the attacker may have additional background knowledge that could help identify Jeremy, even without access to the “Name” attribute. For example, they could know from another source (such as Facebook) that Jeremy visited a hospital and that his ZIP code starts with `438**` If this additional information is readily accessible, it could enable the attacker to single out Jeremy’s record. This post aims to demonstrate that query results themselves - such as the outcomes from Query 1, 2, and  3 - can be considered personal or sensitive data under GDPR, especially when combined with background knowledge that makes it possible to identify an individual.

[^2]: Let $x_p$ be the projection of $x^{(i)}$ onto hyperplane $A_j\mathbf{x} = b_j$ and  $x'$  be any reference point that lies on this hyperplane.  Since vector $A_j$ is orthogonal to this hyperplane, $x_p$ can be described as the result of shifting $x^{(i)}$ by the projection of vector  $(x^{(i)} - x')$ to vector $A_j$. The vector  $(x^{(i)} - x')$  represents the displacement from a reference point  $x'$ to the point $x^{(i)}$. The [projection](https://en.wikipedia.org/wiki/Vector_projection)of this displacement vector onto $A_j$ is given by
$$
\left\langle \frac{A_j}{||A_j||_2}, x^{(i)} - x' \right\rangle \frac{A_j^{\mathsf{T}}}{{||A_j||_2}}
$$and hence
$$
\begin{align}
x_p &= x^{(i)} - \left\langle \frac{A_j}{||A_j||_2}, x^{(i)} - x' \right\rangle \frac{A_j^{\mathsf{T}}}{{||A_j||_2}}\\
&= x^{(i)} -  \left\langle A_j, x^{(i)} - x' \right\rangle \frac{A_j^{\mathsf{T}}}{{||A_j||_2^2}} \\
&= x^{(i)} - \left(\langle A_j, x^{(i)}\rangle - \langle A_j, x'\rangle  \right) \frac{A_j^{\mathsf{T}}}{{||A_j||_2^2}}  \\
&=
x^{(i)} + \left(b_j - \langle A_j, x^{(i)}\rangle\right) \frac{A_j^{\mathsf{T}}}{{||A_j||_2^2}} 
\end{align}
$$ for any $x'$ such that $\langle A_j, x'\rangle = b$.

