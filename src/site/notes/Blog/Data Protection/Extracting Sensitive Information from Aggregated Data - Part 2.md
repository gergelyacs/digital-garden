---
{"dg-publish":true,"dg-path":"Data Protection/Extracting Sensitive Information from Aggregated Data - Part 2.md","permalink":"/data-protection/extracting-sensitive-information-from-aggregated-data-part-2/","created":"2025-01-11T08:34:35.603+01:00","updated":"2026-09-26T12:17:43.661+02:00"}
---

Consider a hospital dataset with 100,000 patients, where only one patient has a rare genetic disease. To protect privacy, the hospital enforces _50,000-anonymity_: it only answers aggregate queries that cover at least half of the records. Can the record of this patient still be isolated? And if so, how many queries does it take? In this post, I show that, perhaps surprisingly, fewer than 20 queries, each covering about 50,000 patients, are enough to find the single patient with the rare disease.

The same question arises whenever the contributions of multiple participants are aggregated over several rounds. For example, aggregating the decisions of multiple agents or experts can mitigate hallucinations and correct erroneous (or even malicious) individual decisions, while aggregating model updates in collaborative (federated) learning avoids sharing sensitive training data directly. To protect the participants, cryptographic primitives such as _secure aggregation_ ensure that each round reveals only the aggregate over at least $k$ participants (typically a random subset of them), but never the individual contributions. Is this protection sufficient? And if not, how many rounds (or queries) are enough to reveal the private contributions of all participants?

# The Problem

We assume that all queries are _counting queries_ represented by a binary query matrix $A$, where $A_{i,j} = 1$ if query $i$ covers record $j$, and $A_{i,j} = 0$ otherwise. For example, the SQL query
```
SELECT COUNT(*) FROM Patients WHERE AGE = 32 and SEX = 'F'
```
counts the number of HIV+ patients who are both 32 years old and female. In terms of the matrix $A$, such a query corresponds to a row selecting exactly those records that satisfy the predicate after the WHERE clause.

Suppose we have $n$ patients and $m$ counting queries. Let $x_j$ be a Boolean value indicating whether patient $j$ is HIV-positive. Then, for query $i$, the result is
$$
\langle A_{i,:}, x \rangle \;=\; \sum_{j} A_{i,j}\, x_j \;=\; b_i ,
$$
where $b_i$ is the number of HIV-positive patients covered by query $i$.

We consider an adversary who (1) aims to design the query structure $A$ (i.e., which patients are covered by which queries), and (2) knows the query results $b$. The adversary’s goal is to reconstruct the unknown vector $x$. Importantly, the database manager only answers a query $A_i$ if it covers at least $k$ patients, ensuring a minimal privacy guarantee with $k$-anonymity.

The key questions addressed in this post:
1. **How an adversary should choose the queries, and hence design $A$, so that $x$ can be reconstructed from $b$?**
2. **How many queries are required for such an attack?**

The fewer queries needed, the cheaper the attack becomes, and the harder it is to detect.

# The Theory

In the [[Blog/Data Protection/Extracting Sensitive Information from Aggregated Data  - Part 1\|previous post]], I showed how individual records can be reconstructed from the results of aggregated SUM queries by solving a system of equations of the form $A \cdot x = b$, where $A$ is the query matrix, $x$ is the vector of unknowns, and $b$ contains the query results. In that setting, any unknown $x_i$ can be recovered if its corresponding column $A_{:,i}$ is linearly independent of all other columns of $A$. 
  
Let us look for a matrix $A$ whose columns are linearly independent, assuming for simplicity that $m = n$. Rather than verifying linear independence directly, it is often easier to check **orthogonality**, which is a stronger condition and automatically implies linear independence.

 >[!FAQ]- Why?
 > Vectors $A_1, A_2, \dots, A_k$ are **linearly independent** if no nontrivial linear combination of them equals zero, i.e. $c_1 A_1 + c_2 A_2 + \dots + c_k A_k = 0 \implies c_1 = c_2 = \dots = c_k = 0$. In other words, they span a vector space with dimension $k$ and the $A_1, \ldots A_k$ are not "linearly redundant".  $A_i, A_j$ are **orthogonal** if their inner product is zero:  $\langle A_i, A_j \rangle = 0$ for $i \neq j$. Orthogonality is the same as saying that the vectors are perpendicular, or equivalently, the cosine of their angle (aka cosine similarity) $\frac{\langle A_i, A_j\rangle}{\|A_i\|\|A_j\|}$ is 0. If, in addition, each vector has unit norm $\|A_i\|_2 = 1$, the set is **orthonormal**. 
 > 
 > _Orthogonality implies linear independence_, because if $c_1 A_1 + c_2 A_2 + \dots + c_k A_k = 0$, then  $\langle A_i, c_1 A_1 + \dots + c_k A_k \rangle = 0$. Since $\langle A_i, c_1 A_1 + \dots + c_k A_k \rangle= \langle A_i, c_i A_i \rangle = c_i \|A_i\|_2^2$ also holds due to pairwise orthogonality, where $\|A_i\|_2^2 > 0$, $c_i$ must be zero 0. On the other hand, _linear independence does not imply orthogonality_; for example, $(1,0)$ and $(1,1)$ are linearly independent but not orthogonal.  A set of linearly independent vectors can be orthogonalized with the [Gram-Schmidt orthogonalization](https://en.wikipedia.org/wiki/Gram–Schmidt_process).
 > 
 > All these mean that if $A_i$ are the columns of a matrix $A$, then 
 > - the columns are orthogonal $\iff$ $A^T A$ is diagonal
 > - the columns are orthonormal $\iff$ $A^T A = \mathbf{I}$  
 > - the columns are linearly independent $\iff$ $Ax=0$ only for $x=0$, that is, the only vector in the null space of $A$ is the zero vector. In that case $x^\top (A^\top A) x = 0$ only for $x = 0$, because $x^\top (A^\top A) x = (A x)^\top (A x) = \|A x\|_2^2$, and $\|A x\|_2^2 = 0$ only if  $A x = 0$. This means that $A^\top A$ is positive definite $\iff \text{det}(A^\top A) > 0 \iff A^\top A$ is invertible and symmetric, but not necessarily diagonal
 
Because $A$ is restricted to be binary (each query either includes $x_i$ or not), there is in fact only a single $n \times n$ binary matrix with orthogonal columns: the standard basis (aka identity matrix), in which each column is a unit (one-hot) vector. For example, for four records (and queries) the standard basis is:
$$
\mathbf{I} = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & 1 \end{bmatrix}
$$
>[!FAQ]- Why?
>We want  that $\langle A_{i}, A_{j} \rangle = 0$  for all $i \neq j$, which is only possible if, for any position $j$, there is at most one column which has a value of 1 at position $j$. Since $A$ is a squared and binary matrix with non-zero column vectors, this is only possible if every column has 1 at a single position, or in other words, if each column is a different unit vector.  

The problem with using the standard basis $\mathbf{I}$ is that each query covers only a single record. This corresponds to a trivial disclosure with zero anonymity and would never be answered by a database manager providing $k$-anonymity for any $k > 1$. We are interested in the more realistic case where every query covers multiple records, that is, $A$ is a binary matrix in which each row contains multiple ones. However, based on the argument above, no such square binary matrix with orthogonal columns exists.

However, the situation changes if we allow negative entries in the query matrix, that is, we can not only add but also subtract attribute values from the aggregates. Consider the _Hadamard matrix_, where the first row and column are constant $1$, and every other row and column contains an equal number of $+1$ and $-1$. For example, the Hadamard matrix of size 4 is
$$
H_4 = \begin{bmatrix} 1 & 1 & 1 & 1 \\ 1 & -1 & 1 & -1 \\ 1 & 1 & -1 & -1 \\ 1 & -1 & -1 & 1 \end{bmatrix}
$$
In general, a Hadamard matrix of size $n$ can be constructed recursively as
$$
H_n = \begin{bmatrix} H_{n/2} & H_{n/2} \\ H_{n/2} & -H_{n/2} \end{bmatrix},
$$
with $H_1 = [1]$. Because Hadamard matrices are orthogonal by construction, every pair of columns (and rows) is linearly independent.

Although the Hadamard matrix is not strictly binary (entries are $\pm 1$ rather than $0/1$), its rows are the linear combination of strictly binary queries; the sum of the records with negative entries in a row can be expressed as the total sum of all records minus the sum of the records with positive entries. More precisely, define a binary matrix $H’$ by replacing each $-1$ in $H$ with $0$:
$$
H’_{i,j} = \begin{cases} 1, & \text{if } H_{i,j} = 1, \\ 0, & \text{if } H_{i,j} = -1. \end{cases}
$$
Then, for any row $i$,
$$
\langle H_{i,:}, x \rangle = \langle H’_{i,:}, x \rangle - \left(\sum_i x_i - \langle H’_{i,:}, x \rangle \right)= \langle H’_{i,:}, x \rangle - \big(\langle H’_{1,:}, x \rangle - \langle H’_{i,:}, x \rangle \big) = 2 \langle H’_{i,:}, x \rangle - \langle H’_{1,:}, x \rangle.
$$
where $H’_{1,:} = H_{1,:}$ is the vector of all-ones, hence $\sum_i x_i = \langle H’_{1,:}, x \rangle$. This means the adversary can request the database to evaluate all binary queries in $H’$, obtaining $b’ = H’ x$. From these query results, the adversary can reconstruct the original Hadamard query responses $b = Hx$, and then solve the resulting system of equations to recover $x$. The Hadamard matrix is particularly appealing since each query (row) covers half of the records (so it appears "highly aggregated" and compliant with $k$-anonymity type privacy rules), and the resulting system can be inverted efficiently using the fast Walsh–Hadamard transform in $\mathcal{O}(n\log n)$ time.

Conclusion: **The adversary can recover every record using a number of queries equal to the total number of records, even though each query aggregates over half of the database.**
## Can the adversary reconstruct all $n$ records with less than $n$ queries?

If we have more unknowns (records) than equations (queries), the above system of linear equations $Ax=b$ is underdetermined and cannot be solved in general. However, it is no longer the case if $x$ is sparse, that is, it has less than $m$ non-zero elements like in the hospital dataset above where we have only a single HIV-positive patient. Indeed, we only need to determine the non-zero elements in $x$ whose number is significantly smaller than $n$. Vectors with $s$ nonzero entries are also called $s$-sparse vectors, and the set of non-zero positions is called the support of this vector.

Our objective is to **minimize the number of queries** $m$, given that the dataset is $s$-sparse, i.e., it contains only $s \ll n$ nonzero entries. Equivalently, we seek a query matrix $\mathbf{A}$ with as few rows as possible. Trivially, $m \leq n$, as shown above. On the other hand, $m \geq s$, since $\mathbf{A}x$ is a linear combination of the columns of $\mathbf{A}$. If we knew **which** $s$ coordinates were nonzero, we could simply query those coordinates with $s$ linearly independent rows, which would be sufficient for recovery (because the number of linearly independent rows equals the number of independent columns). Thus, in principle, the answers to $\mathcal{O}(s)$ queries contain enough information.

However, the indices of the nonzero entries is unknown. A classical result in linear algebra states that if every subset of $2s$ columns of $\mathbf{A}$ is linearly independent, then the sparsest solution to $\mathbf{A}x = b$ is **unique** for any $s$-sparse vector $x$. Consequently, $2s$ linear (not necessarily binary) queries are sufficient to guarantee unique recovery of any $s$-sparse vector.

>[!FAQ]- Why?
> We will first show that $Ax=b$ has a unique $s$-sparse solution if and only if ($\iff$) there is no vector with sparsity at most $2s$ (except the zero vector) that $A$ maps to 0 (in other words, the null space of $A$ contains no nonzero vector with sparsity $\leq 2s$).
> ($\implies$:) Suppose there exists a vector $v$ with $\leq 2s$ nonzero entries such that $Av=0$. Write $v=u-z$, where both $u$ and $z$ has support size  $\leq s$ and $u\neq z$. Such a decomposition is always possible by splitting the nonzero coordinates of $v$ into two disjoint parts, each of size at most $s$, and assigning them to $u$ and $z$, respectively. Then, $A(u-z)=Av=0 \implies Au=Az$. Since $u\neq z$ by construction, $u$ is not a unique $s$-sparse solution, yielding a contradiction. 
>($\impliedby$:) Conversely, let $x$ and $y$ be $s$-sparse vectors such that $A x = b$ and $A y = b$. Then, $A(x-y) = 0$, where the difference vector $x-y$ has at most $2s$ nonzero entries (since both $x$ and $y$ are $s$-sparse). Hence, if the null space of $A$ contains only the zero vector with support size $\leq 2s$ , it follows that $x=y$. 
> Finally, if $A$ does not map any nonzero vector with $\leq 2s$ nonzeros to 0, then by the definition of linear independence, every subset of $2s$ columns of $A$ must be linearly independent. Therefore, the uniqueness of the sparse solution is guaranteed if and only if every set of $2s$ columns of $A$ is linearly independent.

This proof suggests a simple **brute-force search** to recover $x$: enumerate all possible $s$-sparse candidates $x$ and select the one that satisfies $\mathbf{A}x = b$, which must be the single correct solution. Unfortunately, this requires solving $\binom{n}{s}$ linear systems, which quickly becomes infeasible as $s$ grows. In fact, it can be shown that finding the sparsest solution (i.e., solving the $\ell_0$-minimization problem) is **NP-complete** for arbitrary choices of $\mathbf{A}$ and $x$ (unless $\mathrm{P} = \mathrm{NP}$). Fortunately, for specially designed matrices $\mathbf{A}$, there exist efficient algorithms that can recover $s$-sparse signals. These will be discussed next.

We're looking for a $m\times n$ rectangular matrix such that every set of $2s$ columns of this matrix is linearly independent. More formally, the question: is there a rectangular $(m\times n)$ matrix whose **any** $(m \times m)$-sized submatrix is invertible? One construction is the Vandermonde matrix:
$$
\begin{bmatrix}
1 & c_0 & c_0^2 & \ldots & c_0^{n-1} \\
1 & c_1 & c_1^2 & \ldots & c_1^{n-1} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & c_{m-1} & c_{m-1}^2 & \ldots & c_{m-1}^n
\end{bmatrix}
$$
if $c_0, \ldots, c_{m-1}$ are all **positive and distinct**.

>[!FAQ]- Why?
>It's enough to show that the determinant of any $(m\times m)$-sized submatrix of the Vandermonde matrix is non-zero with the above conditions. Let $a_{\lambda}=det(c_i^{\lambda_j})$ where $\lambda=(\lambda_1, \lambda_2, \ldots, \lambda_{\ell})$ is a weakly increasing sequence of non-negative integers. Using this notation, $a_\delta$ with $\delta=(0, 1, \ldots, n-2, n-1)$ defines the determinant of  a squared Vandermonde matrix. If $a_{\lambda+\delta} = det(c_i^{\lambda_j+\delta_j})$, then $a_{\lambda+\delta} = s_{\lambda} \cdot a_{\delta}$ where $s_{\lambda}$ is the [Shur function](https://en.wikipedia.org/wiki/Schur_polynomial). Since the Vandermonde determinant $a_{\delta}$ [is always non-zero if $c_i$ are all distinct](https://en.wikipedia.org/wiki/Vandermonde_matrix), we have to show that the Shur function $s_{\lambda}$ is non-zero. This follows from the [combinatorial definition of Shur function](https://users.math.msu.edu/users/bsagan/Papers/Old/schur.pdf) if all $c_i$ are positive (selecting columns $k_1, k_2, \ldots, k_m$ from the $m\times n$-sized Vandermonde matrix, $\lambda=(k_m-m, k_m-m+1, \ldots,  k_2-2, k_1-1)$). 

## Error correcting codes for reconstruction

We require a binary query matrix $A$, where an attribute $x_j$ is either included in the aggregate ($A_{i,j} = 1$) or excluded ($A_{i,j} = 0$) for a given query. As shown above, there is no such square matrix that allows reconstruction of $x$, except for the trivial and non-private choice of the standard basis. However, this limitation no longer holds if we consider finite field arithmetic, which connects the problem to *binary error-correcting codes*.  

In particular, a practical instantiation of a Vandermonde matrix is the parity-check matrix of a binary BCH code:
$$
H=\begin{bmatrix}
1 & \alpha_0 & \alpha_0^2 & \ldots & \alpha_0^{n-1} \\
1 & \alpha_1 & \alpha_1^2 & \ldots & \alpha_1^{n-1} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & \alpha_{m-1} & \alpha_{m-1}^2 & \ldots & \alpha_{m-1}^n
\end{bmatrix}
$$
where $\{\alpha_0, \ldots, \alpha_{m-1}\}$ are distinct elements of a finite field of size at least $n$. In practice, the field is typically $GF(2^q)$, while the code symbols are restricted to $GF(2)$, yielding binary codewords. The code length is then $n = 2^q - 1$, and if $m = 2s$, the BCH code can correct up to $s$ errors in a codeword as long as all $\alpha_i$ are distinct field elements. Therefore, if $x$ is interpreted as a potentially corrupted codeword of this BCH code, then the nonzero locations of $x$ can be identified from its syndrome $H \cdot x$ using any efficient BCH decoding procedure. This is because the zero vector is always a valid codeword of linear code, and all vectors with at most $2s$ nonzero entries will decode to the zero vector. Put simply, once $H \cdot x$ can be computed, the adversary can in principle solve for $x$ in $GF(2^q)$. 

The problem that $H$ is not binary, its entries live in $GF(2^q)$. Therefore, the inner products $\langle H_i, x \rangle$ must be computed over $GF(2^q)$, requiring finite-field arithmetic, which is seemingly not realizable in SQL. However, one can expand each field element of $GF(2^q)$ into a binary vector of length $q$ (via a basis representation of the field). With this expansion, the parity-check matrix can be represented as a binary matrix which means that a single query $\langle H_i, x \rangle$ can be realized with exactly $q=\log(n)$ binary queries. 

>[!FAQ]- Why?
> Each element of finite field $GF(2^q)$ can be represented as polynomials over $GF(2)$, reduced modulo an irreducible polynomial of degree $q$. Field operations (addition and multiplication) are therefore carried out as polynomial operations modulo this irreducible polynomial. 
> 
> For a concrete example, consider $q=3$, i.e., the field $GF(2^3)$. This field has $2^3=8$ elements. A common irreducible polynomial of degree 3 is $p(z) = z^3 + z + 1$. Each element of $GF(2^3)$ can be expressed as a binary polynomial of degree at most 2, with coefficients in $\{0,1\}$. Using the basis $\{1, z, z^2\}$, we can represent the field elements by 3-bit vectors (coefficients of $1, z, z^2$):
> $$
> \begin{align}
> 0 &\mapsto (0,0,0)\\ 
> 1 &\mapsto (1,0,0)\\
> z &\mapsto (0,1,0)\\
> z^2 &\mapsto (0,0,1)\\
> z^3 = z+1 &\mapsto (1,1,0)\\
> z^4 = z^2+z &\mapsto (0,1,1)\\
> z^5 = z^2+z+1 &\mapsto (1,1,1)\\ 
> z^6 = z^2+1 &\mapsto (1,0,1)
> \end{align}
> $$
> If we denote by $\alpha$ a primitive element of $GF(2^3)$ (e.g., $\alpha=z$), then $H_i = \{1, \alpha^i, \alpha^{2i}, \ldots, \alpha^{6i}\}$ is the $i$th row of a valid parity matrix. Following the above polynomial representation,
> $\alpha^3 = \alpha+1$, $\alpha^4 = \alpha^2 + \alpha$, and so on. Thus, each $\alpha^i$ has a unique binary vector representation.
> For example, $H_1 = [1, \alpha, \alpha^2, \ldots, \alpha^6]$ has a binary form $A_{1} = \begin{bmatrix} 1 & 0 & 0 & 1 & 0 & 1 & 1 \\ 0 & 1 & 0 & 1 & 1 & 1 & 0 \\ 0 & 0 & 1 & 0 & 1 & 1 & 1 \end{bmatrix}$. 
> 
> When computing the dot product $\langle H_i, x \rangle$ in $GF(2^3)$, the operation in polynomial form is equivalent to evaluating the same dot product in the vector space representation over $GF(2)$. In other words, $\langle H_i, x \rangle$ in $GF(2^q)$ corresponds exactly to $\langle A_i, x \rangle$ in $GF(2)$, where each field element is expanded into its binary vector representation. For example, let $x = (1,0,1,0,0,0)$. Then, in the field representation, $\langle H_1, x \rangle = 1 + \alpha^2 = \alpha^6$. In the vector space representation, this is computed as $\langle A_1, x \rangle = (1,0,0) \oplus (0,0,1) = (1,0,1)$, which also corresponds to the polynomial $z^2 + 1$.

Therefore, the adversary can transform $H$ into a binary matrix $A$ with $m \cdot q = m \cdot \log_2(n+1)$ rows. By querying the database to compute $A \cdot x$ instead of $H\cdot x$ using standard integer arithmetic, and then reducing the result elementwise modulo 2, the adversary obtains the binary representation of the syndrome $H \cdot x$ in $GF(2)$ which then can be directly mapped to its original representation in $GF(2^q)$. Finally, standard BCH decoding can be applied on $H \cdot x$ in $GF(2^q)$ to efficiently reconstruct $x$. In total, this reconstruction procedure requires  $2s \cdot \log_2(n+1) = \mathcal{O}(s \log_2 n)$ **binary** queries. 

How many records are covered by a binary query $A_i$? It is not hard to see that if $\alpha_i$ used in the parity check matrix $H$ is a primitive element of $GF(2^q)$, then _every_ query $A_i$ covers about half of the records. 

>[!FAQ]- Why?
>If $\alpha_i$ is a primitive element of $GF(2^q)$, then it has multiplicative order $n$. Therefore, each entry of $H_i$ is a distinct field element, which in the binary expansion maps to a unique nonzero $q$-bit vector. Each row of $A_i$ corresponds to a single bit position across all these vectors. Since there are $2^q - 1$ nonzero vectors in total, and exactly half of the $2^q$ possible vectors have a 1 in any given bit position, each row of $A_i$ contains $2^{q-1}$ ones and $2^{q-1}-1$ zeros. Thus, every binary query covers exactly $2^{q-1}=(n+1)/2$ records.

Conclusion: **An adversary can exactly reconstruct all records in a database containing $s$ nonzero entries using only $2s \log n$ binary queries, even though each individual query covers roughly half of the records, that is, they are $(n/2)$-anonym.**

For the single patient with the rare disease in the introductory example ($s=1$), even fewer queries are enough, if this query matrix is the parity-check matrix of the binary [Hamming code](https://en.wikipedia.org/wiki/Hamming_code), which is the BCH code for $s=1$: Each query covers about half of the records. If a query covers fewer than half, the adversary asks its complement instead (the patients with a 0 in the $k$-th bit, which then cover more than half) and flips the answer. For the hospital with 100,000 patients, this means only $\lceil \log_2 100{,}000 \rceil = 17$ queries, each covering at least 50,000 patients.

>[!FAQ]- How?
>In SQL, this attack only needs bit operations on a numeric patient ID. Query $k$ selects the patients whose ID has a 1 in the $k$-th bit, and sums the sensitive attribute over them: 
>```
> SELECT SUM(CASE WHEN diagnosis = 'rare disease' THEN 1 ELSE 0 END) FROM Patients WHERE (id >> k) & 1 = 1 
>```
>  Note that the WHERE clause contains only the bit condition, so each query covers about half of the patients, and the sensitive attribute appears only inside the aggregate. If the database does not support bit shifts, `MOD(FLOOR(id / POWER(2, k)), 2) = 1` selects the same patients. 
> The toy example below has 8 patients, numbered from 0 to 7, where only patient 5 has the sensitive attribute. Three queries, each covering half of the patients, are enough to identify this patient unambiguously.
> $$
>\begin{array}{l|cccccccc|c} \text{Patient } j & 0 & 1 & 2 & 3 & 4 & \mathbf{5} & 6 & 7 & b_i = \langle A_{i,:}, x \rangle \\ \hline \text{Sensitive attribute } x_j & 0 & 0 & 0 & 0 & 0 & \mathbf{1} & 0 & 0 & \\ \hline A_{0,:} \text{ (bit 0 = 1)} & 0 & 1 & 0 & 1 & 0 & \mathbf{1} & 0 & 1 & 1 \\ A_{1,:} \text{ (bit 1 = 1)} & 0 & 0 & 1 & 1 & 0 & \mathbf{0} & 1 & 1 & 0 \\ A_{2,:} \text{ (bit 2 = 1)} & 0 & 0 & 0 & 0 & 1 & \mathbf{1} & 1 & 1 & 1 \end{array}
>$$
>  For example, suppose the patients have IDs from 0 to 99,999, and the patient with ID 70,000 (binary `1 0001 0001 0111 0000`) has the rare disease. The adversary runs the query above for $k = 0, 1, \ldots, 16$. The query for $k=4$ returns 1, since the 4th bit of 70,000 is 1, while the query for $k=0$ returns 0. The only exception is $k = 16$: only the 34,464 patients with IDs from 65,536 to 99,999 have a 1 in this bit, which is below the 50,000 threshold. So the adversary asks for the complement instead (`WHERE (id >> 16) & 1 = 0`, covering 65,536 patients), receives 0, and concludes that the bit is 1. The 17 answers spell out `1 0001 0001 0111 0000`, the ID of the patient with the rare disease.

## Stable reconstruction

The reconstruction techniques described above work only under strong assumptions: namely, that the sparsity $s$ of $x$ is known in advance, that the database manager returns **exact** query results (i.e., without perturbations as in Differential Privacy), and that the adversary has fairly accurate knowledge of patients' non-sensitive attributes (those appearing in the WHERE clause of SQL queries) in order to select the exact records targeted by the pre-defined queries. If these conditions are not satisfied, the recovered vector may significantly differ from the true $x$, rendering these techniques unstable. This concern is realistic; for example, an adversary may not know the exact number of AIDS patients, especially if the number is small, and querying it directly would violate $k$-anonymity.

Moreover, the binary code construction described earlier is not even asymptotically optimal in terms of the number of **binary** queries required. While a naive argument suggests that identifying a single non-zero position among $n$ requires $\log_2(n)$ bits and thus at least $s \log_2(n)$ binary queries in total, support recovery does not require distinguishing every position individually. Instead, to uniquely identify all subsets of size s, one needs 
$$
\log_2 \binom{n}{s} \;\leq\; \log_2 \left(\frac{e n}{s}\right)^s \;=\; \frac{s \big(1 + \ln(n/s)\big)}{\ln 2} \;=\; \Omega\!\big(s \ln(n/s)\big)
$$
binary queries.

This raises a natural question: is there a **stable** reconstruction technique that does not rely on rigidly pre-defined query structures? The answer comes from _compressed sensing_, which provides methods that are not only stable but also asymptotically optimal. Using a carefully designed random query matrix, the support of an $s$-sparse vector can be reconstructed with only $\mathcal{O}(s \ln(n/k))$ queries, while tolerating noise and perturbations in the responses. As a plus, the same techniques also work for non-binary queries!

We want a matrix $A$ such that every submatrix $A_S$ (formed by selecting $m$ columns of $A$) is **close to orthogonal**. Orthogonality guarantees that the columns are linearly independent, meaning the dot product between any pair of columns (rows) is zero, or equivalently, $A_S^\top A_S = A_S A_S^\top = \mathbf{I}$. In practice, exact orthogonality is often too strong to require, so we instead demand that $A_S^\top A_S$ be close to the identity matrix, or in other words,  $(A_S^{T}\cdot A_S - \mathbf{I})$ is small according to a suitable norm, such as the matrix (or operator) norm. Formally, this means that $\max_{\|u\|_2 \leq 1} \|(A_S^\top A_S - \mathbf{I})u\|_2 = \delta$ is as small as possible. Intuitively, this condition says that $A_S^\top A_S - \mathbf{I}$ squeezes any unit vector to almost zero. Equivalently, since $A_S^\top A_S - \mathbf{I}$ is symmetric, we have for any vector $x$:
$$
|x^\top (A_S^\top A_S - \mathbf{I})x| \leq \delta \|x\|_2^2 .
$$
>[!FAQ]- Why?
>   We'll show that $\max_v ||Mv||_2=\max_v v^T M v=\max_i|{\lambda_i}|$ for $v$ unit vectors where $\{\lambda_{i}\}$ are the eigenvalues of $M$. 
> 
> Since $M$ is symmetric, the [Spectral theorem](https://eecs16b.org/notes/sp24/note14.pdf) guarantees that its eigenvectors $\{q_i\}$ form an orthonormal basis with real eigenvalues $\{\lambda_i\}$. Therefore any unit vector $v$ can be expanded as  $v=\sum_i\alpha_iq_i$ . Thus, 
> $$
> Mv = \sum_i \alpha_i M(q_i) = \sum_i \alpha_i \lambda_i q_i \leq \max_i|\lambda_i| \sum_i \alpha_i q_i = \max_i|\lambda_i| \cdot v
> $$
> by the definition of eigenvectors and eigenvalues.
> Then, 
> $$
> v^\top M v \leq  \max_i |\lambda_i| v^\top v = \max_i |\lambda_i|
> $$
> and similarly,  
> $$
> \|Mv\|_2 \leq \max_i |\lambda_i| \cdot \|v\|_2 =\max_i |\lambda_i|
> $$
> for any $v$ unit vector. In both cases, the maximum is attained when $v$ equals the eigenvector corresponding to the eigenvalue with the largest magnitude. This implies that $||Mv||_2 \leq \delta \iff v^T M v \leq \delta$.

But the left-hand side equals
$$
|x^T \cdot (A_S^{T}\cdot A_S - \mathbf{I})\cdot x|= \left| ||A_S \cdot x||_2^2 - ||x||_2^2 \right|  
$$
so we obtain:
$$
\left| ||A_S \cdot x||_2^2 - ||x||_2^2 \right| \leq \delta 
$$
and hence:
$$
(1-\delta)\|x\|_2^2 \;\leq\; \|A_S x\|_2^2 \;\leq\; (1+\delta)\|x\|_2^2.
$$
Thus, when $\delta$ is small, the “stretching” or “shrinking” effect of the transformation $A_S$ is negligible: $A_S$ acts almost like an isometry that preserves $\ell_2$-norms (and hence distances between vectors). Orthogonal matrices achieve this exactly, since they preserve the inner product: $\langle A_Sx, A_Sy \rangle = x^\top (A_S^\top A_S) y = x^\top y = \langle x,y\rangle$ and therefore the $\ell_2$-norm:  $||x||_2 = \langle x, x \rangle^{1/2} = \langle A_S x, A_S x \rangle^{1/2} = ||A_Sx||_2$. The constant $\delta$ measures how far $A_S$ deviates from perfect orthogonality; if the same $\delta$ works for all submatrices $A_S$ of $A$, it is called the **restricted isometry constant** of $A$.

## Random query matrix

Let $A_{i,j}$ be $1$ or $-1$ with equal probability for every $i,j$ independently. Then, for any row $A_i$, 
$$
E|\langle A_i, x\rangle|^2 = E\left[\sum_{k,\ell} x_\ell x_k A_{i,\ell} A_{i,k}\right] =  \sum_{j} x_j x_j E[A_{i,j} A_{i,j}] = E\left[A_{i,j}^2\right]\sum_{\ell} x_\ell^2 = \| x \|_2^2
$$
due to independence (i.e., $E[A_{i,\ell}A_{i,k}]=1$ for $k=\ell$ and 0 otherwise) and  $E\left[A_{i,j}^2\right]= 1^2 \cdot 0.5 + (-1)^2 \cdot 0.5=1$ is the variance. For the whole matrix $A$:
$$
E(\|A x \|_2^2) = E\left(\sum_{i=1}^m|\langle A_i, x\rangle|^2\right) =  \sum_{i=1}^mE|\langle A_i, x\rangle|^2=m \cdot \|x\|_2^2
$$
which means that, on expectation, the matrix $\frac{1}{\sqrt{m}}A$ will preserve the norm and has a good chance to be "quasi-orthogonal". However, to have a better estimate on how close we are to perfect orthogonality, we need to know how well $E(\frac{1}{m}\|A x \|_2^2)$ concentrates around its mean $||x||_2^2$, that is, what's the probability that $\frac{1}{m}\|A x \|_2^2 - \|x\|_2^2$ is outside of a certain range of 0. 

We will derive a more general result for a wider family of random matrices that will reveal a few  design principles about the queries.  We will show that if the elements of the matrix $\mathbf{A}$ “tightly” concentrate around their mean 0, then the same holds for the inner products $\langle A_i, x\rangle$. Consequently, the squared quantities $\langle A_i, x\rangle^2$ also “tightly” concentrate around their mean value $\|x\|_2^2$, and therefore the average $\frac{1}{m}\|\mathbf{A}x\|_2^2$ does as well. More precisely, if the entries of $A$ are subgaussian, then it follows that both the random variables $\langle A_i, x\rangle^2$ and $\frac{1}{m}\|\mathbf{A}x\|_2^2$ are subexponential in $m$. Intuitively, this means that the probability that $\mathbf{A}$ deviates from perfect orthogonality is exponentially small in the number of rows $m$. Thus, the larger the number of rows, the more “orthogonal” the columns of $\mathbf{A}$ become, on average.

>[!FAQ]- Subgaussian vs Subexponential concentration
>A random variable $X$ has subgaussian concentration (or subgaussian tails) if the probability that it deviates from its mean by more than a threshold $t$ decays exponentially fast. Formally, $\Pr(|X - E[X]| \geq t) \leq \beta e^{-\kappa t^2}$ for all $t >0$ and  some constants $\beta, \kappa > 0$. Sub-exponential distribution has fatter tails as they are bounded above with exponential tails; $\Pr(|X - E[X]| \geq t) \leq \beta e^{-\kappa t}$. Hence, they decay slower than Gaussian, albeit still exponentially fast. For example, the normal (Gaussian) distribution is subgaussian, or any **bounded random variable** (such as the Bernoulli random variables taking values $\pm 1$  with equal probability), as guaranteed by [Hoeffding’s inequality](https://en.wikipedia.org/wiki/Hoeffding%27s_inequality). 

But why do we want subexponential concentration? Because we want "approximate orthogonality" to hold not just for a single submatrix of $A$, but _any_ submatrices of a given size $s$ simultaneously. Indeed, if $\frac{1}{m}\|\mathbf{A}_S x\|_2^2$ is subexponential then roughly 
$$
\Pr(\text{$A_S$ is NOT orthogonal with error $t$}) = \Pr\left(\left|\frac{1}{m}\|\mathbf{A}_Sx\|_2^2 - \|x\|_2^2\right| \geq t\right) \leq \beta e^{-\kappa m t^2}$$
Applying a union bound over all possible subsets $S$ of size $s$, we obtain
$$
\Pr(\text{any submatrix $\mathbf{A}_S$ is NOT orthogonal}) \le \binom{n}{s} \, \beta e^{-\kappa m}
$$
We want this overall failure probability to be much smaller than 1. Therefore, we require
$\binom{n}{s} \, \beta e^{-\kappa m} \ll 1$. _Taking logarithms_ on both sides gives the condition:
$$
m \ge \frac{1}{\kappa}\ln \binom{n}{s} + \frac{1}{\kappa}\ln \beta.
$$
Using the standard combinatorial upper bound $\binom{n}{s} \le (en/s)^s$ and ignoring constants $\kappa$ and $\beta$, we find that
$$
m = \mathcal{O}(s \ln(n/s))
$$
In other words, subexponential concentration is useful because it gives you exponentially small failure probabilities for each event, which allows us to extend guarantees that hold for a single submatrix to _exponentially many_ submatrices _simultaneously_, _without requiring exponentially many queries (rows)_.

Next, we will provide more mathematical details on how to prove subexponential (or subgaussian) concentration using one of the most versatile tools in the theory of randomized algorithms: the Chernoff bound. In particular, we will show the following chain of implications for a single submatrix $A_S$:

Any entry of $A_S$ is subgaussian $\stackrel{\text{Step 1}}{\implies}$ $\langle A_i, x\rangle$ is subgaussian $\stackrel{\text{Step 2}}{\implies}$ $\langle A_i, x\rangle^2$ is subexponential $\stackrel{\text{Step 3}}{\implies}$ $\frac{1}{m}\|A_S x \|_2^2$ is subexponential $\implies$ $A_S$ is "quasi-orthogonal"

Finally, we extend this subexponential concentration for all submatrices of $A$.

### Concentration of a single submatrix

To decide if a random variable $X$ is subgaussian (or subexponential), one need to check the [moment generating function (MGF)](https://en.wikipedia.org/wiki/Moment-generating_function) $\mathbb{E}[e^{\lambda X}]$ of $X$, which is intimately related to the tail bound based on [Chernoff bounds](https://en.wikipedia.org/wiki/Chernoff_bound):
$$ P(X-E[X] \geq t) \leq \inf_{\lambda>0}e^{-\lambda t}\,E\left[e^{\lambda(X-E[X])}\right] $$
>[!FAQ]- Why? 
>By Markov's inequality, for any non-negative random variable $Y$ and any $a>0$:
>$$ 
>P(Y \geq a) \leq \frac{E[Y]}{a} 
>$$ 
>Since $x \mapsto e^{\lambda x}$ is increasing for $\lambda>0$, the event $X-E[X] \geq t$ is the same as $e^{\lambda(X-E[X])} \geq e^{\lambda t}$. Applying Markov's inequality to the non-negative random variable $Y = e^{\lambda(X-E[X])}$ with $a = e^{\lambda t}$: 
>$$ 
>P(X-E[X] \geq t) = P\left(e^{\lambda(X-E[X])} \geq e^{\lambda t}\right) \leq e^{-\lambda t}\,E\left[e^{\lambda(X-E[X])}\right] 
>$$ 
>This holds for every $\lambda >0$, so it also holds for the smallest right-hand side: 
>$$ 
>P(X-E[X] \geq t) \leq \inf_{\lambda>0}e^{-\lambda t}\,E\left[e^{\lambda(X-E[X])}\right] 
>$$ 
>Therefore, the upper tail of $X$ is bounded by the moment generating function of the centered variable $X-E[X]$, multiplied by $e^{-\lambda t}$ and optimized over $\lambda>0$.

One direct implication of Chernoff bound that if $\mathbb{E}[e^{\lambda X}] \;\le\; \exp\!\left(c \lambda^2\right)$ for a constant $c$, then $X$ is subgaussian: $\Pr(|X| \geq t) \leq 2\cdot e^{-\frac{t^2}{4c}}$

>[!FAQ]- Why?
>For any $t>0$ and any $\lambda>0$, Chernoff gives
>$$
>\Pr(X\ge t)=\Pr(e^{\lambda X}\ge e^{\lambda t}) \le e^{-\lambda t}\,\mathbb{E}e^{\lambda X} \le \inf_{\lambda>0}\exp\!\Big(c\lambda^2-\lambda t\Big)
>$$
>The quadratic in the exponent is minimised at $\lambda = \frac{t}{2c}$. Plugging in gives
>$$
>\Pr(X\ge t)\le \exp\!\Big(-\frac{t^2}{4c}\Big)
>$$
>Applying the same argument on $-X$, we can prove that $\Pr(X \leq -t) \leq \exp\left(-\frac{t^2}{4c}\right)$ for the other tail. Combining the two tails by a union bound we get $\Pr(|X| >t)  \leq 2\cdot\exp\left(-\frac{t^2}{4c}\right)$, which means $X$ is subgaussian.

In fact, for any centered subgaussian random variable $X$, $\mathbb{E}[e^{\lambda X}] \leq e^{c \lambda^2/2}$ implies that $E[X^2] \leq c$ that is also called the proxy variance of $X$. 

>[!FAQ]- Why?
>Let $\mu = E[X]$. It follows from the Taylor expansion of the exponential function and the linearity of expectation that  
>$$
>\begin{align}
>\mathbb{E}[e^{\lambda (X -\mu)}] &= 1 + \lambda E[X-\mu] + \frac{\lambda^2 E[(X-\mu)^2]}{2!} + \frac{\lambda^3  E[(X-\mu)^3]}{3!} + \ldots \\
>&=1 + \lambda E[X-\mu] + \frac{\lambda^2 E[(X-\mu)^2]}{2!} + \mathcal{O} (\lambda^3)\\
>&=1 +  \frac{\lambda^2 E[(X-\mu)^2]}{2!} + \mathcal{O} (\lambda^3)
>\end{align}
>$$
>because $E[X-\mu] =0$.
>Similarly, 
>$$
>\begin{align}
>e^{c \lambda^2/2} = 1 + \frac{\lambda^2 c}{2} + \frac{\lambda^4 c^2}{4} + \ldots = 1 + \frac{\lambda^2 c}{2} + \mathcal{O}(\lambda^4)
>\end{align}
>$$
>Therefore, if $\mathbb{E}[e^{\lambda (X-\mu)}] \leq e^{c \lambda^2/2}$ then 
>$$
>1 +  \frac{\lambda^2 E[(X-\mu)^2]}{2!} + \mathcal{O} (\lambda^3) \leq  1 + \frac{\lambda^2 c}{2} + \mathcal{O}(\lambda^4)
>$$
>Dividing both sides by $\frac{1}{2}\lambda^2$:
>$$
>E[(X-\mu)^2] + \mathcal{O} (\lambda^2) \leq  c + \mathcal{O}(\lambda)
>$$
>This must also hold for all $\lambda$ even when $\lambda \rightarrow 0$, in which case $\mathcal{O}(\lambda^2)$ and $\mathcal{O}(\lambda)$ vanish yielding $E[(X-\mu)^2  \leq c$.

From now on, we’ll assume that $X = A_{i,j}$ is a subgaussian random variable: $\mathbb{E}[e^{\lambda X}] \le \exp\left(c \lambda^2/2\right)$ with proxy variance $E[X^2] \leq c$.

For example, when $A_{i,j}=1$ or $A_{i,j}=-1$ with probability $0.5$, the moment generating function (MGF) of $X = A_{i,j}$ is
$$
\mathbb{E}[e^{\lambda X}] = 0.5\, e^{\lambda} + 0.5\, e^{-\lambda} = \cosh(\lambda) \le e^{\lambda^2 / 2}
$$
and therefore $\Pr(|X| \geq t) \leq 2\cdot e^{-\frac{t^2}{2}}$ (the same bound can also be derived from [Hoeffding's inequality](https://en.wikipedia.org/wiki/Hoeffding%27s_inequality)). The proxy variance $2\cdot (1/2)=1$ equals the true variance.

#### Step 1: $A_{i,j}$ is i.i.d subgaussian $\implies$ $\langle A_i, x\rangle$ is subgaussian

One can show that if $A_{i,j}$ is subgaussian with proxy variance $c$, then $\langle A_i, x\rangle$ is also subgaussian with proxy variance $c \|x\|_2^2$.

>[!FAQ]- Why?
>Suppose $X=A_{i,j}$ is subgaussian: $\mathbb{E}[e^{\lambda X}] \;\le\; \exp\!\left(c \lambda^2/2\right)$.  Hence:
>$$
>\begin{align}
>\mathbb{E}[\exp(\lambda \sum_{j} A_{i,j} \cdot x_j)] &= \mathbb{E}\left[\prod_j [\exp(\lambda \cdot A_{i,j} \cdot x_j)\right]\\
> &= \prod_j \mathbb{E}[\exp(\lambda \cdot x_j \cdot A_{i,j})] \\
>& \leq \prod_j \exp(c \cdot x_j^2\lambda^2/2) \\
>& = \exp(0.5\cdot c \cdot \lambda^2 \cdot \sum_j x_j^2) \\
>& = \exp(c \|x\|_2^2 \cdot \lambda^2/2 )
>\end{align}
>$$
>where $c\dot \|x\|_2^2/2$ is constant, therefore $\langle A_i, x\rangle$ is also subgaussian.  

#### Step 2: $\langle A_i, x\rangle$ is subgaussian $\implies$ $\langle A_i, x\rangle^2$ subexponential 
	
Recall that $\mathbb{E}[\langle A_i, x\rangle^2]  = \| x \|_2^2$. Let's assume (we'll relax this later) that $\mathbb{E}[\langle A_i, x\rangle^2] = \| x \|_2^2\leq 1$.
Then, 
$$
\Pr\left(|\langle A_i, x\rangle^2-\|x\|_2^2|\ge t\right) \le 2\exp\left(-\frac{1- \ln 2}{2}\min\left(\frac{t^2}{c^2},\frac{t}{c}\right)\right)
$$
>[!FAQ]- Why?
> Let $X$ be a subgaussian random variable such that $\mathbb{E}[e^{\lambda X}] \;\le\; \exp\!\left(\tfrac{1}{2} c \lambda^2\right)$ where $\mathbb{E}X=0$, $\mathbb{E}[X^2]\le c$, and $\lambda\in\mathbb{R}$. We start from the general Chernoff bound:
>$$
>P(X^2-E[X^2] \geq t) \leq \inf_{\lambda>0}e^{-\lambda t}E[e^{\lambda(X^2-\mathbb{E}[X^2])}]
>$$
>This requires to upper bound the moment generating function of $X^2$:
>  $$ 
> \mathbb{E}\big[e^{\lambda X^2}\big]\le \frac{1}{\sqrt{\,1-2\lambda c}}
> $$
>  for $0\le \lambda <\frac{1}{2x}$. 
>> [!FAQ]- Why?
>> Use the [power-series of the exponential](https://en.wikipedia.org/wiki/Exponential_function):
>> $$
>> \mathbb{E}[e^{\lambda X^2}] = \mathbb{E}\left[\sum_{k\ge0}\frac{\lambda^k X^{2k}}{k!}\right] = \sum_{k\ge0}\frac{\lambda^k\mathbb{E}[X^{2k}]}{k!} \tag{1}
>>$$
>>By the [definition of MGF](https://en.wikipedia.org/wiki/Moment-generating_function), we can obtain the moment $\mathbb{E}[X^{2k}]$ by differentiating the MGF of $X$ $2k$ times with respect to $\lambda$ and set $\lambda=0$:
>>$$
>>\mathbb{E}[X^{2k}]=\left.\frac{d^{2k}}{d\lambda^{2k}}\mathbb{E}[e^{\lambda X}]\right|_{\lambda=0} \le \left.\frac{d^{2k}}{d\lambda^{2k}}e^{\lambda^2 \sigma/2}\right|_{\lambda=0} = (2k)!\,\frac{c^k}{2^k k!}
>>$$
>>where we used that $X$ is subgaussian and $\exp\!\left(\tfrac{1}{2} c \lambda^2\right) = \sum_{m=0}^\infty \frac{1}{m!} \left(\tfrac{1}{2} c \lambda^2\right)^m$. Plugging $\mathbb{E}[X^{2k}]$ back to (1): 
>>$$
>>\mathbb{E}\big[e^{\lambda X^2}\big] = \sum_{k=0}^\infty\frac{\lambda^k\mathbb{E}[X^{2k}]}{k!} \le \sum_{k=0}^\infty \frac{\lambda^k}{k!}\cdot\frac{(2k)!}{2^k k!}\,c^{\,k} =\sum_{k=0}^{\infty}\binom{2k}{k}\left(\frac{\lambda c}{2}\right)^k = \frac{1}{\sqrt{\,1-2\lambda c\,}}
>>$$
>>since $\frac{(2k)!}{2^k(k!)^2}=\binom{2k}{k}2^{-k}$ and the last equality comes from the [central binomial generating function](https://math.stackexchange.com/questions/1064216/generating-functions-and-central-binomial-coefficient) for $0 \leq \lambda \leq \frac{1}{2c}$.
>
>Since
>$$
>\mathbb{E}\big[e^{\lambda(X^2-E[X^2])}\big] \leq e^{-\lambda c}\,\mathbb{E}[e^{\lambda X^2}] \leq \frac{e^{-\lambda c}}{\sqrt{1-2\lambda c}}
>$$
>for $0 \leq \lambda \leq 1/(2c)$, the tail inequality becomes:
>$$
>\begin{align}
>\Pr\big(X^2-E[X^2]\ge t\big) &\le \inf_{0\le \lambda<1/(2c)} e^{-\lambda t}\,\mathbb{E}\big[e^{\lambda(X^2-c)}\big]\\
> &\le \inf_{0\le \lambda<1/(2c)} \frac{e^{-\lambda(t+c)}}{\sqrt{1-2\lambda c}}\\
> &= \inf_{0\le \lambda<1/(2c)} \exp\left(-\lambda(t+c)-\tfrac12\ln(1-2\lambda c)\right)
>\end{align}
>$$
>This can be optimized in closed form. Differentiate the exponent $f(\lambda) =-\lambda(t+c)-\tfrac12\ln(1-2\lambda c)$ and set to zero: 
>$$
>f’(\lambda)=-(t+c)+\frac{c}{1-2\lambda c}=0
>$$
>which gives $\lambda_{\min}=\frac{t}{2c(t+c)}$ lying in the allowed range. Plugging $\lambda_{\min}$ into $f$ gives the explicit exponent at optimum:
>$$
>f(\lambda_{\min}) = -\frac{t}{2c} + \tfrac12\log\!\Big(1+\frac{t}{c}\Big)
>$$
> Hence, we obtain the closed-form Chernoff bound:
> $$
> \Pr\big(X^2-E[X^2]\ge t\big) \le \exp\!\Big(-\frac{t}{2c}+\frac{1}{2}\ln\!\Big(1+\frac{t}{c}\Big)\Big)
> $$
> After bounding the exponent:
> $$
> \Pr\big(X^2-E[X^2]\ge t\big) \le \exp\!\Big(-\frac{1-\ln 2}{2}\min\!\Big(\frac{t^2}{c^2},\frac{t}{c}\Big)\Big)
> $$
> > [!FAQ]- Why?
>>  Let $z =t/c$. The exponent in the Chernoff bound above equals $\frac{1}{2}\left(-z+\ln(1+z)\right)$.  Let $\Phi(z)=-z+\ln(1+z)$. We will give a quadratic upper bound on $\phi(z) \leq C_1\cdot z^2$ when $0\le z \le 1$, and a linear upper bound $\phi(z) \leq C_2\cdot z$  if $z \geq 1$. This implies that $\phi(z) \leq \min(C_1,C_2)\cdot \min(z^2, z)$ for $z>0$.
>>   1. _Small deviations_ ($0\le z\le 1$): $C_1=\max_{z\in[0,1]} \phi(z)/z^2=\max_{z\in[0,1]} \frac{-z+\ln(1+z)}{z^2}$. Since $\phi(z)/z^2$ is monotonically increasing, it takes the maximum at $z=1$: $C_1=\phi(1)/1=\ln2-1$. 
>>  2. _Large deviations_ ($z\ge 1$):  $C_2=\max_{z\ge 1} \Phi(z)/z=\max_{z\ge 1} \frac{-z+\ln(1+z)}{z}$. $\Phi(z)/z$ is monotonically decreasing and hence takes the maximum at $z=1$: $C_2=\Phi(1)/1=\ln2-1$
>>  
>>  Therefore, we get 
>>  $$
>>  \begin{align}
>>  \Pr\left(X^2-E[X^2]\ge t\right) &\le \exp\left( \frac{1}{2} \left(-z+\ln(1+z)\right)\right) \\ 
>>  &\le \exp\left(-\frac{1 - \ln2}{2}\min\left( z^2,z \right)\right)
>>  \end{align}
>> $$
>
> We get similar result for $-X^2$. The lower and upper tail inequalities can be combined with a union bound:
>  $$
> \Pr\big(|X^2-E[X^2]|\ge t\big) \le 2\exp\!\Big(-\frac{1-\ln 2}{2}\min\!\Big(\frac{t^2}{c^2},\frac{t}{c}\Big)\Big)
> $$

#### Step 3: $\langle A_i, x\rangle^2$ is subexponential $\implies$ $\frac{1}{m}\|A x \|_2^2$ is subexponential

Since  $\frac{1}{m}\|A x \|_2^2 = \frac{1}{m}\sum_{i=1}^m \langle A x \rangle^2$, we need to upper bound the probability
$$
\begin{align}
\Pr\left( \left| \frac{1}{m} \sum_{i=1}^m \langle A_i, x\rangle^2- \mathbb{E}\left[\frac{1}{m}\sum_{i=1}^m \langle A_i, x\rangle^2\right] \right| \ge t\right) &= \Pr\left(\frac{1}{m}\left| \sum_{i=1}^m \left(\langle A_i, x\rangle^2- \|x\|_2^2\right)\right| \ge t\right) \\
&=\Pr\left(\left| \sum_{i=1}^m \left(\langle A_i, x\rangle^2- \|x\|_2^2\right)\right| \ge t\cdot m\right) \\
& \leq 2\exp\left(-\frac{1-\ln 2}{2}\cdot m \cdot \min\left(\frac{t^2}{c^2},\frac{t}{c}\right)\right)
\end{align}
$$
where the centered random variable $Z_i= \langle A_i, x\rangle^2- \|x\|_2^2$ is subexponential as we showed above, and $\mathbb{E}\left[\frac{1}{m}\sum_{i=1}^m \langle A x \rangle^2 \right]= \| x \|_2^2$. 

>[!FAQ]- Why?
>We need to derive another tail inequality for the sum of independent random variables with zero mean.  We start again from the general Chernoff bound:
>$$
>\begin{align}
>P\left(\sum_i Z_i \geq t\right) &\leq \inf_{\lambda>0}e^{-\lambda t}\,\mathbb{E}\left[e^{\lambda \sum_i Z_i}\right] \\ 
>&= \inf_{\lambda>0}e^{-\lambda t}\,\mathbb{E}\left[\prod_i e^{\lambda  Z_i}\right] \\
>& = \inf_{\lambda>0}e^{-\lambda t}\,\prod_i\mathbb{E}\left[ e^{\lambda  Z_i}\right] 
>\end{align}
>$$
>where the last equality is due to independence.
>Since $\mathbb{E}\big[e^{\lambda Z_i}\big]\le \frac{e^{-\lambda c}}{\sqrt{1-2\lambda c}}$ for $0\le \lambda <\frac{1}{2c}$,
>$$
>e^{-\lambda t}\prod_i\mathbb{E}\left[ e^{\lambda  Z_i}\right] \leq \frac{e^{-\lambda(t+m c)}}{(1-2\lambda c)^{m/2}} = \exp\left(-\lambda(t+m c) - \frac{m}{2}\ln(1-2\lambda c)\right)
>$$
>Define the exponent
>$$
>f(\lambda)\;=\; -\lambda(t+m c)-\tfrac{m}{2}\ln(1-2\lambda c).
>$$
>We minimize $f(\lambda)$ over $0\le\lambda<\frac{1}{2c}$. Differentiate:
>$$
>f'(\lambda) = -(t+m c)+\frac{m c}{1-2\lambda c}
>$$
>Set $f’(\lambda)=0$. Solving gives the interior minimizer (valid for $t>0$):
>$$
>1-2\lambda_{\min} c = \frac{m c}{t+m c} \quad\Longrightarrow\quad \lambda_{\min}=\frac{t}{2c(t+m c)}\in\Big(0,\frac{1}{2c}\Big)
>$$
>Plugging $\lambda_\min$ into $f$ yields the closed-form exponent:
>$$
>f(\lambda_{\min}) = -\frac{t}{2c} + \frac{m}{2}\ln\!\Big(1+\frac{t}{m c}\Big)
>$$
>and the final Chernoff bound:
>$$
>\Pr\left(\sum_i Z_i\ge t\right) \le \exp\!\Big(-\frac{t}{2c}+\frac{m}{2}\ln\!\Big(1+\frac{t}{m c}\Big)\Big)
>$$
>Put $z=\dfrac{t}{m c}$. Then the exponent:
>$$
>-\frac{t}{2c}+\frac{m}{2}\ln(1+z) = -\frac{m c z}{2c} + \frac{m}{2}\ln(1+z) = \frac{m}{2}\Big(-z+\ln(1+z)\Big)
>$$
>We can bound $\phi(z)=-z+\ln(1+z)$ just like above. Hence we get:
>$$
>\Pr\left(\sum_i Z_i \ge t\right)\le \exp\left(-\frac{1-\ln 2}{2}\cdot m \cdot \min\!\Big(\frac{t^2}{m^2c^2},\frac{t}{mc}\Big)\right)
>$$

Therefore, in the general case when $\| x \|_2^2 > 1$, the above tail bounds extend to normalized vectors $x/\|x\|_2$:
$$
\boxed{\qquad
\Pr\left(\left| \|\hat{A} x \|_2^2- \|x\|_2^2\right| \ge t \cdot \|x\|_2^2 \right) \leq 2\cdot \exp\left(-\frac{1-\ln 2}{2}\cdot m \cdot \frac{t^2}{c^2} \right)\qquad
\tag{*}
}
$$
for any $0 < t < 1$, where $\hat{A} = \frac{1}{\sqrt{m}}A$. Informally, this inequality says that if the entries of $A$ are independent, mean-zero subgaussian random variables, then $\hat{A}$ preserves the norm of any vector fixed vector $x$ up to a relative error $t$, with high probability as soon as $m$ is large enough.

### Concentration of all submatrices

To simplify exposition, suppose that $x$ is binary with support $s$. Then,
$$
\boxed{\qquad
m \geq \frac{4}{1-\ln2} \cdot \frac{c^2}{\delta^2} \cdot s \cdot  \ln\frac{en}{s}
\qquad} \tag{**}
$$
where $\delta$ is the restricted isometry constant of matrix $A$. This confirms that the required number of measurements scales as $m=\mathcal{O}(s\ln(en/s))$.

>[!FAQ]- Why?
>Since $x$ is binary, its Euclidean norm satisfies $\|x\|_2^2= s$. The total number of distinct supports - and hence distinct binary vectors $x$ -  is ${n \choose s}$.
>For a fixed vector $x$, the probability that the concentration fails is given by 
>$$
>\Pr\left(\left| \|\hat{A} x \|_2^2- s\right| \ge t s \right) \leq 2\cdot \exp\left(-\frac{1-\ln 2}{2}\cdot m \cdot \frac{t^2}{c^2} \right)
>$$
>Applying the union bound over all possible supports of size s, the probability that the concentration fails for **any** s-sparse binary vector is bounded by
>$$
>\Pr\left( \exists\,x:  \left| \|\hat{A} x \|_2^2- s\right| \ge t s  \right) \leq 2\cdot {n \choose s} \cdot \exp\left(-\frac{1-\ln 2}{2}\cdot m \cdot \frac{t^2}{c^2} \right)
>$$
>Using the standard inequality ${n \choose s} \leq \left( \frac{en}{s}\right)^s$, we obtain
>$$
>\Pr\left( \exists\,x:  \left| \|\hat{A} x \|_2^2- s\right| \ge t s  \right) \leq 2\cdot \exp\left(s\ln\frac{en}{s}-\frac{1-\ln 2}{2}\cdot m \cdot \frac{t^2}{c^2} \right)
>$$
>If we require this failure probability to be at most $\eta$, we set 
>$$
>2\cdot \exp\left(s\ln\frac{en}{s}-\frac{1-\ln 2}{2}\cdot m \cdot \frac{t^2}{c^2} \right) \leq \eta
>$$
>Taking the logarithms and rearranging yields
>$$
>m \geq \frac{2}{1-\ln2} \cdot \frac{c^2}{t^2} \cdot\left(s \ln\frac{en}{s} + \ln \frac{2}{\eta}  \right)
>$$
>Here, $t$ corresponds to the restricted isometry constant $\delta$ of the matrix $A$ (with probability at least $1 - \eta$). Finally, if we set the failure probability to $\eta = 2\exp\left(-\frac{1-\ln 2}{4}\cdot m\cdot \frac{t^2}{c^2}\right)$, we obtain Formula ( ** ).

## Beyond symmetric Bernoulli matrices

So far we assumed that every entry of $A$ is $+1$ or $-1$ with equal probability. However, the analysis above relied on only two properties of the distribution of $A_{i,j}$, so it carries over to other distributions, up to constant factors:
- *Property 1 (isotropy)*: $E|\langle A_i, x\rangle|^2 = \| x \|_2^2$, that is, every row $A_i$ follows an **isotropic** distribution: its covariance matrix is the identity. If the entries of $A$ are independent, have zero mean and unit variance ($E[A_{i,j}^2]=1$), this holds by the same calculation as in the [[Blog/Data Protection/Extracting Sensitive Information from Aggregated Data - Part 2#Random query matrix\|#Random query matrix]] section.
- *Property 2 (subgaussianity)*: $\mathbb{E}[e^{\lambda A_{i,j}}] \le \exp\!\left(c \lambda^2/2\right)$ for every $i,j$, every $\lambda \in \mathbb{R}$, and a constant $c$. This guarantees that $\mathcal{O}(s\ln(n/s))$ queries are enough for reconstruction, and $c$ only shows up in the constant factor of Formula ( ** ). Based on [Hoeffding's lemma](https://en.wikipedia.org/wiki/Hoeffding%27s_lemma), every bounded random variable is subgaussian, though the constant given by Hoeffding is often not the best one.

When $c = 1$, that is, the proxy variance equals the true variance $E[A_{i,j}^2] = 1$, the random variable is called **strictly subgaussian**. The symmetric $\pm 1$ entries are strictly subgaussian ($\cosh(\lambda) \leq e^{\lambda^2/2}$), and Steps 1-3 apply to them word for word.

>[!FAQ]- What if $c > 1$?
>In Step 2, we used $E[X^2] = c$ to center the moment generating function of $X^2$. If the proxy variance $c$ is strictly larger than the true variance $E[X^2]$, this shortcut is not valid, and one has to use the general [Bernstein inequality](https://en.wikipedia.org/wiki/Bernstein_inequalities_(probability_theory)) for sums of independent subexponential random variables instead (see Theorem 2.8.1 and Section 5.3 in R. Vershynin, [High-Dimensional Probability](https://www.math.uci.edu/~rvershyn/papers/HDP-book/HDP-book.pdf)). The conclusion is the same, $m = \mathcal{O}(s\ln(en/s))$, but the constant in Formula ( ** ) becomes $C \cdot c^2$ for some absolute constant $C$. In other words, the number of queries grows (at most) with the square of the proxy variance.

We demonstrate the above conditions on two practical examples:
1. $A_{i,j}$ follows a **sparse Rademacher** distribution: $\Pr[A_{i,j} = +1] = \Pr[A_{i,j} = -1] = p$, and $\Pr[A_{i,j} = 0] = 1 - 2p$, where $p=0.5$ corresponds to the fully dense case analyzed above. If $p<0.5$, then zeros appear, but $+1$ and $-1$ still occur with equal probability, hence $A_{i,j}$ is symmetric.
2. $A_{i,j}$ follows a **non-symmetric Bernoulli** distribution: $\Pr[A_{i,j}=1]=p$ and $\Pr[A_{i,j}=0]=1-p$. This is the most natural query model: every query includes each record independently with probability $p$, so the numbers of ones and zeros differ when $p \neq 1/2$.
### Sparse Rademacher

If each entry of $A$ follows a sparse Rademacher distribution, then

- _Property 1_ holds if each entry of $A$ is scaled by $1/\sqrt{2p}$. Indeed, $E[A_{i,j}^2] = \left(\frac{1}{\sqrt{2p}}\right)^2 \cdot p + \left(-\frac{1}{\sqrt{2p}}\right)^2 \cdot p + 0^2\cdot (1-2p)=1$, and hence $E|\langle A_i, x\rangle|^2 = \sum_{\ell} x_\ell^2$ if all entries of $A$ are chosen independently.
- _Property 2_ holds with $c = \frac{1}{2p}$:
$$

\mathbb{E}[e^{\lambda A_{i,j}}] \leq \exp\left(\frac{\lambda^2}{4p}\right) = \exp\left(\frac{1}{2p}\cdot\frac{\lambda^2}{2}\right)

$$
>[!FAQ]- Why?
>This follows from the dense case. With $u = \lambda/\sqrt{2p}$,
>$$
>\mathbb{E}[e^{\lambda A_{i,j}}] = (1-2p) + p\,e^{u} + p\,e^{-u} = (1-2p) + 2p\cosh(u) \leq \cosh(u) \leq e^{u^2/2} = \exp\left(\frac{\lambda^2}{4p}\right)
>$$
>where the first inequality holds because $\cosh(u) \geq 1$, and the second one is the bound $\cosh(u) \le e^{u^2/2}$ we used for the dense $\pm 1$ entries. [Hoeffding's lemma](https://en.wikipedia.org/wiki/Hoeffding%27s_lemma) gives the same constant, since $A_{i,j}$ lies in an interval of length $2/\sqrt{2p}$.
  
This bound is not tight, because the first inequality treats the zeros as if they were $\pm 1/\sqrt{2p}$, ignoring that most of the probability mass sits at 0. For $p=1/2$, it gives $c=1$ as in the dense case, but $c$ grows as $1/(2p)$ when the matrix becomes sparser. A [finer analysis](https://doi.org/10.1016/S0022-0000(03)00025-4), shows that the entries are **strictly subgaussian** ($c=1$) for every $p \geq 1/6$, and that $c \leq 1/(6p)$ for $p < 1/6$. The argument [carries over](https://arxiv.org/abs/1901.09188) to any larger $p$. Therefore, the entries are strictly subgaussian exactly as long as at most two thirds of them are zero.

This means that sparse Rademacher queries with $p = 1/6$, that is, entries $\sqrt{3}\cdot\{+1, 0, -1\}$ with probabilities $1/6, 2/3, 1/6$, [give exactly the same guarantee](https://doi.org/10.1016/S0022-0000(03)00025-4) as the dense $\pm1$ matrix, even though two thirds of the entries are zero. For sparser matrices ($p<1/6$), the number of queries in Formula ( ** ) grows by at most a factor $c^2 = 1/(36p^2)$ (or $1/(4p^2)$ with the simpler constant $c = 1/(2p)$).

How can we realize a sparse Rademacher row with binary queries? Let $P_i$ and $N_i$ be the binary indicator vectors of the $+1$ and $-1$ positions in row $i$. Then $\langle A_i, x\rangle = \frac{1}{\sqrt{2p}}\left(\langle P_i,x\rangle - \langle N_i,x\rangle\right)$, which requires two binary queries. Each of them covers only about $pn$ records, which may be too few for the database manager. However, the complement trick of the Hadamard construction helps again. Since $\langle P_i, x\rangle = \sum_j x_j - \langle \bar{P}_i, x\rangle$, where $\bar{P}_i = \mathbf{1} - P_i$ is the complement query (and similarly for $N_i$),
$$

\langle P_i,x\rangle - \langle N_i,x\rangle = \langle \bar{N}_i,x\rangle - \langle \bar{P}_i,x\rangle

$$
where both $\bar{P}_i$ and $\bar{N}_i$ cover about $(1-p)n$ records. For $p=1/6$, every query covers about $5/6$ of the database. Hence, a sparse Rademacher attack needs twice as many binary queries as the dense one, but each query looks _more_ aggregated.

### Non-symmetric Bernoulli

Let $B_{i,j}=1$ with probability $p$ and $B_{i,j}=0$ otherwise. If each entry of the query matrix $B$ follows this non-symmetric Bernoulli distribution, then
- _Property 1_ does not hold. $B$ cannot be even approximately orthogonal, since the inner product of two non-negative vectors is never negative. Indeed,
  $$
  E|\langle B_i, x\rangle|^2 = \sum_{j} x_j^2 E[B_{i,j}^2] + \sum_{j\neq \ell} x_j x_\ell E[B_{i,j}]E[B_{i,\ell}] = p(1-p)\|x\|_2^2 + p^2\Big(\sum_j x_j\Big)^2
  $$
  where the second term, which comes from the non-zero mean $E[B_{i,j}] = p$, breaks isotropy. To fix this, we **center and normalize** $B$:
  $$
  A_{i,j} = \frac{B_{i,j} -  E[B_{i,j}]}{\sigma} = \frac{B_{i,j} - p}{\sqrt{p(1-p)}}
  $$
  where $\sigma^2 = E\left[(B_{i,j}-p)^2\right] = p(1-p)$ is the variance of $B_{i,j}$. Hence, $A_{i,j} = \frac{1-p}{\sqrt{p(1-p)}}=\sqrt{\frac{1-p}{p}}$ with probability $p$, and $A_{i,j}=\frac{-p}{\sqrt{p(1-p)}} = -\sqrt{\frac{p}{1-p}}$ with probability $1-p$. $A_{i,j}$ has zero mean and unit variance, so $E|\langle A_i, x\rangle|^2 = \| x \|_2^2$ holds.
- _Property 2_ holds because $A_{i,j}$ is bounded. In particular, it follows from [Hoeffding's lemma](https://en.wikipedia.org/wiki/Hoeffding%27s_lemma) that
  $$
  \mathbb{E}\left[e^{\lambda A_{i,j}}\right]\leq \exp\left(\frac{\lambda^2}{8\sigma^2}\right)=\exp\left(\frac{1}{4p(1-p)}\cdot\frac{\lambda^2}{2}\right)
  $$
  that is, $c = \frac{1}{4p(1-p)}$.

>[!FAQ]- Why?
>Let $X=B_{i,j} - E[B_{i,j}]$ be a mean-zero and bounded random variable: $X\in[-p,1-p]$, so its range has length 1. Hoeffding's lemma states that any mean-zero random variable with range length $L$ satisfies
>$$
>\mathbb{E}[e^{\lambda X}] \le \exp\left(\frac{\lambda^2 L^2}{8}\right)
>$$
>Since $L=1$ and $A_{i,j} = X/\sigma$, we get the bound by the substitution $\lambda\mapsto \lambda/\sigma$: $\mathbb{E}[e^{\lambda A_{i,j}}] = \mathbb{E}[e^{(\lambda/\sigma)\cdot X}] \leq \exp\left(\frac{\lambda^2}{8\sigma^2}\right)$.

Unlike the sparse Rademacher case, $c>1$ for every $p \neq 1/2$. This is not an artifact of Hoeffding's lemma: a centered Bernoulli variable is never strictly subgaussian unless $p=1/2$, because it is not symmetric. Its third moment $E[A_{i,j}^3] = \frac{1-2p}{\sqrt{p(1-p)}}$ is non-zero, which adds a $\lambda^3$ term to its MGF that $e^{\lambda^2/2}$ cannot dominate for small $\lambda$ (with the appropriate sign). Hoeffding's constant is not optimal though; the [best possible constant is known](https://arxiv.org/abs/1210.3248) to be $c^* = \frac{1-2p}{2p(1-p)\ln\frac{1-p}{p}}$:
$$ 
\begin{array}{l|cccc} p \text{ (or } 1-p) & 0.5 & 0.75 & 0.9 & 0.99 \\ \hline c \text{ (Hoeffding)} & 1 & 1.33 & 2.78 & 25.3 \\ c^* \text{ (optimal)} & 1 & 1.21 & 2.02 & 10.8 \end{array}
$$
Both constants grow only when the queries become very small ($p\to 0$) or cover almost the whole database ($p \to 1$).

How can we realize $A$ with strictly binary queries? Records with $B_{i,j}=1$ get weight $\frac{1-p}{\sigma}$ and records with $B_{i,j}=0$ get weight $-\frac{p}{\sigma}$, so
$$
\langle A_i, x \rangle = \frac{1-p}{\sigma}\langle B_{i}, x \rangle - \frac{p}{\sigma} \left(\sum_j x_j - \langle B_{i}, x \rangle \right)= \frac{1}{\sigma} \left( \langle B_{i}, x \rangle - p\sum_j x_j\right)
$$
This means that $A$ can be realized with the $m$ binary queries $B$ plus a single query returning the total count $\sum_{j} x_j$, which covers all $n$ records and is therefore always allowed. If $p < 1/2$, the adversary can also ask the complement query $\bar{B}_i$ instead of $B_i$, since $\langle B_{i}, x \rangle = \sum_j x_j - \langle \bar{B}_{i}, x \rangle$.

**What does this mean for $k$-anonymity?** Suppose the database manager answers only queries covering at least $k = \kappa n$ records, for any $\kappa < 1$. The adversary picks $p$ slightly above $\kappa$ (or asks the complements of queries with $p$ slightly below $1-\kappa$), so that every query passes the threshold. The price is the factor $c^2 \approx 1/(16\kappa^2(1-\kappa)^2)$ in Formula ( ** ) (or less with $c^*$), which does not depend on $n$. Raising $k$ does not stop the attack; it only makes it a constant factor more expensive. In a quick simulation with $n=5000$ records and $s=10$ non-zero entries, OMP (see below) reconstructed $x$ in all trials with 140 queries for $p=0.5$, 200 queries for $p=0.9$, and 800 queries for $p=0.99$, where every query covered about 4950 of the 5000 records.

>[!Conclusion]
>The numbers of ones and zeros in the queries do not need to be balanced. A random 0/1 query matrix, where every query covers a $p$ fraction of the records, plus a single query for the total count, still reconstructs an $s$-sparse dataset $x$ from $\mathcal{O}\!\big(s\ln(en/s)\big)$ queries. The imbalance only costs a constant factor, which grows as the queries shrink ($p\to 0$) or approach the full database ($p\to 1$). Sparse Rademacher rows with $p \geq 1/6$ cost nothing extra asymptotically, and need two binary queries per row.

# Reconstruction

We still need an efficient algorithm which computes $x$ from $b = Ax$, or from noisy answers $b = Ax + e$. [Least squares methods](https://en.wikipedia.org/wiki/Least_squares) from the [[Blog/Data Protection/Extracting Sensitive Information from Aggregated Data  - Part 1#Solving the equations\|previous post]] are not suitable: if $m<n$, they return the solution with minimum $\ell_2$-norm, which is dense and usually far from the sparse $x$. Good decoders exploit the sparsity of $x$, and they come in two forms.

**Convex relaxation.** Finding the sparsest solution ($\ell_0$-minimization) is NP-hard, so we replace the number of non-zero entries with the $\ell_1$-norm, which is convex but still favors sparse solutions:
$$
\hat{x} = \arg\min_z \|z\|_1 \quad \text{subject to} \quad \|Az-b\|_2 \le \varepsilon
$$
With $\varepsilon = 0$, this is called _Basis Pursuit_ and can be solved as a linear program. With $\varepsilon > 0$ (noisy answers), it is called _Basis Pursuit Denoising_, whose Lagrangian form $\min_z \frac{1}{2}\|Az-b\|_2^2 + \lambda\|z\|_1$ is the well-known [LASSO](https://en.wikipedia.org/wiki/Lasso_(statistics)). Why does $\ell_1$ promote sparsity? The $\ell_1$-ball is a "spiky" polytope whose corners lie on the coordinate axes, so when we inflate it until it touches the set of solutions $\{z : Az = b\}$, it typically touches it at a corner, which is a sparse vector. The [key guarantee](https://doi.org/10.1016/j.crma.2008.03.014) is that if the restricted isometry constant of $A$ for $2s$-sparse vectors satisfies $\delta_{2s} < \sqrt{2}-1$, then
$$
\|\hat{x} - x\|_2 \le C_0\, \frac{\|x - x_s\|_1}{\sqrt{s}} + C_1\, \varepsilon
$$
where $x_s$ is the best $s$-sparse approximation of $x$, and $C_0, C_1$ are small constants. Hence, recovery is exact if $x$ is $s$-sparse and the answers are exact, and the error grows only linearly with the noise $\varepsilon$ otherwise. This is exactly the stability we were looking for. Note that $\ell_1$-minimization does not need to know $s$ at all. If $x$ is binary, one can also add the constraints $0 \leq z_j \leq 1$ and round the result.

**Greedy methods.** [Orthogonal Matching Pursuit (OMP)](https://en.wikipedia.org/wiki/Matching_pursuit) builds the support of $x$ one index at a time. We apply OMP due to its simplicity and scalability; it works well even with many queries (i.e., larger $A$). The basic idea is quite simple:
1. Start with the residual $r = b$ and an empty support $S = \emptyset$.
2. Find the column of $A$ which is the most correlated with the residual, $j = \arg\max_j |\langle A_{:,j}, r\rangle|$, and add $j$ to $S$.
3. Solve least squares restricted to the columns in $S$, $z_S = \arg\min_z \|A_S z - b\|_2$, and update the residual $r = b - A_S z_S$.
4. Repeat steps 2-3 until $|S| = s$ or $\|r\|_2$ is small enough.

Why does it work? Since $A$ (after scaling with $1/\sqrt{m}$) is quasi-orthogonal, $A^\top A x \approx x$. Therefore, $A^\top r$ is a noisy copy of the not yet recovered part of $x$, and its largest entry points to a non-zero record. Each iteration costs a single matrix-vector product and a small least squares problem, so the total cost is $\mathcal{O}(smn)$. It is [known](https://doi.org/10.1109/TIT.2007.909108) that that OMP recovers any fixed $s$-sparse vector from $\mathcal{O}(s\ln n)$ random subgaussian queries with high probability. Variants such as [CoSaMP](https://arxiv.org/abs/0803.2392) and [Iterative Hard Thresholding](https://arxiv.org/abs/0805.0510) select several indices per iteration and come with the same RIP-based stability guarantees as $\ell_1$-minimization. In practice, OMP is a good default when the answers are exact and $s$ (or an upper bound on it) is known, while LASSO is preferable for noisy answers.

## What if the queries cannot be random?

In SQL, the adversary selects records through predicates on their attributes, so an arbitrary random subset may not be expressible directly. The simplest trick is to derive randomness from the data itself: if the database allows arithmetic on a high-entropy attribute (an ID, a birth date, a timestamp), a predicate such as `WHERE MOD(a*id + b, P) < P/2` selects a pseudo-random half of the records. [Cohen and Nissim](https://journalprivacyconfidentiality.org/index.php/jpc/article/view/711) used this idea to reconstruct records from Diffix, a commercial system for anonymized queries. Similarly, bit shifts such as `WHERE (id >> k) & 1 = 1` select records by the binary digits of their ID, which is exactly what the Hadamard and BCH constructions need. Either way, the adversary needs predicates that can tell records apart: records that agree on every attribute usable in a predicate can never be separated, only their sum can be recovered.

If the queries are fixed and have no useful structure, for example because only pre-defined statistical tables are published, the most general solution is to write all answers as constraints of an integer program (or a SAT instance) and hand it to a solver. There is no recovery guarantee, but it [works in practice](https://dl.acm.org/doi/10.1145/3287287): the US Census Bureau reconstructed a large fraction of the individual records of the 2010 census from its own published tables, which led to the adoption of Differential Privacy for the 2020 census.
# Summary

The table below summarizes the reconstruction techniques discussed in this post. $n$ is the number of records, $s$ is the number of non-zero records, and "records per query" shows how aggregated each query looks to a $k$-anonymity type defense.

| Method (decoder)                             | Queries needed                                         | Query type                             | Records per query        | Sparsity assumption                    | Noisy answers                                 |
| -------------------------------------------- | ------------------------------------------------------ | -------------------------------------- | ------------------------ | -------------------------------------- | --------------------------------------------- |
| Identity (baseline)                          | $n$                                                    | 0/1                                    | 1                        | None                                   | Robust                                        |
| Hadamard (fast Walsh-Hadamard transform)     | $n+1$                                                  | 0/1, complement + total count          | $n/2$                    | None                                   | Robust unless the noise is $\Omega(\sqrt{n})$ |
| Vandermonde / Fourier (Prony's method)       | $2s$                                                   | Weighted SUM (real or complex weights) | $n$                      | $s$-sparse, known $s$                  | Very unstable                                 |
| BCH syndrome (Berlekamp-Massey)              | $2s\lceil\log_2(n+1)\rceil$                            | 0/1, answers taken mod 2               | $(n+1)/2$                | Binary $x$, upper bound on $s$         | Fails (a single $\pm1$ error flips a parity)  |
| Random $\pm1$ ($\ell_1$-minimization or OMP) | $\mathcal{O}(s\ln(en/s))$                              | 0/1, complement + total count          | $\approx n/2$            | Upper bound on $s$ (none for $\ell_1$) | Stable: error $\propto$ noise                 |
| Sparse Rademacher, $p \geq 1/6$              | $\mathcal{O}(s\ln(en/s))$, same constant               | Two 0/1 complement queries per row     | $\approx (1-p)n$         | Upper bound on $s$                     | Stable                                        |
| Random 0/1 with $\Pr[1]=p$                   | $\mathcal{O}(c^2 s\ln(en/s))$, $c = \frac{1}{4p(1-p)}$ | 0/1 + total count                      | $\approx pn$ or $(1-p)n$ | Upper bound on $s$                     | Stable                                        |

Main observations:
 1. **Exact algebraic constructions (Vandermonde, BCH) need the fewest queries, but they break down as soon as the answers are perturbed.**
 2. **Random query matrices need only a logarithmic factor more queries, and they tolerate noise, do not require the exact sparsity, and do not care whether the queries are balanced.**
 3. **In every row of the table, individual queries can be made to cover at least half of the database.**
# Conclusion

_Aggregation alone does not protect individual records._ A $k$-anonymity type threshold on the query size, even $k = n/2$ or larger, does not prevent an adversary from reconstructing every record with $n$ queries. If the sensitive attribute is sparse, like the single patient with a rare genetic disease among 100,000 patients, a logarithmic number of queries is enough. The most practical attack is also the simplest one: ask random queries, each covering a random subset of the records, and decode the answers with $\ell_1$-minimization or OMP. This attack does not need a carefully designed query structure, does not need to know the exact number of non-zero records, tolerates noisy answers, and works for any query size threshold $k < n$ at the cost of a constant factor.

The same argument applies beyond databases. If a secure aggregation protocol reveals the sum over a random subset of participants in every round, and the individual contributions stay (approximately) the same across rounds, then the rounds form exactly a random Bernoulli query matrix. In that case, the adversary does not even need to choose the queries; the protocol does it for them.

These techniques come from _compressed sensing_, which was developed to reconstruct sparse signals from few measurements in signal processing. Many naturally occurring signals are sparse in some basis, such as images in the wavelet domain or audio in the frequency domain, and compressed sensing is used, for example, to speed up MRI scans. All reconstruction techniques above are efficient, but they only work for _linear_ queries such as COUNT and SUM. Non-linear aggregates (MIN, MAX, median) lead to combinatorial problems where SAT or integer programming solvers may be required. Moreover, the adversary must be able to target specific records with the query predicates, which requires fairly accurate knowledge of their non-sensitive attributes.

Interestingly, the key ingredients of this attack have been known for more than two decades, only from the other side of the table. In 2003, [Achlioptas](https://dl.acm.org/doi/10.1016/S0022-0000%2803%2900025-4) showed that random $\pm 1$ matrices, and even sparse matrices with entries $\sqrt{3}\cdot\{+1, 0, -1\}$ with probabilities $1/6, 2/3, 1/6$, preserve distances as well as Gaussian random matrices in the [Johnson-Lindenstrauss lemma](https://en.wikipedia.org/wiki/Johnson%E2%80%93Lindenstrauss_lemma). He called them _database-friendly_ random projections, because computing them needs no multiplications: each projected coordinate is the sum of some attributes minus the sum of others, which any database can compute with standard SQL aggregates.  Since a random matrix satisfying the concentration inequality behind the Johnson-Lindenstrauss lemma also [satisfies](https://link.springer.com/article/10.1007/s00365-007-9003-x)the restricted isometry property, these matrices are also good reconstruction queries. The property which made these projections friendly to database owners, namely that they can be computed with plain SUM and COUNT queries, is exactly what makes them friendly to the adversary.

## What does work as a defense?

 Every accurate answer leaks some information, and the leakage accumulates across queries. [Dinur and Nissim](https://doi.org/10.1145/773153.773173) showed that if every answer is accurate up to an error of $o(\sqrt{n})$, then $\mathcal{O}(n\log^2 n)$ random queries are enough to reconstruct almost all records in polynomial time. This is known as the _Fundamental Law of Information Recovery_: overly accurate answers to too many questions destroy privacy. Their attack uses exactly the random 0/1 queries of this post: every query includes each record independently with probability $1/2$. The decoder is different, though. It does not assume that $x$ is sparse, so neither $\ell_1$-minimization of $x$ nor OMP applies. Instead, it solves a linear program to find any fractional solution which is then rounded. Since sparsity is not exploited, it needs more than $n$ queries, as opposed to the $\mathcal{O}(s\ln(n/s))$ queries for sparse data. The extra logarithmic factors pay for the noise, since exact answers would require only $n$ queries, as in the Hadamard construction. The exact algebraic constructions (Vandermonde, Prony, BCH) play no role here, since they fail under any noise. The principled defense is therefore to add noise calibrated to the total number of queries and to bound the overall leakage, which is what [Differential Privacy](https://en.wikipedia.org/wiki/Differential_privacy) does. 
