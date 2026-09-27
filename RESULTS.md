# Results Summary

Reproduction run: N = 151, split 114/37, epochs = 2000, seed = 42.
Full numbers are in `results/PARAMETERS--RESULTS.txt` and `results/results.json`;
this file is a narrative guide to them.

## 1. Dataset (Table 1, Figure 2)

- Inputs: `Fr1` (2.80–9.89), `h1_cm` (1.50–4.50 cm), `γ` (0.89–2.22)
- Outputs: `Y` (3.36–12.69), `S` (1.07–6.25), `X` (8.39–56.52)
- All variables are right-skewed to some degree; `X` has the widest relative
  spread (CV ≈ 44%).
- **Figure 2** shows the marginal density of each input/output.
- **Figure 3** shows the Pearson correlation matrix: `Fr1` correlates very
  strongly with all three outputs (r = 0.87–0.99), while `γ` correlates weakly
  or negatively with them (r = −0.02 to −0.39), consistent with `γ` acting as a
  *conditioning* variable rather than a direct predictor.

## 2. Role of γ (Section 3.2, Figure 6, Figure 13)

γ-conditioning is the central architectural idea: an activation `w(γ)` gates
how strongly the physics constraints act, decaying to ~0 above γ ≈ 0.7
(Figure 6a). A paired t-test across 5 folds shows conditioning on γ
significantly improves all three outputs relative to a γ-free version of the
same network:

| Output | R² (γ-conditioned) | R² (γ-free) | t | p |
|---|---|---|---|---|
| Y | 0.982 | 0.951 | 6.23 | 0.0034 ** |
| S | 0.965 | 0.854 | 4.72 | 0.0092 ** |
| X | 0.974 | 0.699 | 6.09 | 0.0037 ** |

The effect is largest for `X` (jump length), where the γ-free model collapses
in one fold to R² ≈ 0.51 (Figure 13, right panel).

## 3. Cross-validated performance (Table 4, Figure 12)

Stratified 5-fold CV, mean ± std:

| Model | R²(Y) | R²(S) | R²(X) | PVI (%) |
|---|---|---|---|---|
| DNN | 0.985 ± 0.005 | 0.978 ± 0.006 | 0.976 ± 0.012 | 1.3 ± 1.6 |
| gamma_free | 0.951 ± 0.014 | 0.854 ± 0.050 | 0.699 ± 0.097 | 2.6 ± 3.9 |
| **C-PINN-Att** | 0.982 ± 0.004 | 0.965 ± 0.005 | 0.974 ± 0.007 | 1.3 ± 1.6 |

PVI = physics-violation index (lower is better; fraction of predictions
breaking a physical constraint). Figure 12 shows C-PINN-Att's per-fold R²
distributions are tight and consistently above ~0.94 for every output.

**C-PINN-Att vs plain DNN (Section 3.3):** not significantly different for
Y and X (p = 0.23, 0.88), but the DNN is *significantly better* for S
(p = 0.0057). In other words, adding the physics/attention machinery does not
buy accuracy over a plain DNN on this dataset — its value is in
interpretability, physical consistency, and behavior outside the training
distribution (see §6 below), not raw R².

## 4. Per-γ-class performance (Table 5)

Accuracy degrades as γ grows (fewer, more extreme samples):

| γ class | n | R²(Y) | R²(S) | R²(X) |
|---|---|---|---|---|
| ≈1.00 | 8.4 | 0.979 | 0.957 | 0.989 |
| ≈1.15 | 6.4 | 0.983 | 0.958 | 0.957 |
| ≈1.30 | 5.4 | 0.924 | 0.944 | 0.963 |
| ≈1.45 | 3.4 | 0.944 | 0.884 | 0.695 |
| ≈1.60 | 3.4 | 0.965 | 0.880 | 0.745 |
| ≈1.93 | 3.2 | 0.870 | 0.473 | 0.451 |

`X` is the output most sensitive to sparse, high-γ data.

## 5. Progressive ablation (Table 6)

Adding components one at a time to the DNN baseline (fixed 114/37 split):

| Model | R² | RMSE | PVI (%) |
|---|---|---|---|
| DNN baseline | 0.976 | 0.77 | 5.4 |
| + C1, C2 constraints (PINN-A) | 0.977 | 0.71 | 8.1 |
| + C8 constraint (PINN-B) | 0.978 | 0.60 | **0.0** |
| + C9 constraint (PINN-C) | 0.972 | 0.81 | 0.0 |
| + Attention (PINN-Att) | 0.969 | 0.87 | 0.0 |
| + Cascade (C-PINN-Att, full model) | 0.967 | 0.81 | 0.0 |

Adding constraints C8/C9 eliminates physics violations entirely (PVI → 0%)
at a small, monotonic cost in raw R²/RMSE — the expected accuracy/physical-
consistency trade-off.

## 6. Fixed-split validation (Section 3.8, Figures 16–19)

| Output | R² | 95% CI | RMSE | NRMSE | MAE |
|---|---|---|---|---|---|
| Y | 0.990 | [0.981, 0.994] | 0.224 | 2.58% | 0.187 |
| S | 0.942 | [0.892, 0.967] | 0.290 | 5.84% | 0.242 |
| X | 0.970 | [0.944, 0.983] | 1.909 | 4.01% | 1.602 |

PVI on this split is 0%. Shapiro-Wilk tests do not reject normality of
residuals for any output (p > 0.07 in all three cases).

- **Figure 16** — observed vs predicted, colored by γ class.
- **Figure 17** — same test set, DNN (grey) vs C-PINN-Att (blue) overlaid.
- **Figure 18** — observed vs predicted with both the 1:1 line and the fitted
  regression line; fitted slopes are close to 1 (0.98, 0.95, 1.05).
- **Figure 19** — residuals vs predicted. `X` shows a mild negative trend
  (over-prediction at low X, under-prediction at high X); Y and S residuals
  are more evenly scattered.

MC-dropout coverage is markedly under-nominal (e.g., Y: 32% inside ±1σ vs a
nominal 68%), meaning the network's native dropout-based uncertainty
intervals are overconfident and should not be used as-is.

### Functional consistency against classical formulas (full dataset)

| Check | Result |
|---|---|
| C-1 Bélanger equation | mean deviation −8.3%, Spearman ρ = −0.63 (p ≈ 5.6×10⁻¹⁸) |
| C-2 Hager–Bremen–Kawagoshi | satisfied by construction (γ = Lr*/x) |
| C-3 Hager–Li | dataset lies outside the original calibration range (b = 0.295 m) |
| C-4 Forster–Skrinde | R² vs observed = −0.385, vs predicted = 0.036 (poor fit either way) |

This indicates the dataset is only partially consistent with some classical
empirical hydraulic-jump relations, motivating the data-driven approach.

## 7. Uncertainty quantification & applicability domain (Figure 21)

Deep ensemble (5 seeds) + MC-Dropout combined uncertainty:

| Output | mean σ | coverage ±1σ | coverage ±2σ | nominal |
|---|---|---|---|---|
| Y | 0.178 | 54.1% | 86.5% | 68.3 / 95.4% |
| S | 0.090 | 27.0% | 43.2% | 68.3 / 95.4% |
| X | 0.632 | 21.6% | 48.6% | 68.3 / 95.4% |

Combined ensemble+dropout uncertainty is better calibrated than dropout alone
for `Y`, but still substantially under-covers for `S` and `X`.

**PCA(2) applicability domain**: the first two principal components explain
85.5% of input variance (50.9% + 34.6%). 36/37 test points fall inside the
training convex hull; the single extrapolated point has relative errors of
4.2% (Y), 1.3% (S), 7.4% (X) — not dramatically worse than in-domain points,
but a larger out-of-domain test set would be needed to confirm robustness.

Figure 21 shows training converges within ~250 epochs; total/physics loss
plateau around 0.06–00.07 and 0.045 respectively, with the data-loss term
continuing to drift down slightly to ~0.015 by epoch 2000.

## 8. Multi-model benchmark (Table 7)

Fixed 114/37 split, averaged over seeds [42, 0, 1, 7, 123]:

| Model | R²(Y) | R²(S) | R²(X) | RMSE | MAE | PVI (%) | Time (s) |
|---|---|---|---|---|---|---|---|
| ANFIS | 0.982 | 0.979 | 0.797 | 1.811 | 1.562 | 8.1 | 34.7 |
| XGBoost | 0.980 | 0.954 | 0.978 | 0.732 | 0.601 | 10.8 | 0.6 |
| Random Forest | 0.981 | 0.958 | 0.984 | 0.645 | 0.502 | 0.0 | 3.8 |
| MLP (no physics) | 0.988 | 0.954 | 0.982 | 0.655 | 0.243 | 1.6 | 14.8 |

Takeaway: classical/ensemble ML methods (Random Forest, XGBoost) are
competitive with or better than the neural approaches on raw fit quality and
are far cheaper to train, but only C-PINN-Att achieves near-zero physics
violations *by design* while also modeling the γ-dependent behavior
explicitly — a property the tabular ML baselines have no mechanism to
guarantee.

## Bottom line

- γ-conditioning is essential (large, significant R² gains, §2).
- Physics constraints (C8/C9 in particular) essentially eliminate physical
  violations at a small accuracy cost (§5).
- C-PINN-Att is *not* more accurate than a plain DNN or tree ensembles on
  this dataset (§3, §8) — its contribution is enforcing physical consistency
  and explicit γ-dependent behavior, not raw predictive power.
- Native uncertainty estimates (MC-Dropout, and to a lesser extent the
  ensemble) are overconfident and need recalibration before being used for
  interval predictions (§7).
