# CS 740: Mathematical Methods for Visual Computing
## Midterm Theory Practice Problems

**Instructions:** These problems are designed for a written theory exam. Try to solve them on paper, showing all your mathematical steps and derivations clearly. Detailed solutions are provided at the end of the document.

---

### Part 1: Practice Questions

**Question 1: SVD and the Frobenius Norm**
Let $A$ be an $m \times n$ matrix with Singular Value Decomposition (SVD) $A = U \Sigma V^T$. Using the properties of the trace operator and orthogonal matrices, mathematically prove that the squared Frobenius norm of $A$ is equal to the sum of the squares of its singular values. 
*(Hint: Start with $||A||_F^2 = trace(A^T A)$)*

**Question 2: Efficient Principal Components Analysis (PCA)**
In applications like face recognition, you are often given a data matrix $X$ of size $D \times N$, where $D$ (number of pixels) is much larger than $N$ (number of images). The data covariance matrix is $C = \frac{1}{N} XX^T$, which is of size $D \times D$. Computing the eigendecomposition of this $D \times D$ matrix is computationally intractable. 
Derive an efficient mathematical method to find the non-zero eigenvalues and the corresponding eigenvectors of $C$ by utilizing an $N \times N$ matrix instead. 

**Question 3: Structure from Motion (SfM) and the Rank Theorem**
Under the orthographic camera model in Structure from Motion, the measurement matrix $W$ of size $2F \times P$ contains the 2D tracked coordinates of $P$ feature points across $F$ frames. Assume the coordinates in each frame have been centered (the centroid is subtracted).
1. Prove that in a noise-free scenario, the theoretical maximum rank of $W$ is 3.
2. In reality, $W$ contains tracking noise and is full rank. Explain mathematically how the Eckart-Young theorem and SVD are applied to factorize $W$ into a Motion matrix $M$ and a Structure matrix $S$.

**Question 4: Linear Least Squares and Matrix Calculus**
Consider an overdetermined system of linear equations $Ax = b$, where $A$ is an $m \times n$ matrix ($m > n$) with full column rank. The linear least squares objective function is defined as $J(x) = ||Ax - b||_2^2$.
Using matrix calculus, expand this objective function and derive the normal equations used to find the optimal vector $\hat{x}$.

**Question 5: Procrustes Problem and Translation**
Recall the Similarity Procrustes problem where we seek to minimize $f(\alpha, R, t) = ||P_1 - \alpha R P_2 - t1^T||_F^2$. Here, $P_1$ and $P_2$ are $3 \times N$ point matrices, and $1$ is an $N \times 1$ vector of ones. 
Assume that as a preprocessing step, both $P_1$ and $P_2$ have already been perfectly centered at the origin, meaning their centroids are zero ($P_1 1 = 0$ and $P_2 1 = 0$). 
Take the derivative of $f$ with respect to the translation vector $t$ and formally prove that the optimal translation $\hat{t}$ in this centered case is exactly the zero vector.

**Question 6: SVD and Nullspace**
Let $A$ be an $m \times n$ matrix of rank $r < n$, with SVD $A = U \Sigma V^T$. Prove that the last $n - r$ columns of the right singular matrix $V$ form an orthonormal basis for the nullspace of $A$.

---
\pagebreak

### Part 2: Detailed Solutions

**Solution 1: SVD and the Frobenius Norm**
We know that the squared Frobenius norm can be written using the trace operator:
$$||A||_F^2 = trace(A^T A)$$
Substitute the SVD of $A$ ($A = U \Sigma V^T$) into the equation:
$$A^T A = (U \Sigma V^T)^T (U \Sigma V^T) = V \Sigma^T U^T U \Sigma V^T$$
Since $U$ is an orthogonal matrix, $U^T U = I$. Also, $\Sigma$ is a diagonal matrix, so $\Sigma^T = \Sigma$.
$$A^T A = V \Sigma I \Sigma V^T = V \Sigma^2 V^T$$
Now substitute this back into the trace:
$$||A||_F^2 = trace(V \Sigma^2 V^T)$$
Using the cyclic property of the trace ($trace(ABC) = trace(CAB)$):
$$||A||_F^2 = trace(\Sigma^2 V^T V)$$
Since $V$ is also orthogonal, $V^T V = I$:
$$||A||_F^2 = trace(\Sigma^2)$$
$\Sigma^2$ is a diagonal matrix whose diagonal entries are $\sigma_i^2$. The trace is the sum of the diagonal elements:
$$||A||_F^2 = \sum_{i=1}^{\min(m,n)} \sigma_i^2$$

**Solution 2: Efficient Principal Components Analysis (PCA)**
The standard covariance matrix is $C = \frac{1}{N} XX^T$. We want to solve the eigenvalue problem:
$$(\frac{1}{N} XX^T) u = \lambda u$$
Since $D >> N$, finding $u \in \mathbb{R}^D$ directly is too slow. Instead, consider the $N \times N$ matrix $\frac{1}{N} X^T X$. Let $v$ be an eigenvector of this smaller matrix with eigenvalue $\lambda$:
$$(\frac{1}{N} X^T X) v = \lambda v$$
Pre-multiply both sides of this equation by $X$:
$$X (\frac{1}{N} X^T X) v = X (\lambda v)$$
$$\frac{1}{N} X X^T (X v) = \lambda (X v)$$
Let $u = Xv$. Substituting this into the equation yields:
$$(\frac{1}{N} XX^T) u = \lambda u$$
This proves that if $v$ is an eigenvector of the small $N \times N$ matrix $\frac{1}{N} X^T X$, then $Xv$ is an eigenvector of the large $D \times D$ covariance matrix $C$, with the same eigenvalue $\lambda$. To finish the process, we must normalize the resulting vector $u$ to have unit length: $\hat{u} = \frac{Xv}{||Xv||_2}$.

**Solution 3: Structure from Motion (SfM) and the Rank Theorem**
1. Under orthographic projection, the centered coordinate trajectories can be modeled as the matrix product of a camera motion matrix $M$ (size $2F \times 3$) and a 3D structure matrix $S$ (size $3 \times P$):
$$W = M S$$
A standard property of linear algebra states that the rank of a matrix product is bounded by the minimum rank of its factors:
$$rank(W) \le \min(rank(M), rank(S))$$
Since the inner dimension of the factorization is 3, $rank(M) \le 3$ and $rank(S) \le 3$. Therefore, $rank(W) \le 3$.
2. Because of noise, $W$ will have a rank greater than 3. According to the Eckart-Young theorem, the best rank-3 approximation of $W$ (in the least-squares/Frobenius norm sense) is obtained via its SVD.
We compute $W = U \Sigma V^T$.
We retain only the top 3 singular values and their corresponding vectors to form $\hat{W} = U_3 \Sigma_3 V_3^T$.
We can then factorize this approximated matrix into $M$ and $S$. One valid factorization is:
$$M = U_3 \Sigma_3^{1/2} \quad \text{and} \quad S = \Sigma_3^{1/2} V_3^T$$

**Solution 4: Linear Least Squares and Matrix Calculus**
The objective function is $J(x) = (Ax - b)^T (Ax - b)$. Expand this using the distributive property:
$$J(x) = (x^T A^T - b^T)(Ax - b)$$
$$J(x) = x^T A^T A x - x^T A^T b - b^T A x + b^T b$$
Since $x^T A^T b$ is a scalar, it is equal to its transpose: $(x^T A^T b)^T = b^T A x$. Thus, we can combine the middle terms:
$$J(x) = x^T A^T A x - 2 b^T A x + b^T b$$
To find the minimum, take the gradient with respect to $x$ and set it to the zero vector. 
Using the matrix calculus identities $\frac{\partial (x^T M x)}{\partial x} = 2Mx$ (for symmetric $M$) and $\frac{\partial (c^T x)}{\partial x} = c$:
$$\frac{\partial J}{\partial x} = 2 A^T A x - 2 A^T b = 0$$
Dividing by 2 and rearranging yields the normal equations:
$$A^T A x = A^T b$$

**Solution 5: Procrustes Problem and Translation**
We want to minimize $f(t) = ||P_1 - \alpha R P_2 - t1^T||_F^2$. The optimal $t$ occurs where the derivative w.r.t $t$ is zero. Let $X = P_1 - \alpha R P_2 - t1^T$.
The Frobenius norm squared can be written as $trace(X X^T)$. Taking the derivative with respect to the vector $t$ and setting it to 0 gives:
$$-2(P_1 - \alpha R P_2 - t1^T)1 = 0$$
Distribute the vector of ones ($1$) into the parentheses:
$$P_1 1 - \alpha R P_2 1 - t(1^T 1) = 0$$
We are given that the points are centered, which mathematically means $P_1 1 = 0$ (the sum of the columns is zero) and $P_2 1 = 0$. 
Also, $1^T 1 = N$ (the dot product of an $N$-vector of ones with itself). Substituting these in:
$$0 - \alpha R (0) - t N = 0$$
$$-t N = 0 \implies t = 0$$
This formally proves that if the point clouds are centered at the origin, the optimal translation between them is zero, effectively decoupling translation from rotation and scale.

**Solution 6: SVD and Nullspace**
By definition, a vector $x$ is in the nullspace of $A$ if $Ax = 0$.
Substitute the SVD of $A$:
$$(U \Sigma V^T) x = 0$$
Since $U$ is an orthogonal matrix, it is invertible, and we can pre-multiply by $U^T$:
$$\Sigma V^T x = 0$$
Let $y = V^T x$. Since $A$ has rank $r$, the diagonal matrix $\Sigma$ has exactly $r$ strictly positive singular values ($\sigma_1, \dots, \sigma_r$), and the remaining $n - r$ diagonal entries are zero.
For the equation $\Sigma y = 0$ to hold, the first $r$ elements of $y$ must be zero, because $\sigma_i y_i = 0 \implies y_i = 0$ for $i \le r$.
However, the elements $y_{r+1}$ through $y_n$ can be anything, because their corresponding singular values are $0$ ($0 \cdot y_i = 0$, which is always true).
Therefore, any vector $y$ in the nullspace of $\Sigma$ can be formed by a linear combination of the standard basis vectors $e_{r+1}, \dots, e_n$.
Since $y = V^T x$, we have $x = V y$. 
This means $x$ is a linear combination of the columns of $V$ corresponding to $y_{r+1}, \dots, y_n$. These are exactly the last $n - r$ columns of $V$. Because $V$ is an orthogonal matrix, its columns are mutually orthogonal and have unit length, forming an orthonormal basis for the nullspace.