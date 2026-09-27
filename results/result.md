# Results Summary

## Primary Result (PhysioNetMI Subject 1, 3-class)

| Method | Test Accuracy | Test Macro F1 | CV Macro F1 |
|---|---|---|---|
| Majority-class baseline | 0.6538 | — | — |
| Full FBCSP (12 features) + SVM | 0.7308 | 0.6245 | 0.8983 |
| QGA-selected (7 features) + SVM | 0.7692 | 0.6476 | 0.9522 |

## Selected Feature Indices

`[1, 2, 4, 6, 7, 8, 9]`

Mapping (0-indexed, concatenated across bands):
- 0-3: mu CSP components
- 4-7: beta CSP components
- 8-11: gamma CSP components

## Statistical Testing

**McNemar's exact test:**
- Contingency table: [[6, 1], [0, 19]]
- Exact p-value: 1.0000
- Asymptotic p-value (with correction): 1.0000
- Conclusion: QGA vs Full FBCSP difference is NOT statistically significant at alpha = 0.05

**Bootstrap 95% Confidence Intervals (1000 resamples):**

| Method | Accuracy 95% CI | Macro F1 95% CI |
|---|---|---|
| Full FBCSP (12 feat) | [0.5385, 0.8846] | [0.4032, 0.8107] |
| QGA-selected (7 feat) | [0.5769, 0.9231] | [0.4157, 0.8352] |

## Hyperparameters

- CSP components per band: 4
- Number of bands: 3 (mu 8-12, beta 13-30, gamma 30-40 Hz)
- QGA population size: 20
- QGA generations: 30
- QGA rotation step: 0.05 * pi
- QGA minimum features: 4
- SVM kernel: RBF
- SVM C: 1.0 (best from grid search)
- SVM gamma: 'scale'
- SVM class_weight: 'balanced'
- Train/test split: 80/20 stratified, random_state=42

## Known Limitations

1. Single-subject study (Subject 1 only)
2. Small test set (26 trials); wide confidence intervals
3. Statistical difference between QGA and full FBCSP is not significant
4. QGA-selected subset may not generalize to other subjects without retraining
