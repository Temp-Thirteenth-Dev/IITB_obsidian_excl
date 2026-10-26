
To ask doubts

1. Eigen Values capture the streching on the Eigen vectors only, while singular values gives the maximum strecth no matter the direction!


---

$\sum ((X-c)^2)$

$E[x] \ge \int_{a}^{\infty} x f(x) dx \ge\int_{a}^{\infty} x f(x) dx$

$Σ p_X(x) p_Y(z−x)$

$e^{-t \sqrt n} E[e^{tX_{n}/\sqrt n}]$

MGF of poisson : 



$$
e^{-t \sqrt{ n }} \sum_{i} e^{\frac{t}{\sqrt n} i} \frac{e ^{- \lambda} \lambda ^i}{i!} 
$$
$$
e^{-t \sqrt{ n }-\lambda} \sum_{i} e^{\frac{t}{\sqrt n} i} \frac{ \lambda ^i}{i!} 
$$
$$
e^{-t \sqrt{ n }-\lambda} \sum_{i}  \frac{{e^{\frac{t}{\sqrt n} } \lambda}^i} {i!} 
$$



Ahh, good catch! 😄 You're asking something important: we started with the CDF, but then suddenly used a change-of-variables formula. Why did we introduce the CDF at all?

The truth is, there are two different ways to solve this problem. Let me show you.

## 1. Method 1: Using the CDF (the direct approach)

We want to find the PDF of \(Z=XY\).

By definition,



Since \(X\) can be positive or negative, we would need to split the probability into cases:

- If \(X>0\), then \(XY\le z\) implies \(Y\le z/X\).
    
- If \(X<0\), then \(XY\le z\) implies \(Y\ge z/X\).
    

Using independence, we get

\[ \begin{aligned} F_Z(z) ={}&\int_0^\infty F_Y(z/x)f_X(x)\,dx\\ &+\int_{-\infty}^0 \left[1-F_Y(z/x)\right]f_X(x)\,dx \end{aligned} \]

Here, \(F_Y\) is the CDF of \(Y\).

We could now differentiate this expression with respect to \(z\) to obtain the PDF of \(Z\). Doing so gives

\[ \boxed{ f_Z(z)=\int_{-\infty}^{\infty} \frac{1}{|x|}f_Y(z/x)f_X(x)\,dx } \]

(assuming the usual regularity conditions that allow differentiation under the integral).

Notice that the positive and negative cases combine into the absolute value \(|x|\).

## 2. Method 2: Using conditional PDFs (the approach we used earlier)

Instead of finding the CDF first, we can:

1. Fix \(X=x\).
    
2. Find the conditional PDF of \(Z=xY\) using change of variables.
    
3. Average over all possible values of \(X\).
    

This gives us directly:

\[ \boxed{ f_Z(z)=\int_{-\infty}^{\infty} \frac{1}{|x|}f_Y(z/x)f_X(x)\,dx } \]

No explicit CDF calculation is necessary!

## 3. So, why did I mention the CDF?

Because the CDF is one fundamental way to derive a PDF, and it helps explain where probability distributions come from.

But for this particular question, if you already know the change-of-variables formula and the law of total probability for densities, you can skip the CDF entirely.

For your test, the shorter derivation is:

\[ \begin{gathered} Z=XY,\quad X=x\\ Y=z/x,\qquad \left|\frac{dY}{dz}\right|=\frac1{|x|}\\[4pt] f_{Z\mid X}(z\mid x) =\frac1{|x|}f_Y(z/x)\\[6pt] f_Z(z)=\int_{-\infty}^{\infty} f_{Z\mid X}(z\mid x)f_X(x)\,dx \end{gathered} \]

Therefore,

\[ \boxed{ f_Z(z)=\int_{-\infty}^{\infty} f_X(x)f_Y(z/x)\frac1{|x|}\,dx } \]

The key takeaway: The CDF was an alternative starting point, not an extra step you must always perform.

---

$F(Z) = \int_{-\infty}^{z} f(z)\,dz$

- PDF: $f_W(w)=\sum_i p_i f_{X_i}(w)$
    
- CDF: $F_W(w)=\sum_i p_i F_{X_i}(w)$

for E\[w\] :
$E[w] =  \sum_{i=1}^{n}  \int_{- \infty}^{\infty} w [p_i f_{x_i} (w) ] dw$

$E[w] =  \sum_{i=1}^{n} p_i  \int_{- \infty}^{\infty} w [ f_{x_i} (w) ] dw$

$E[w] =  \sum_{i=1}^{n} p_i  E[X_i]$

for Var\[W\]:
$Var[w] =  \sum_{i=1}^{n}  \int_{- \infty}^{\infty} w^2 [p_i f_{x_i} (w) ] dw -   (\sum_{i=1}^{n} p_i \mu_i)^2$ 

$Var[w] =  \sum_{i=1}^{n} p_i  \int_{- \infty}^{\infty} w^2 [ f_{x_i} (w) ] dw -   (\sum_{i=1}^{n} p_i \mu_i)^2$

$Var[w] =  \sum_{i=1}^{n} p_i  E[X_i^2] -   (\sum_{i=1}^{n} p_i \mu_i)^2$

$E[X_i^2] = \sigma_i^2 + \mu_i^2$

$Var[w] =  \sum_{i=1}^{n} p_i [\sigma_i^2 + \mu_i^2]  -   (\sum_{i=1}^{n} p_i \mu_i)^2$

MGF(W)

$M_W(t) = E[e^tW] =  \int_{- \infty}^{\infty} e^{tW} f_W(w)dw$
$=  \int_{- \infty}^{\infty} e^{tW}\sum_{i=1}^{n} p_i f_{X_i}(w) dw$
$=\sum_{i=1}^{n} p_i  \int_{- \infty}^{\infty} e^{tW} f_{X_i}(w) dw$

$=\sum_{i=1}^{n} p_i   (M_{X_i}(t))$


---
Since WW is equal to XiX_i with probability pip_i, the PDF and CDF of WW are weighted mixtures of the PDFs and CDFs of the random variables X1,…,XnX_1,\ldots,X_n.

## 1. PDF of WW

Let fW(w)f_W(w) denote the PDF of WW. Then

fW(w)=∑i=1npifXi(w)\boxed{f_W(w)=\sum_{i=1}^{n}p_i f_{X_i}(w)}

Equivalently,

fW(w)=p1fX1(w)+p2fX2(w)+⋯+pnfXn(w).f_W(w)=p_1f_{X_1}(w)+p_2f_{X_2}(w) +\cdots+p_nf_{X_n}(w).

Explanation: Since W=XiW=X_i with probability pip_i, each PDF contributes to the overall PDF in proportion to its selection probability.

## 2. CDF of WW

Let FW(w)=P(W≤w)F_W(w)=P(W\leq w) denote the CDF of WW. By the law of total probability,

FW(w)=P(W≤w)=∑i=1nP(W≤w∣W=Xi)pi=∑i=1npiP(Xi≤w).\begin{aligned} F_W(w)&=P(W\leq w)\\ &=\sum_{i=1}^{n}P(W\leq w\mid W=X_i)p_i\\ &=\sum_{i=1}^{n}p_iP(X_i\leq w). \end{aligned}

Therefore,

FW(w)=∑i=1npiFXi(w)\boxed{F_W(w)=\sum_{i=1}^{n}p_iF_{X_i}(w)}

where

FXi(w)=∫−∞wfXi(t) dt.F_{X_i}(w)=\int_{-\infty}^{w}f_{X_i}(t)\,dt.

Hence, the CDF can also be written in terms of the PDFs as

FW(w)=∑i=1npi∫−∞wfXi(t) dt\boxed{ F_W(w)=\sum_{i=1}^{n}p_i \int_{-\infty}^{w}f_{X_i}(t)\,dt }

## Final result

fW(w)=∑i=1npifXi(w),FW(w)=∑i=1npiFXi(w).\boxed{ \begin{aligned} f_W(w)&=\sum_{i=1}^{n}p_i f_{X_i}(w),\\[6pt] F_W(w)&=\sum_{i=1}^{n}p_i F_{X_i}(w). \end{aligned} }

These formulas assume that the selection of XiX_i is governed by the probabilities pip_i, independently of the realized values of the random variables.

Note: The given means μi\mu_i and variances σi2\sigma_i^2 are not needed to determine the PDF and CDF. They can, however, be used to calculate the mean and variance of WW.

---
$$ \|A\|_F^2=\operatorname{tr}(A^TA). \$$$$


---

Since you've already gone through the lectures, the best practice now is a midsem-style problem set that tests whether you can derive results, prove properties, and apply concepts to unfamiliar matrices — not just reproduce the assignment solutions.

I'll follow the style and mathematical level of your two CS 740 assignments, use the full Lecture 1–19 syllabus, and exclude MATLAB/programming questions. I'll include some familiar question types with changed conditions, some multi-step derivations, and some conceptual traps.

The questions below are intended for practice without looking at the solutions. They are newly constructed practice questions, not predictions of the actual exam.

# CS 740 Midsem Practice Set — Set 1

## Mathematical Methods for Visual Computing

Practice paper

Questions

# 15

Total marks

# 100

Style

## Written

Suggested practice time: 2.5–3 hours. All questions are theoretical or numerical; no MATLAB is required.

## Section A — Linear algebra foundations

### Q1. Trace, norms and matrix properties $$6 marks$$

Let $A\in\mathbb R^{m\times n}$.

(a) Prove that

$\\|A\\|\_F^2=\operatorname{tr}(A^TA).$
(Done this in the assignment, show the diagnol elements sum is the sum of the sq of all elements, so I'm skipping this.)

(b) Given that $A$ has full column rank, show that $A^TA$ is positive definite.

for pd,$\forall x \neq 0 , x^t P x > 0$
$\text{Here , } x^t A^T A x = (Ax)^T(Ax)$
Since$Ax$ is a vecotr ,$(Ax)^T(Ax) = \| Ax \|$
Since A is a full col rank matrix;$x\neq 0$ ;$Ax\neq 0$ ; i.e x won't be in null space of A. so as Ax !=0,$\| Ax \|>0$ Hence$x^t A^T A x > 0$

Good Sol :
Since (A) has full column rank, its nullspace is trivial:  
$$
\mathcal N(A)={0}.  
$$

For any ($x\in\mathbb R^n), (x\neq 0$),  
$$
\begin{aligned}  
x^T A^T A x  
&=(Ax)^T(Ax)\  
&=|Ax|_2^2.  
\end{aligned}  
$$

Since ($x\neq 0$) and ($\mathcal N(A)={0}$), we have ($Ax\neq 0$). Therefore,  
$$
|Ax|_2^2>0.  
$$

Hence $(x^TA^TAx>0)$ for every ($x\neq 0$), so ($A^TA$) is positive definite.


(c) If $Q\in\mathbb R^{m\times m}$ is orthogonal, prove that

$\\|QA\\|\_F=\\|A\\|\_F.$

$\|QA\|_F \text{  } tr((QA)^T (QA)) = tr(A^T Q^T (QA)) = tr(A^T A) = \|A\|_F$ 
(since Q^T Q =I)

Good sol. :
$$  
\begin{aligned}  
|QA|_F^2  
&=\operatorname{tr}\big((QA)^T(QA)\big)\  
&=\operatorname{tr}(A^TQ^TQA)\  
&=\operatorname{tr}(A^TIA)\  
&=\operatorname{tr}(A^TA)\  
&=|A|_F^2.  
\end{aligned}  
$$

Since norms are nonnegative, taking square roots gives  
$$  
\boxed{|QA|_F=|A|_F}.  
$$

You may use the cyclic property of trace, $\operatorname{tr}(XY)=\operatorname{tr}(YX)$, whenever the matrix products are defined.

### Q2. Rank, nullspace and linear independence $$6 marks$$

Consider the matrix

$A= \begin{bmatrix} 1&2&3\\\ 2&4&6\\\ 1&1&2 \end{bmatrix}.$

(a) Determine the rank of $A$, showing your reasoning.
my sol: since we can see the R2 is just 2xR1 , and r3 is independent, rank is 2. (but reasoning seems vague? can you help me how to write good solution for this?)

(b) Find a basis for the nullspace of $A$.
$A= \begin{bmatrix} 1&2&3\\\ 0&0&0\\\ 0&0&-1 \end{bmatrix} \begin{bmatrix} x_1\\\ x_2 \\\ x_3 \end{bmatrix} = \begin{bmatrix} 0\\\ 0 \\\ 0 \end{bmatrix}  .$
solving it gives $k * \begin{bmatrix} -0.5\\\ 1 \\\ 0 \end{bmatrix}$

(c) Are the columns of $A$ linearly independent? Explain.
No, as rank is 2 (and clearly C3 is sum of C2, C1) . How to show it better shall I find det and make it to 0?? 
(d) If $A$ represents a linear transformation from $\mathbb R^3$ to $\mathbb R^3$, describe the dimension of its column space and nullspace.
col space dim is the rank which is 2, and nullity is 1. (more details required?)

(If you are giving solutions, use dollor signs $ $ for math, i.e which are supprted by obsidian. please remeber it for all responses. Thank you.)

### Q2 Sols:
Your reasoning is mostly correct! The main thing to improve is how you justify each conclusion mathematically, rather than relying on visual inspection. Let's review each part as a midsem answer.

#### (a) Determine the rank of $A$

Your observation that $R_2=2R_1$ is correct. However, saying that $R_3$ is independent needs justification.

Given

$$ A=\begin{pmatrix} 1&2&3 \\ 2&4&6 \\ 1&1&2 \end{pmatrix} $$

**Step 1: Perform elementary row operations.**

Apply

$$ R_2\leftarrow R_2-2R_1,\qquad R_3\leftarrow R_3-R_1. $$

This gives

$$ \begin{pmatrix} 1&2&3 \\ 0&0&0 \\ 0&-1&-1 \end{pmatrix}. $$

Swap the second and third rows to obtain row-echelon form:

$$ \begin{pmatrix} 1&2&3 \\ 0&-1&-1 \\ 0&0&0 \end{pmatrix}. $$

**Step 2: Count the nonzero rows.**

There are two nonzero rows, so

$$ \boxed{\text{rank}(A) = 2}. $$

This is a strong exam answer because row reduction explicitly demonstrates the rank.

An alternative: You can argue that $R_2=2R_1$, so the first two rows contribute only one independent row. Also, $R_3$ is not a scalar multiple of $R_1$, since $(1,1,2)$ is not a scalar multiple of $(1,2,3)$. Thus, the first and third rows are independent, and the rank is $2$.

Both approaches are valid. I recommend row reduction because it is systematic and works for larger matrices, too.


## (b) Find a basis for the nullspace of $A$

Your final answer is correct, but there is a small issue with how you wrote the equation: the reduced matrix should be multiplied by the three-component vector $x$, not a vector with only two components.

We want to solve

$$ Ax=0,\qquad x=\begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix}. $$

Using the row-reduced matrix from part (a), we get

$$ \begin{pmatrix} 1&2&3 \\ 0&-1&-1 \\ 0&0&0 \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \\ x_3 \end{pmatrix} = \begin{pmatrix} 0 \\ 0 \\ 0 \end{pmatrix}. $$

This gives two independent equations:

$$ x_1+2x_2+3x_3=0 $$

$$ -x_2-x_3=0. $$

From the second equation,

$$ x_2=-x_3. $$

Substitute this into the first equation:

$$ x_1+2(-x_3)+3x_3=0 $$

$$ x_1=-x_3. $$

Let the free variable $x_3=t$. Then

$$ x=t\begin{pmatrix} -1 \\ -1 \\ 1 \end{pmatrix}. $$

Therefore, the nullspace is

$$ \mathcal N(A) = \text{span}\left\{ \begin{pmatrix} -1 \\ -1 \\ 1 \end{pmatrix} \right\}. $$


### Q3. Least-squares estimation 8 marks

Suppose a linear system $Ax=b$ is overdetermined, with $A\in\mathbb R^{m\times n}$, $m>n$, and $A$ has full column rank.

(a) Starting from the minimization problem

$\min_x\\|Ax-b\\|\_2^2,$

derive the normal equations.

(b) Show that the least-squares solution is

$\hat x=(A^TA)^{-1}A^Tb.$

(c) Explain geometrically why the residual $r=b-A\hat x$ is orthogonal to every column of $A$.

(d) What difficulty arises if the columns of $A$ are linearly dependent?

## (c) Geometric explanation

The least-squares problem asks for the point in the column space $\operatorname{col}(A)$ that is closest to $b$. That point is $A\hat{x}$, and it is the **orthogonal projection** of $b$ onto $\operatorname{col}(A)$.

The residual $r = b - A\hat{x}$ is the vector from the projection back to $b$. The shortest path from a point to a subspace is the perpendicular one, so $r \perp \operatorname{col}(A)$. Since $\operatorname{col}(A)$ is spanned by the columns $a_1,\dots,a_n$, this means $a_j^T r = 0$ for every $j$.

**Why it must be perpendicular:** Suppose $r$ had a nonzero component along some column $a_j$. Then you could move $A\hat{x}$ a small step along $a_j$ (by adjusting $\hat{x}$) and get closer to $b$, which contradicts $\hat{x}$ being the minimizer. At the minimum, no such component can remain.

**Algebraic check:** The statement $a_j^T r = 0$ for all $j$ is exactly $A^T r = 0$, which gives

$$A^T(b - A\hat{x}) = 0 ;\Longleftrightarrow; A^TA\hat{x} = A^Tb,$$

the normal equations.

## (d) Dependent columns

If the columns of $A$ are linearly dependent, then $\ker(A) \neq {0}$, and $A^TA$ is **singular**. The consequences:

- **The solution is not unique.** The normal equations have infinitely many solutions. If $\hat{x}$ is one, so is $\hat{x} + z$ for any $z \in \ker(A)$, since $A(\hat{x}+z) = A\hat{x}$.
- **The projection is still unique.** The vector $A\hat{x}$ and the residual $r$ are the same for every solution. Only the coefficients are ambiguous.
- **You can't use $\hat{x} = (A^TA)^{-1}A^Tb$**, because the inverse doesn't exist.

**Remedies:**

- Select the minimum-norm solution $\hat{x} = A^{+}b$ using the pseudoinverse (computed via the SVD).
- Remove redundant columns, for example with QR with column pivoting, so that the remaining columns are independent.
- Add regularization such as ridge regression, which makes $A^TA + \lambda I$ invertible.

**Numerical note:** Even when columns are only _nearly_ dependent, $A^TA$ is badly conditioned, since $\kappa(A^TA) = \kappa(A)^2$. In practice it is better to solve with QR or the SVD than with the normal equations directly.
### Q4. Orthogonal matrices and vector norms $$5 marks$$

Let $Q\in\mathbb R^{n\times n}$ satisfy $Q^TQ=I$.

(a) Prove that $\\|Qx\\|\_2=\\|x\\|\_2$ for any $x\in\mathbb R^n$.

(b) Prove that the angle between two nonzero vectors $x,y$ is unchanged when both are transformed by $Q$.

(c) Does every orthogonal matrix represent a pure rotation? Explain the role of $\det(Q)$.

## Section B — Eigenvalues and numerical stability

### Q5. Eigenvalues of a structured matrix $$7 marks$$

Let $u\in\mathbb R^n$ be a unit vector, and define

$H=I-2uu^T.$

(a) Prove that $H$ is symmetric and orthogonal.

(b) Determine all possible eigenvalues of $H$ and their corresponding eigenspaces.

(c) What is the determinant of $H$?

(d) Geometrically, describe the transformation represented by $H$.

### Q6. Rank-one matrices and their eigenstructure $$7 marks$$

Let $u,v\in\mathbb R^n$ be nonzero vectors and let

$A=uv^T.$

(a) Prove that $\operatorname{rank}(A)=1$.

(b) Show that $v$ is an eigenvector of $A$, and determine its eigenvalue.

(c) Find a condition under which $u$ is an eigenvector of $A$.

(d) Show that every vector $x$ satisfying $v^Tx=0$ belongs to the nullspace of $A$.

(e) Is $uv^T$ necessarily symmetric? State the condition for symmetry.

### Q7. Power iteration and inverse iteration $$8 marks$$

Consider the symmetric matrix

$A= \begin{bmatrix} 4&0\\\ 0&-2 \end{bmatrix}, \qquad x_0=\frac{1}{\sqrt{5}} \begin{bmatrix} 1\\\2 \end{bmatrix}.$

(a) Perform two iterations of the power method, normalizing the vector after every iteration. Use the normalization convention that the first component is positive.

(b) Which eigenvector does the power method converge to? Explain why.

(c) Suppose instead that you want the eigenvector corresponding to the eigenvalue closest to $\alpha= -1.8$. Which iterative method would you use, and why?

(d) Explain how the eigenvalues of $(A-\alpha I)^{-1}$ are related to those of $A$.

### Q8. Condition number and sensitivity $$6 marks$$

Suppose $A$ is an invertible matrix, and $Ax=b$. A perturbed right-hand side is $b+\delta b$, producing the solution $x+\delta x$.

(a) Starting from the two linear systems, derive

$\frac{\\|\delta x\\|\_2}{\\|x\\|\_2} \leq \kappa_2(A) \frac{\\|\delta b\\|\_2}{\\|b\\|\_2},$

where $\kappa_2(A)=\\|A\\|\_2\\|A^{-1}\\|\_2$.

(b) What does a large condition number imply about the stability of solving the system?

(c) If the singular values of $A$ are $10$ and $0.01$, determine $\kappa_2(A)$.

## Section C — Singular value decomposition

### Q9. Deriving SVD from eigendecomposition $$8 marks$$

Let $A\in\mathbb R^{m\times n}$ have rank $r$, and suppose

$A=U\Sigma V^T$

is its SVD.

(a) Explain why the nonzero singular values of $A$ are the square roots of the nonzero eigenvalues of $A^TA$.

(b) Prove that if $v_i$ is a unit eigenvector of $A^TA$ corresponding to a positive eigenvalue $\sigma_i^2$, then

$u_i=\frac{Av_i}{\sigma_i}$

is a unit vector.

(c) Prove that the resulting $u_i$ and $v_i$ satisfy $Av_i=\sigma_i u_i$.

(d) Why can independently computing eigenvectors of $AA^T$ and $A^TA$ cause problems when reconstructing $A$?

### Q10. Best low-rank approximation and the Eckart–Young theorem $$7 marks$$

Consider the matrix

$A= \begin{bmatrix} 5&0\\\ 0&3\\\ 0&0 \end{bmatrix}.$

(a) Determine its singular values and one valid SVD.

(b) Find the best rank-one approximation $A_1$ to $A$ in Frobenius norm.

(c) Compute $\\|A-A_1\\|\_F$.

(d) State the Eckart–Young theorem and explain why simply retaining the largest singular value gives the optimal rank-one approximation.

### Q11. Pseudoinverse and nullspace using SVD $$7 marks$$

Suppose $A\in\mathbb R^{m\times n}$ has the reduced SVD

$A=U_r\Sigma_rV_r^T,$

where $r=\operatorname{rank}(A)$.

(a) Write the Moore–Penrose pseudoinverse $A^+$ in terms of $U_r,\Sigma_r,V_r$.

(b) Show that $A^+A=V_rV_r^T$.

(c) Explain why $A^+A$ acts as an orthogonal projector onto the row space of $A$.

(d) Explain how the right singular vectors corresponding to zero singular values can be used to determine the nullspace of $A$.

## Section D — Procrustes, PCA and numerical methods

### Q12. Orthogonal Procrustes and nearest orthogonal matrix $$8 marks$$

(a) Let $C\in\mathbb R^{n\times n}$. We wish to solve

$\min\_{R^TR=I}\\|X-RY\\|\_F^2.$

Show that this is equivalent to maximizing an expression involving $\operatorname{tr}(R^TC)$, where $C=XY^T$.

(b) Given the SVD $C=U\Sigma V^T$, derive the optimal orthogonal matrix $R^\star$.

(c) Using the SVD $A=U\Sigma V^T$, find the orthogonal matrix closest to $A$ in Frobenius norm.

(d) Explain the difference between an orthogonal matrix and a proper rotation matrix. How might the solution need to change if reflections are prohibited?

### Q13. Similarity Procrustes with translation $$6 marks$$

Two sets of corresponding points are represented by matrices $P_1,P_2\in\mathbb R^{3\times N}$, related by

$P_1=\alpha RP_2+t\mathbf 1^T+W,$

where $R^TR=I$, $\alpha\geq0$, and $t\in\mathbb R^3$.

(a) Define the centroids of the two point sets and derive their centered representations $X,Y$.

(b) Show that the optimal translation, for fixed $\alpha$ and $R$, is

$\hat t=\bar p_1-\alpha R\bar p_2.$

(c) Explain how the SVD of a matrix formed from $X$ and $Y$ is used to estimate $R$.

(d) Why is centering the points useful before estimating rotation and scale?

### Q14. Principal component analysis $$6 marks$$

Suppose the columns of $X\in\mathbb R^{d\times N}$ are centered data samples.

(a) Define the covariance matrix $C=\frac1NXX^T$. Explain why it is positive semidefinite.

(b) Show that if $v$ is a unit eigenvector of $C$ with eigenvalue $\lambda$, then the variance of the data projected onto $v$ is $\lambda$.

(c) Explain why PCA chooses eigenvectors associated with the largest eigenvalues.

(d) If $d$ is much larger than $N$, explain how eigendecomposition of $X^TX$ can help avoid constructing the $d\times d$ covariance matrix.

### Q15. QR decomposition and Cholesky factorization $$5 marks$$

(a) Suppose the columns of $A\in\mathbb R^{m\times n}$ are linearly independent. Explain how Gram–Schmidt produces a factorization $A=QR$, and state the properties of $Q$ and $R$.

(b) Let

$B= \begin{bmatrix} 4&2\\\ 2&3 \end{bmatrix}.$

Find a Cholesky factorization $B=LL^T$, where $L$ is lower triangular with positive diagonal entries.

(c) State one advantage of using Cholesky factorization instead of general Gaussian elimination when solving a linear system with a symmetric positive definite matrix.

## How to practise effectively

I recommend attempting this set in three passes:

1. Pass 1 — Closed-book: Solve the proofs and derivations without referring to your notes.
2. Pass 2 — Numerical problems: Check your calculations for power iteration, singular values, rank, and Cholesky factorization.
3. Pass 3 — Exam-style writing: Make sure every proof states its assumptions, shows the essential algebra, and ends with the required conclusion.

### Practice tracker

0/15 completed

Q1

Q2

Q3

Q4

Q5

Q6

Q7

Q8

Q9

Q10

Q11

Q12

Q13

Q14

Q15

My recommendation: Start with Q1–Q8, then move to SVD and Procrustes. These questions build directly on the mathematical style of your assignments, while Q14–Q15 broaden practice to topics explicitly included in your lecture outline.

When you're ready, send me your answers — even if they're incomplete — and I'll check them like an examiner, point out missing steps, and explain how to improve them.