# CS 740 Midsem — Formula Quick Reference

## Linear algebra

- $\|A\|_F^2=\sum_{i,j}a_{ij}^2=\operatorname{tr}(A^TA)=\sum_i\sigma_i^2$.
- $\|A\|_2=\sigma_{\max}(A)$; $\kappa_2(A)=\sigma_{\max}/\sigma_{\min}$.
- $\operatorname{tr}(XY)=\operatorname{tr}(YX)$; cyclic permutations inside a trace are allowed when dimensions match.
- Orthogonal $Q$: $Q^TQ=I$, $\|Qx\|=\|x\|$, $\|QA\|_F=\|A\|_F$, $\det Q=\pm1$.
- $A^TA$ is PSD; it is PD iff $A$ has full column rank.
- $\operatorname{rank}(A)+\operatorname{nullity}(A)=n$; $\dim\operatorname{Col}(A)=\operatorname{rank}(A)$.

## Least squares

$$\min_x\|Ax-b\|_2^2\Rightarrow A^TA\hat x=A^Tb,\quad \hat x=(A^TA)^{-1}A^Tb$$

Residual orthogonality: $A^T(b-A\hat x)=0$. For matrix fitting $AP_1\approx P_2$:

$$\hat A=P_2P_1^T(P_1P_1^T)^{-1}$$

(Requires $P_1P_1^T$ invertible.) Jacobian of $Ax$ w.r.t. $x$ is $A$.

## Householder and rank-one matrices

$$H=I-2\frac{vv^T}{v^Tv},\quad H^T=H,\quad H^2=I$$

$Hv=-v$; if $v^Tw=0$, then $Hw=w$. For $B=uv^T$: rank $1$, $Bx=u(v^Tx)$, nullspace $v^\perp$, and $u$ has eigenvalue $v^Tu$.

## Iteration and errors

- Power: $x_{k+1}=Ax_k/\|Ax_k\|$; targets dominant eigenvalue **by magnitude**.
- Rayleigh quotient: $\hat\lambda=x^TAx/(x^Tx)$.
- Inverse iteration near shift $\alpha$: solve $(A-\alpha I)y=x$, then normalize; targets eigenvalue closest to $\alpha$.
- Relative eigenvalue error: $|\hat\lambda-\lambda|/|\lambda|$.
- Eigenvector angle: $\cos^{-1}(|v^T\hat v|/(\|v\|\|\hat v\|))$ (radians; multiply by $180/\pi$ for degrees).

## SVD and pseudoinverse

$$A=U\Sigma V^T,\quad A^TA v_i=\sigma_i^2v_i,\quad u_i=Av_i/\sigma_i$$

$$A_k=\sum_{i=1}^k\sigma_i u_iv_i^T,\quad \|A-A_k\|_F=\sqrt{\sum_{i>k}\sigma_i^2},\quad \|A-A_k\|_2=\sigma_{k+1}$$

$$A^+=V_r\Sigma_r^{-1}U_r^T,\quad A^+A=V_rV_r^T,\quad AA^+=U_rU_r^T$$

$\kappa_2(A^TA)=\kappa_2(A)^2$. For singular values $10$ and $0.01$, $\kappa=1000$.

## QR and Cholesky

- QR: $A=QR$, $Q^TQ=I$; Gram–Schmidt $u_j=a_j-\sum_{i<j}(q_i^Ta_j)q_i$, $q_j=u_j/\|u_j\|$.
- Cholesky: SPD $B=LL^T$. Example $\begin{bmatrix}4&2\\2&3\end{bmatrix}=\begin{bmatrix}2&0\\1&\sqrt2\end{bmatrix}\begin{bmatrix}2&1\\0&\sqrt2\end{bmatrix}$.

## Procrustes and PCA

- Orthogonal Procrustes: minimize $\|X-RY\|_F^2$, set $C=XY^T=U\Sigma V^T$, then $R=UV^T$ (reflection allowed).
- Proper rotation: $R=UDV^T$, $D=\operatorname{diag}(1,\ldots,\det(UV^T))$.
- Similarity Procrustes: center $X=P_1-\bar p_1\mathbf1^T$, $Y=P_2-\bar p_2\mathbf1^T$; $C=XY^T=U\Sigma V^T$; $R=UV^T$; $\alpha=\operatorname{tr}(\Sigma)/\|Y\|_F^2$; $t=\bar p_1-\alpha R\bar p_2$.
- PCA: center data, $C=XX^T/N$; eigenvectors are PCs, eigenvalues are projected variances; keep largest eigenvalues.

## Geometry / image coordinates

- Column homogeneous coordinates: $[x',y',1]^T=H[x,y,1]^T$ with $H=\begin{bmatrix}a&b&t_x\\c&d&t_y\\0&0&1\end{bmatrix}$.
- Reverse warping maps destination pixel back using $H^{-1}$.
- Bilinear interpolation uses four surrounding pixels with weights $(1-d_x)(1-d_y)$, $d_x(1-d_y)$, $(1-d_x)d_y$, $d_xd_y$.

## Exam traps

1. $x^TA^TAx=\|Ax\|^2$, not $\|Ax\|$.
2. Prove full column rank before calling $A^TA$ PD or inverting it.
3. Non-square rank: use pivots/SVD, not determinant.
4. Power iteration uses largest absolute eigenvalue.
5. Eigenvector signs are arbitrary; use $|v^T\hat v|$ for angle error.
6. General Procrustes may return a reflection; correct the determinant only if a proper rotation is required.
