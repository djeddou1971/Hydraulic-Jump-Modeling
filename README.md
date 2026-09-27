# Hydraulic Jump Modeling — Cascaded Physics-Informed Neural Network with Multi-Head Attention (C-PINN-MHA)

This directory contains the code, data, and results for a physics-informed machine
learning study of the classical **hydraulic jump** problem. The goal is to predict
three dimensionless jump characteristics from two dimensionless inputs, using a
cascaded, attention-augmented physics-informed neural network (C-PINN-MHA) and
comparing it against a plain DNN and several classical ML baselines.

## Problem setup

| Symbol | Meaning | Role |
|---|---|---|
| `Fr1` | Upstream Froude number | Input |
| `h1` (`h1_cm`) | Upstream flow depth | Input |
| `γ` (gamma) | Compacity / conjugacy ratio, `γ = Lr*/x` | Input (physics-derived, used for conditioning) |
| `Y = h2/h1` | Conjugate (downstream) depth ratio | Output |
| `S = s/h1` | Jump roller/surface length ratio | Output |
| `X = x/h1` | Jump length ratio | Output |

Dataset: N = 151 experimental points, fixed 114/37 train/test split, seed = 42.

## Directory contents

```
Hydraulic-Jump-Modeling/
├── README.md                          — this file
├── RESULTS.md                         — narrative summary of all reported metrics
├── notebooks/
│   └── Hydraulic_Jump_Prediction_using_Cascaded_Physics-Informed_Neural_Network_with_Multi-Head_Attention_12-09-2026.ipynb
│                                       — full reproduction pipeline (data prep, DNN /
│                                         gamma-free / C-PINN-Att training, CV, ablations,
│                                         benchmarking, UQ, applicability domain, all figures)
├── data/
│   └── CPINN_Att_Data_Transformed_CORRECTED_AUDIT.xlsx
│                                       — audited/transformed input dataset used for training
├── results/
│   ├── CPINN_Att_Results.xlsx         — tabulated results (all tables, spreadsheet form)
│   ├── results.json                   — same results in structured JSON (tables 1–8,
│   │                                     t-tests, calibration, UQ, applicability domain)
│   ├── fixed_split_predictions.npz    — raw observed/predicted arrays for the fixed
│   │                                     114/37 split (numpy archive)
│   └── PARAMETERS--RESULTS.txt        — plain-text console log of the full run
└── figures/
    ├── Figure02.png  — input/output probability density distributions
    ├── Figure03.png  — Pearson correlation matrix
    ├── Figure06.png  — the compacity ratio γ as governing variable
    ├── Figure12.png  — per-fold R² boxplots, C-PINN-Att (5-fold CV)
    ├── Figure13.png  — γ-conditioning ablation (paired, 5 folds)
    ├── Figure16.png  — observed vs predicted, C-PINN-Att (fixed split)
    ├── Figure17.png  — DNN vs C-PINN-Att comparison (same test set)
    ├── Figure18.png  — observed vs predicted with 1:1 and regression lines
    ├── Figure19.png  — residuals vs predicted (fixed split)
    └── Figure21.png  — training-loss evolution (C-PINN-Att, fixed split)
```

## Models compared

- **DNN** — standard feed-forward baseline (no physics terms)
- **gamma_free** — physics-informed network without γ-conditioning
- **CPINN / C-PINN-Att** — cascaded, γ-conditioned, physics-informed network with
  multi-head attention (final proposed model)
- Classical baselines: **ANFIS, XGBoost, Random Forest, MLP (no physics)**

See [`RESULTS.md`](./RESULTS.md) for the full numeric summary and how to interpret
each figure and table.

## Reproducing

Open the notebook in `notebooks/` — it is self-contained and, when run top to
bottom, regenerates every table in `results/` and every figure in `figures/`
(the notebook writes them to an `OUTPUT_DIR/figures/*.png` at its final
"MAIN EXECUTION" section).
