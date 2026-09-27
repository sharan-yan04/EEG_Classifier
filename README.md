# EEG Motor Imagery Classification with Quantum Genetic Algorithm Feature Selection

Full pipeline for 3-class motor imagery EEG classification
(left hand, right hand, rest) based on Filter Bank Common Spatial Patterns (FBCSP)
with feature subset selection by Quantum Genetic Algorithm (QGA).

## Key Findings

- **76.92% test accuracy** and **0.6476 macro F1** on PhysioNetMI Subject 1
- QGA selects **7 of 12 FBCSP features** but beats the performance of the full 12-features baseline
- No leaks: CSP, scikit-learn scaler, and QGA fitness are fitted exclusively on the training set
- Rigorous statistics: McNemar's test and bootstrap 95% confidence intervals provided
- Gradio interactive demo with balanced sampling and manually entered EEG

## Architecture

![Pipeline Architecture](figures/Architecture.png)

**Architecture summary:**

1. Raw EEG from PhysioNetMI Subject 1 (64 channels, 160 Hz, 4 sec trials)
2. Three-band Butterworth filter bank: Mu (8-12 Hz), Beta (13-30 Hz), Gamma (30-40 Hz)
3. Filter Bank CSP: 4 components for each band => 12 features per trial
4. StandardScaler fit on train fold alone (no leakage)
5. **Quantum Genetic Algorithm** determines the optimal subset (new addition to the literature)
6. Balanced RBF SVM classifier for left hand, right hand, or rest


## Results

| Algorithm | Accuracy on test set | Test Macro-F1 | Cross validation macro-F1 |
|---|---|---|---|
| Majority class baseline | 0.6538 | — | — |
| Fully functional FBCSP (12 features) + SVM | 0.7308 | 0.6245 | 0.8983 |
| **QGA selected (7 features) + SVM** | **0.7692** | **0.6476** | **0.9522** |

**Indices of selected features:** [1, 2, 4, 6, 7, 8, 9]

### Comparison Chart

![Classical vs QGA](figures/classical_vs_qga.png)

### Confusion Matrices

![Confusion Matrices](figures/confusion_matrices.png)


## Novelty

- Application of QGA to **components of FBCSP** (not channel selection, not raw features)
- Filterbank using three bands creating a 12-dimensional representation in space and frequency
- Fully leak-proof pre-processing and evaluation procedure
- Transparent statistics (McNemar p = 1.0000, overlapping bootstrap confidence intervals)
- Demo application using balanced classes and user interface

## Demo

The Gradio interface supports pre-staged test sample evaluation and manual EEG trial upload.

### Left Hand Predictions

Correct classifications:
![Left Hand Prediction 1](screenshots/Left-1.png)
![Left Hand Prediction 2](screenshots/Left-2.png)

Incorrect classification (misclassified as Rest):
![Left Hand Incorrect](screenshots/Left-Inc.png)

### Right Hand Predictions

Correct classifications:
![Right Hand Prediction 1](screenshots/Right-1.png)
![Right Hand Prediction 2](screenshots/Right-2.png)

Incorrect classification (misclassified as Rest):
![Right Hand Incorrect](screenshots/Right-Inc.png)

### Rest Predictions

Correct classifications:
![Rest Prediction 1](screenshots/Rest-1.png)
![Rest Prediction 2](screenshots/Rest-2.png)

Incorrect classification:
![Rest Incorrect](screenshots/Rest-Inc.png)
### Manual EEG Input Tab

![Manual Input](screenshots/gradio_manual_input.png)

### Model Information

![Model Info](screenshots/gradio_model_info.png)

## Installation

### Environment

- Google Colab (T4 GPU recommended, CPU works)
- Python 3.13
- All dependencies listed in `requirements.txt`

### Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/<your-username>/eeg-qga-fbcsp-classifier.git
   cd eeg-qga-fbcsp-classifier
