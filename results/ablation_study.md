# Ablation Study and Cross-Dataset Validation

Supplementary experiments conducted during development, retained for transparency.

## Overview

The main pipeline uses QGA feature selection on FBCSP features for 3-class
motor-imagery classification. This document summarizes every alternative
method tested and every dataset the pipeline was evaluated on. Positive
results and negative results are reported with equal weight.

## Test Configuration

- Primary dataset: PhysioNetMI Subject 1, 129 trials, 3 classes (left, right, rest)
- Secondary datasets: BCIC-IV-2a Subject 1, Cho2017 Subject 1, HGD Subject 1
- Split protocol: 80/20 stratified, `random_state=42`, applied before any
  preprocessing (leak-free)
- Classifier: balanced RBF SVM, 5-fold stratified CV for hyperparameter tuning

## 1. Quantum Feature Map Comparison (PhysioNetMI)

Methods that use a quantum circuit to produce features, before any selection.

| Method | Test Accuracy | Test Macro F1 | CV Macro F1 |
|---|---|---|---|
| Classical FBCSP (12 feat) + SVM | 0.7692 | 0.6216 | 0.8794 |
| Frozen quantum embedding (12q) + SVM | 0.5769 | 0.4601 | 0.5698 |
| Hybrid (classical + quantum, 24-dim) + SVM | 0.6923 | 0.5926 | 0.8791 |
| Variational quantum + MLP head | 0.6923 | 0.4889 | 0.6592 |
| MLP on quantum features | 0.5769 | 0.4353 | — |
| MLP on classical FBCSP features | 0.6538 | 0.5261 | 0.8337 |

**Conclusion:** Quantum feature maps (frozen, variational, hybrid) underperform
classical FBCSP when the input and output dimensions are equal. The random
unitary is a lossy projection at dimension parity.

## 2. Feature Selection Comparison (PhysioNetMI)

| Method | Selected | Test Accuracy | Test Macro F1 | CV Macro F1 |
|---|---|---|---|---|
| QGA | 7 of 12 | 0.7692 | 0.6476 | 0.9522 |
| MIBIF (mutual information) | 7 of 12 | 0.7308 | 0.6245 | 0.9180 |
| Full FBCSP (no selection) | 12 of 12 | 0.7308 | 0.6245 | 0.8983 |

**Conclusion:** QGA outperforms classical mutual-information selection at
matched subset size (7 features), and outperforms the full 12-feature set.

## 3. Alternative Feature Domains (PhysioNetMI)

| Method | Test Accuracy | Test Macro F1 | CV Macro F1 |
|---|---|---|---|
| FBCSP (spatial) | 0.7308 | 0.6245 | 0.8983 |
| Riemannian tangent space (geometric) | 0.7308 | 0.5786 | 0.6373 |
| Hjorth parameters (temporal) | 0.5769 | 0.4440 | 0.3813 |
| Full fusion (42 feat) | 0.7692 | 0.5333 | 0.8810 |
| QGA on fusion pool | 0.7308 | 0.5029 | 0.9297 |

**Conclusion:** Multi-domain fusion dilutes the signal at this sample size.
FBCSP alone remains the strongest single domain.

## 4. CSP Regularization (PhysioNetMI)

| Method | Test Accuracy | Test Macro F1 |
|---|---|---|
| Unregularized CSP | 0.7308 | 0.6245 |
| Ledoit-Wolf shrinkage CSP | 0.6923 | 0.5722 |

**Conclusion:** Ledoit-Wolf over-regularized the covariance, reducing spatial
discriminability on 103 training trials.

## 5. Quantum Kernel Methods (PhysioNetMI)

| Kernel | Test Accuracy | Test Macro F1 | CV Macro F1 |
|---|---|---|---|
| Classical RBF kernel | 0.6923 | 0.5926 | 0.8572 |
| Fidelity quantum kernel (12q) | 0.7308 | 0.6155 | 0.7036 |
| Ensemble quantum kernel (2 x 6q) | 0.7308 | 0.6245 | 0.8342 |

**Conclusion:** Quantum kernels are competitive with classical RBF. The
2 x 6-qubit ensemble mitigates kernel concentration (CV F1 0.8342 vs 0.7036
for the 12-qubit version), confirming the vanishing-similarity effect.

## 6. Cross-Dataset Validation

Same pipeline (FBCSP + QGA + SVM) evaluated on three additional datasets.

### BCIC-IV-2a Subject 1

| Task | Full FBCSP | QGA-Selected | Winner |
|---|---|---|---|
| 4-class (chance = 0.25) | 0.4921 macro F1 | 0.4353 macro F1 | Full |
| 2-class (chance = 0.50) | 0.6535 macro F1 | 0.6548 macro F1 | QGA |

### Cho2017 Subject 1 (2-class)

| Method | Test Accuracy | Test Macro F1 |
|---|---|---|
| Full FBCSP | 0.8000 | 0.7980 |
| QGA-selected | 0.7250 | 0.7248 |

**Winner:** Full FBCSP. QGA loses on Cho2017 by 7.3 macro F1 points.

### High Gamma Dataset Subject 1 (3-class, 5-band filterbank)

| Method | Test Accuracy | Test Macro F1 |
|---|---|---|
| Full FBCSP (20 feat) | 0.6111 | 0.6116 |
| QGA-selected (9 feat) | 0.6389 | 0.6395 |

**Winner:** QGA. Adding high-gamma bands (60-125 Hz) to the filterbank
changes the outcome on HGD, where low-frequency bands alone were not
discriminative.

## Overall QGA Scoreboard

| Dataset | Task | QGA vs Full FBCSP | Winner |
|---|---|---|---|
| PhysioNetMI | 3-class | 0.6476 vs 0.6245 | **QGA** |
| BCIC-IV-2a | 2-class | 0.6548 vs 0.6535 | **QGA** |
| BCIC-IV-2a | 4-class | 0.4353 vs 0.4921 | Full |
| Cho2017 | 2-class | 0.7248 vs 0.7980 | Full |
| HGD | 3-class | 0.6395 vs 0.6116 | **QGA** |

**QGA wins 3 of 5 tasks.** Its advantage is task-dependent: it helps when
class structure aligns with the FBCSP feature space and hurts on harder
classification problems.

## Statistical Caveat

McNemar's exact test on the primary PhysioNetMI result yields p = 1.0000
(b = 1, c = 0), so the QGA improvement over full FBCSP is not statistically
significant at alpha = 0.05 on 26 test samples. Bootstrap 95% confidence
intervals overlap substantially. The CV macro F1 (0.9522 vs 0.8983) uses all
103 training samples across 5 folds and provides a more stable estimate.

## Why These Are Not in the Main Notebook

The main notebook (`QGA_for_FBCSP_feature_selection_Final.ipynb`) contains
the winning pipeline end-to-end with full reproducibility. The experiments
above were conducted during development to select that pipeline. They are
reported here for transparency and to bound the applicability of the method.
