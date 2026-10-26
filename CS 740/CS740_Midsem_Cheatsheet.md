# CS 740 — Midsem Cheatsheet

**Mathematical Methods for Visual Computing**  
*Formula-first reference for assignment-style, no-MATLAB questions*

> **Notation:** Unless stated otherwise, vectors are columns. For a matrix $A\in\mathbb R^{m\times n}$, $A^T$ is its transpose, $\|A\|_F$ is its Frobenius norm, and $\|A\|_2$ is its spectral norm. Use the matrix dimensions to check every product.

## 1. Core identities and matrix properties

### Trace and Frobenius norm

- $\operatorname{tr}(A)=\sum_i A_{ii}$ for a square matrix.
- **Cyclic trace property:** $\operatorname{tr}(XY)=\operatorname{tr}(YX)$ when dimensions allow; more generally, factors can be cyclically rotated inside a trace.
- $\|A\|_F^2=\sum_{i,j}a_{ij}^2=\operatorname{tr}(A^TA)=\operatorname{tr}(AA^T)$.
- $\|A\|_F^2=\sum_i\sigma_i^2$ for singular values $\sigma_i$.
- $\|A\|_2=\sigma_{\max}(A)$.

**Proof pattern for $\|A\|_F^2=\operatorname{tr}(A^TA)$:** the $j$-th diagonal element of $A^TA$ is $\sum_i a_{ij}^2$. Summing all diagonal elements sums the squares of every entry of $A$.

### Orthogonal matrices

$Q$ is orthogonal if $Q^TQ=QQ^T=I$. Therefore:

- $Q^{-1}=Q^T$ and $\|Qx\|_2=\|x\|_2$.
- $(Qx)^T(Qy)=x^Ty$: inner products and angles are preserved.
- $\|QA\|_F=\|A\|_F$ and $\|AQ\|_F=\|A\|_F$ when the products are defined.
- $\det(Q)=\pm1$. $\det(Q)=+1$ corresponds to a proper rotation; $\det(Q)=-1$ includes reflections.

**Proof pattern:** square the Frobenius norm, write it as a trace, then use $Q^TQ=I$:

$$\|QA\|_F^2=\operatorname{tr}((QA)^TQA)=\operatorname{tr}(A^TQ^TQA)=\operatorname{tr}(A^TA)=\|A\|_F^2.$$

Take square roots at the end.

### Positive definite / semidefinite

- Symmetric $M$ is **positive definite (PD)** if $x^TMx>0$ for every $x\ne0$.
- Symmetric $M$ is **positive semidefinite (PSD)** if $x^TMx\ge0$ for every $x$.
- For any $A$, $A^TA$ is PSD because $x^TA^TAx=\|Ax\|_2^2\ge0$.
- If $A$ has **full column rank**, $Ax=0$ implies $x=0$; hence $x^TA^TAx=\|Ax\|_2^2>0$ for every $x\ne0$. Thus $A^TA$ is PD.
- $A^TA$ is always symmetric: $(A^TA)^T=A^TA$.

### Rank, nullspace and dimensions

- $\operatorname{rank}(A)$ = number of pivots = number of linearly independent columns = number of linearly independent rows.
- $\mathcal N(A)=\{x:Ax=0\}$; its dimension is the nullity.
- **Rank–nullity:** $\operatorname{rank}(A)+\dim\mathcal N(A)=n$, where $n$ is the number of columns.
- The dimension of $\operatorname{Col}(A)$ is $\operatorname{rank}(A)$.
- Columns are independent iff $Ax=0$ has only the zero solution iff $\operatorname{rank}(A)=n$.
- For square $A$, $\det(A)=0$ iff $A$ is singular iff its columns are dependent. Determinants are not a rank test for non-square matrices.
- To find a nullspace basis: row-reduce, solve $Ax=0$, introduce one free parameter per free variable, and write the solution as a span.

**Fast exam method:** row-reduce to echelon form and count nonzero rows. Then use pivot/free variables to find rank and nullspace.

## 2. Least squares and derivatives

### Vector least squares

Given an inconsistent or overdetermined system $Ax\approx b$, minimize

$$\min_x\|Ax-b\|_2^2.$$

Expand and differentiate:

$$f(x)=\|Ax-b\|_2^2,\qquad \nabla f(x)=2A^T(Ax-b).$$

Set the gradient to zero to obtain the **normal equations**:

$$A^TA\hat x=A^Tb.$$

If $A$ has full column rank,

$$\hat x=(A^TA)^{-1}A^Tb.$$

Important facts:

- The residual $r=b-A\hat x$ is orthogonal to every column of $A$: $A^Tr=0$.
- Equivalently, $A\hat x$ is the orthogonal projection of $b$ onto $\operatorname{Col}(A)$.
- If columns are dependent, $A^TA$ is singular and the minimizer may not be unique; use a pseudoinverse/SVD conceptually.
- In numerical computation, QR or SVD is generally more stable than explicitly forming normal equations; $\kappa_2(A^TA)=\kappa_2(A)^2$ when $A$ has full column rank.

### Linear and polynomial fitting

For data $(x_i,y_i)$, fit a cubic $y\approx a+bx+cx^2+dx^3$ using

$$X=\begin{bmatrix}1&x_1&x_1^2&x_1^3\\\vdots&\vdots&\vdots&\vdots\\1&x_N&x_N^2&x_N^3\end{bmatrix},\quad c=\begin{bmatrix}a\\b\\c\\d\end{bmatrix},\quad Xc\approx y.$$

Then solve the least-squares problem $\min_c\|Xc-y\|_2^2$. The method is the same regardless of the number of polynomial terms.

### Matrix least squares for point transforms

With point coordinates stored as **columns**, $P_1,P_2\in\mathbb R^{3\times N}$ and $AP_1\approx P_2$, minimize $\|AP_1-P_2\|_F^2$. If $P_1P_1^T$ is invertible,

$$\hat A=P_2P_1^T(P_1P_1^T)^{-1}.$$

Equivalently, $\hat A=P_2P_1^\dagger$ using the pseudoinverse. If the source points are collinear in homogeneous 2D coordinates, the design matrix lacks sufficient rank and the affine map is not uniquely determined.

### Derivative/Jacobian facts

- For $y=Ax$, with $A\in\mathbb R^{m\times n}$, the Jacobian is $\partial y/\partial x=A$ (entries $\partial y_i/\partial x_j=a_{ij}$).
- $\nabla_x\|Ax-b\|_2^2=2A^T(Ax-b)$.
- If $M=M^T$, then $\nabla_x(x^TMx)=2Mx$.

## 3. Eigenvalues, eigenvectors and special matrices

### Eigenvalue facts

- $Av=\lambda v$ for $v\ne0$.
- If $A$ is real symmetric, it has an orthonormal eigenbasis: $A=V\Lambda V^T$.
- For symmetric $A$, all eigenvalues are real; $A$ is PSD iff all eigenvalues are nonnegative, and PD iff all are positive.
- $\operatorname{tr}(A)=\sum_i\lambda_i$ and $\det(A)=\prod_i\lambda_i$ (counting algebraic multiplicity).
- Eigenvectors can be multiplied by $-1$ without changing the eigenpair. Compare eigenvector directions using $|v^T\hat v|/(\|v\|\|\hat v\|)$, not the signed dot product.

### Householder reflection

For nonzero $v$,

$$H=I-2\frac{vv^T}{v^Tv}.$$

If $u=v/\|v\|$, then $H=I-2uu^T$.

- $H^T=H$ and $H^2=I$, so $H$ is orthogonal.
- $Hv=-v$: direction parallel to $v$ has eigenvalue $-1$.
- If $v^Tw=0$, then $Hw=w$: every direction perpendicular to $v$ has eigenvalue $+1$.
- In $n$ dimensions: one $-1$ eigenvalue and $n-1$ eigenvalues $+1$; $\det(H)=-1$.
- Decompose $x=x_\parallel+x_\perp$, where $x_\parallel=uu^Tx$ and $u^Tx_\perp=0$. Then $Hx=-x_\parallel+x_\perp$.

### Rank-one outer product

Let $B=uv^T$ with nonzero $u,v\in\mathbb R^n$.

- $Bx=u(v^Tx)$, so the image is contained in $\operatorname{span}\{u\}$; since $B\ne0$, $\operatorname{rank}(B)=1$.
- $\mathcal N(B)=\{x:v^Tx=0\}=v^\perp$ (dimension $n-1$).
- $Bu=u(v^Tu)$, so $u$ is an eigenvector with eigenvalue $v^Tu$.
- The characteristic polynomial is $\lambda^{n-1}(\lambda-v^Tu)$; if $v^Tu=0$, all eigenvalues are zero even though $B$ is nonzero.
- Special case $B=uu^T$: the eigenvalue in direction $u$ is $\|u\|^2$ and all perpendicular directions have eigenvalue $0$.

**Common trap:** $uv^Tv=\|v\|^2u$ does **not** make $v$ an eigenvector of $uv^T$ unless $u$ is proportional to $v$.

## 4. Power iteration and inverse iteration

### Power iteration

To estimate the eigenpair with largest eigenvalue magnitude:

1. Choose a nonzero initial vector $x_0$.
2. Compute $y_k=Ax_k$.
3. Normalize: $x_{k+1}=y_k/\|y_k\|_2$.
4. Estimate the eigenvalue, e.g. with the Rayleigh quotient:

$$\hat\lambda_k=\frac{x_k^TAx_k}{x_k^Tx_k}.$$

Under the usual conditions (a unique dominant eigenvalue in magnitude and a starting vector with a component in its eigenvector direction), $x_k$ converges to the dominant eigenvector direction. Power iteration finds the eigenvalue of **largest magnitude**, not necessarily the largest algebraic value.

For a symmetric matrix, after finding a normalized eigenpair $(\lambda_1,v_1)$, simple deflation is $A_{\text{new}}=A-\lambda_1v_1v_1^T$.

### Inverse iteration / shift-and-invert

If the desired eigenvalue is close to a known shift $\alpha$, iterate by solving

$$(A-\alpha I)y_k=x_k,\qquad x_{k+1}=y_k/\|y_k\|_2.$$

The eigenvalues of $(A-\alpha I)^{-1}$ are $1/(\lambda_i-\alpha)$. The largest reciprocal magnitude corresponds to the $\lambda_i$ **closest to $\alpha$**. Eigenvectors are unchanged. In practice, solve a linear system rather than explicitly computing a matrix inverse.

Special case $\alpha=0$: inverse iteration uses $A^{-1}$ and targets the eigenvalue of $A$ with smallest magnitude (assuming $A$ is invertible).

### Error measures

- Relative eigenvalue error: $\displaystyle \frac{|\hat\lambda-\lambda|}{|\lambda|}$.
- Angle between eigenvectors (degrees):

$$\theta=\cos^{-1}\!\left(\frac{|v^T\hat v|}{\|v\|_2\|\hat v\|_2}\right)\frac{180}{\pi}.$$

The absolute value handles the sign ambiguity $v$ versus $-v$.

## 5. SVD and pseudoinverse

### Singular value decomposition

For every $A\in\mathbb R^{m\times n}$,

$$A=U\Sigma V^T,$$

where $U,V$ are orthogonal (or have orthonormal columns in a reduced SVD), and $\Sigma$ contains singular values $\sigma_1\ge\sigma_2\ge\cdots\ge0$.

- $A^TAv_i=\sigma_i^2v_i$; right singular vectors are eigenvectors of $A^TA$.
- $AA^Tu_i=\sigma_i^2u_i$; left singular vectors are eigenvectors of $AA^T$.
- For $\sigma_i>0$, $u_i=Av_i/\sigma_i$ and $v_i=A^Tu_i/\sigma_i$.
- **Sign consistency:** if you compute one side's eigenvectors separately, their signs may not match. Deriving $u_i=Av_i/\sigma_i$ aligns the pair and ensures the correct reconstruction.
- $\operatorname{rank}(A)=\#\{i:\sigma_i>0\}$.
- $\|A\|_2=\sigma_1$, $\|A\|_F^2=\sum_i\sigma_i^2$.
- If $A$ is square and full rank, $|\det A|=\prod_i\sigma_i$; the singular values alone do not reveal the sign of $\det A$.

### Rank-$k$ approximation (Eckart–Young)

If $A=\sum_{i=1}^r\sigma_i u_i v_i^T$, the best rank-$k$ approximation in Frobenius and spectral norms is

$$A_k=\sum_{i=1}^k\sigma_i u_i v_i^T=U_k\Sigma_kV_k^T.$$

Errors:

$$\|A-A_k\|_F=\sqrt{\sum_{i=k+1}^r\sigma_i^2},\qquad \|A-A_k\|_2=\sigma_{k+1}.$$

The best rank-one approximation is $A_1=\sigma_1u_1v_1^T$.

### Moore–Penrose pseudoinverse

For compact SVD $A=U_r\Sigma_rV_r^T$ with positive singular values,

$$A^+=V_r\Sigma_r^{-1}U_r^T.$$

Then:

- $A^+A=V_rV_r^T$ is the orthogonal projector onto the row space of $A$.
- $AA^+=U_rU_r^T$ is the orthogonal projector onto the column space of $A$.
- The minimum-norm least-squares solution is $x=A^+b$.
- Right singular vectors associated with zero singular values span $\mathcal N(A)$.

### Row/column permutations

If $B=AP$ for permutation matrix $P$ (columns shuffled), the singular values stay the same; the right singular vectors are permuted. If $B=QA$ (rows shuffled), the singular values stay the same; the left singular vectors are permuted. This follows because permutation matrices are orthogonal.

## 6. Conditioning and numerical stability

For full-column-rank $A$, the 2-norm condition number is

$$\kappa_2(A)=\frac{\sigma_{\max}(A)}{\sigma_{\min}(A)}.$$

- Large $\kappa$ means small input errors may create large relative errors in the solution; the problem is ill-conditioned.
- For fixed $A$ and perturbed right-hand side $b$, a standard bound is

$$\frac{\|\delta x\|_2}{\|x\|_2}\le \kappa_2(A)\frac{\|\delta b\|_2}{\|b\|_2}.$$

- If the singular values are $10$ and $0.01$, then $\kappa_2(A)=10/0.01=1000$.
- Forming $A^TA$ squares the 2-norm condition number: $\kappa_2(A^TA)=\kappa_2(A)^2$ for full column rank.
- SVD reveals near-null directions (very small singular values) and is useful for diagnosing ill-conditioning.

## 7. QR factorization and Cholesky

### QR / Gram–Schmidt

For linearly independent columns $A=[a_1,\ldots,a_n]$, write $A=QR$, where columns of $Q$ are orthonormal and $R$ is upper triangular.

Classical Gram–Schmidt:

$$r_{ij}=q_i^Ta_j,\quad u_j=a_j-\sum_{i=1}^{j-1}(q_i^Ta_j)q_i,\quad r_{jj}=\|u_j\|_2,\quad q_j=u_j/r_{jj}.$$

For least squares, $\min_x\|Ax-b\|_2$ becomes $\min_x\|Rx-Q^Tb\|_2$; solve the triangular system using the first $n$ entries of $Q^Tb$.

### Cholesky factorization

For symmetric positive definite $B$, $B=LL^T$, with $L$ lower triangular. If

$$B=\begin{bmatrix}4&2\\2&3\end{bmatrix},$$

then

$$L=\begin{bmatrix}2&0\\1&\sqrt2\end{bmatrix},\qquad B=LL^T.$$

For general lower-triangular $L$:

$$l_{ii}=\sqrt{b_{ii}-\sum_{k<i}l_{ik}^2},\qquad l_{ji}=\frac{b_{ji}-\sum_{k<i}l_{jk}l_{ik}}{l_{ii}}\quad(j>i).$$

Cholesky is efficient for PD systems; it is not applicable to an arbitrary non-PD matrix.

## 8. Orthogonal Procrustes and nearest orthogonal matrix

### Orthogonal Procrustes

Given $X,Y\in\mathbb R^{d\times N}$, solve

$$\min_{R^TR=I}\|X-RY\|_F^2.$$

Expand the square:

$$\|X-RY\|_F^2=\|X\|_F^2+\|Y\|_F^2-2\operatorname{tr}(R^TC),\qquad C=XY^T.$$

Thus minimize the distance by maximizing $\operatorname{tr}(R^TC)$. If $C=U\Sigma V^T$, then

$$\boxed{\hat R=UV^T}$$

is the optimal orthogonal matrix when reflections are allowed.

If a **proper rotation only** is required ($\det R=+1$), use

$$\hat R=UDV^T,\qquad D=\operatorname{diag}(1,\ldots,1,\det(UV^T)).$$

### Nearest orthogonal matrix to $A$

To solve $\min_{R^TR=I}\|A-R\|_F$, take $A=U\Sigma V^T$ and use $R^*=UV^T$. The minimum squared distance is $\sum_i(\sigma_i-1)^2$ for square $A$.

## 9. Similarity Procrustes (rotation + scale + translation)

Given corresponding points stored as columns,

$$P_1\approx \alpha RP_2+t\mathbf1^T,$$

where $\alpha\ge0$, $R^TR=I$, and $\mathbf1$ is an all-ones vector.

1. **Centroids:** $\bar p_1=P_1\mathbf1/N$, $\bar p_2=P_2\mathbf1/N$.
2. **Center:** $X=P_1-\bar p_1\mathbf1^T$, $Y=P_2-\bar p_2\mathbf1^T$.
3. **Cross-covariance:** $C=XY^T=U\Sigma V^T$.
4. **Rotation:** $\hat R=UV^T$ if reflections are allowed; apply determinant correction if only proper rotations are allowed.
5. **Scale (reflections allowed):** $\displaystyle \hat\alpha=\frac{\operatorname{tr}(\Sigma)}{\|Y\|_F^2}$.
6. **Translation:** $\hat t=\bar p_1-\hat\alpha\hat R\bar p_2$.

If using a proper-rotation correction $D$, replace $\operatorname{tr}(\Sigma)$ in the scale formula with $\operatorname{tr}(D\Sigma)$.

**Why center first?** Translation is absorbed by the centroids, leaving a rotation/scale problem with zero-mean point sets. This separates translation from the remaining parameters.

## 10. PCA and covariance

Let $X\in\mathbb R^{d\times N}$ contain $N$ samples as columns. First subtract the sample mean from each column, so the data are centered.

- Covariance (one common convention): $C=XX^T/N$; some courses use $1/(N-1)$ instead.
- $C$ is symmetric PSD because $z^TCz=\|X^Tz\|_2^2/N\ge0$.
- If $Cv_i=\lambda_i v_i$ with $\|v_i\|=1$, then $\lambda_i$ is the variance along direction $v_i$.
- Sort eigenvalues descending; the top eigenvectors are the principal directions with the largest projected variances.
- Keeping the first $k$ eigenvectors gives the rank-$k$ PCA representation/projection; the explained-variance fraction is $\sum_{i=1}^k\lambda_i/\sum_i\lambda_i$.
- Since $C=XX^T/N$, PCA directions are the left singular vectors of centered $X$, and covariance eigenvalues are $\sigma_i^2/N$.
- If $d\gg N$, solve the smaller $N\times N$ Gram eigenproblem $X^TXv=\sigma^2v$ and recover $u=Xv/\sigma$ for $\sigma>0$.

## 11. Affine transforms, calibration and image interpolation

### Affine transform in homogeneous coordinates

For column-vector convention,

$$\begin{bmatrix}x'\\y'\\1\end{bmatrix}=H\begin{bmatrix}x\\y\\1\end{bmatrix},\qquad H=\begin{bmatrix}a&b&t_x\\c&d&t_y\\0&0&1\end{bmatrix}.$$

Three non-collinear point correspondences are the minimum needed to determine a 2D affine map in the noiseless case. With more points, estimate it by least squares. If the source design matrix is rank-deficient (e.g. the points are collinear), the affine parameters are not uniquely determined.

For graph/image-coordinate calibration using row vectors $M_i=[x_i^{(\mathrm{pixel})},y_i^{(\mathrm{pixel})},1]$ and target rows $G_i=[x_i^{(\mathrm{graph})},y_i^{(\mathrm{graph})}]$, fit $MA\approx G$ by

$$\hat A=(M^TM)^{-1}M^TG$$

when $M^TM$ is invertible. Check the row/column convention before using a formula; transposing the data changes the correct multiplication order.

### Reverse warping and bilinear interpolation

To fill every destination pixel, map destination coordinates backward through the inverse transform into the source image. If source coordinate $(x,y)$ is non-integer, let $x_0=\lfloor x\rfloor$, $y_0=\lfloor y\rfloor$, $d_x=x-x_0$, $d_y=y-y_0$. Then

$$\begin{aligned}
f(x,y)={}&(1-d_x)(1-d_y)f(x_0,y_0)\\
&+d_x(1-d_y)f(x_0+1,y_0)\\
&+(1-d_x)d_yf(x_0,y_0+1)\\
&+d_xd_yf(x_0+1,y_0+1).
\end{aligned}$$

Handle mapped coordinates outside the source image separately (e.g. mark them invalid or fill with a chosen background value).

## 12. Fast proof templates

### Show $A^TA$ is positive definite

1. State $A$ has full column rank, so $\mathcal N(A)=\{0\}$.
2. For $x\ne0$, $x^TA^TAx=\|Ax\|_2^2$.
3. Since $Ax\ne0$, this is strictly positive; also $(A^TA)^T=A^TA$. Conclude PD.

### Show columns are dependent

Give a nonzero coefficient vector $c$ with $Ac=0$ (equivalently, an explicit linear combination of columns equaling zero). Or show $\operatorname{rank}(A)<$ number of columns. For square matrices, $\det(A)=0$ is another route.

### Derive normal equations

Write $f(x)=\|Ax-b\|_2^2$, differentiate to get $2A^T(Ax-b)$, set to zero, and state the residual orthogonality $A^T(b-A\hat x)=0$.

### Show nearest orthogonal matrix / Procrustes solution

Expand the Frobenius norm, discard constant terms, turn minimization into a trace maximization, take the SVD of the cross matrix, and obtain $R=UV^T$ (with determinant correction if a proper rotation is required).

### Derive SVD from eigendecomposition

Diagonalize $A^TA=V\Lambda V^T$. Set $\sigma_i=\sqrt{\lambda_i}$ and, for $\sigma_i>0$, $u_i=Av_i/\sigma_i$. This yields $Av_i=\sigma_i u_i$ and aligns singular-vector signs.

## 13. Common traps to avoid

- Confusing $\|x\|$ with $\|x\|^2$: $x^Tx=\|x\|_2^2$.
- Forgetting to square-root a squared Frobenius norm proof at the end.
- Claiming $A^TA$ is PD without using **full column rank**.
- Using $\det(A)$ to determine rank for a non-square matrix.
- In power iteration, targeting the largest magnitude eigenvalue—not always the largest algebraic eigenvalue.
- Comparing eigenvectors without accounting for sign ($v$ and $-v$ are equivalent).
- Computing left and right singular vectors independently and mismatching their signs.
- Mixing row-point and column-point conventions in geometric transforms.
- Forgetting that a general orthogonal Procrustes solution may be a reflection; enforce $\det(R)=+1$ only when the question requires a rotation.
- Using $(A^TA)^{-1}A^T$ without checking full column rank.

## 14. Last-minute formula strip

$$\begin{gathered}
\|A\|_F^2=\operatorname{tr}(A^TA)=\sum_i\sigma_i^2,\qquad \|A\|_2=\sigma_1,\\
\operatorname{rank}(A)+\operatorname{nullity}(A)=n,\qquad A^TA\succ0\iff A\text{ has full column rank},\\
\hat x_{LS}=(A^TA)^{-1}A^Tb,\qquad A^T(A\hat x-b)=0,\\
H=I-2\frac{vv^T}{v^Tv},\qquad H^T=H,\quad H^2=I,\\
A=U\Sigma V^T,\quad A^+=V_r\Sigma_r^{-1}U_r^T,\\
\kappa_2(A)=\frac{\sigma_{\max}}{\sigma_{\min}},\qquad \hat R_{Proc}=UV^T,\\
\hat\alpha=\frac{\operatorname{tr}(\Sigma)}{\|Y\|_F^2},\qquad \hat t=\bar p_1-\hat\alpha\hat R\bar p_2.
\end{gathered}$$

