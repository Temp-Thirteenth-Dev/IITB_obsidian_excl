# The Goal 
Build a system that learns which image patches are important for classification, using a differentiable top-$k$ selection mechanism based on Sinkhorn normalization

# Roadmap

### 1. Understand the Sinkhorn paper

> Foundation

Learn how a nonnegative matrix is repeatedly scaled so its rows and columns sum to one. Understand doubly stochastic matrices and when the iteration converges.

Deliverable: a working Sinkhorn implementation and numerical tests.

### 2. Implement differentiable top-\(k\)

> Core algorithm

Study the SOFT top-\(k\) paper. Implement its optimal-transport-based relaxation using PyTorch, initially on small synthetic score vectors.

Deliverable: a selection operator that produces a differentiable approximation to selecting the largest \(k\) scores.

### 3. Build the image-patch pipeline

> Computer vision

Use CIFAR-10, divide images into patches, score their importance, select patches, and feed the selected or weighted patches to a small CNN classifier.

Deliverable: an end-to-end model with a patch-scoring network.

### 4. Run experiments and compare

> Compare random selection, hard top-\(k\), and Sinkhorn-based differentiable top-\(k\). Measure accuracy, gradient flow, runtime, and sensitivity to hyperparameters.

Deliverable: results, plots, visualizations, and an analysis of what works and what doesn't.



---
