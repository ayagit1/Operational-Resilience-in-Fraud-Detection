# Operational Resilience in Fraud Detection
Master’s Thesis — CY Tech / CY Cergy Paris Université

[![License](https://img.shields.io/badge/license-Academic%20Use-blue.svg)](#license)
[![Python](https://img.shields.io/badge/python-3.8%2B-green.svg)](#how-to-run)

A research codebase exploring fraud detection as an operational decision problem under realistic constraints:
delayed verification, limited investigator capacity (Top-K), value asymmetry, and adversarial drift. The
project emphasizes business-aligned evaluation (Net Value / ROI) rather than only classical ML metrics.

---

## Table of Contents

- [Overview](#overview)
- [Problem Framing](#problem-framing)
- [Dataset](#dataset)
- [Models & Methods](#models--methods)
- [Operational Metrics](#operational-metrics)
- [Experiments](#experiments)
- [Key Takeaways](#key-takeaways)
- [How to Run](#how-to-run)
  - [Option A — Google Colab (recommended)](#option-a---google-colab-recommended)
  - [Option B — Local environment](#option-b---local-environment)
- [Reproducibility Notes](#reproducibility-notes)
- [Author](#author)
- [License](#license)

---

## Overview

This repository contains the work performed for the Master’s thesis by Aya Amarass. The main artifact is the notebook:

- `Operational_Resilience_in_Fraud_Detection.ipynb` — includes data preprocessing, simulation setup (pseudo-days), rolling evaluation, evaluation metrics (Precision@K, Dollar-Precision@K, Net Value), and Experiments 1–6 with plots.

The aim is to simulate a realistic production loop for fraud alerting and to evaluate models and policies using business metrics that capture profitability under constraints.

---

## Problem Framing

In production, fraud detection is not simply "predict fraud vs non-fraud". The real question is:

Which transactions should we investigate today, given that:
- investigations cost money,
- labels (true fraud confirmations) arrive with delay,
- investigators can only review a limited number of alerts per day,
- transaction values vary and fraud amounts are asymmetric,
- and fraud patterns may shift over time?

The repository simulates the operational loop:

1. Transactions arrive daily.
2. A model produces fraud scores.
3. A selection policy chooses alerts to investigate (Top-K or gated).
4. Verified labels arrive later (label delay).
5. The model is periodically retrained using the most recently verified window.

---

## Dataset

This project uses the widely used credit card fraud dataset (Dal Pozzolo / ULB benchmark) available on Kaggle:

- Kaggle dataset: [Credit Card Fraud Detection by ULB / Dal Pozzolo](https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud)

Key points:
- Highly imbalanced (fraud is rare).
- Features: anonymized PCA components `V1…V28`, plus `Time` and `Amount`.
- Target: `Class` (0 = normal, 1 = fraud).
- The original dataset spans a short time window. The notebook converts `Time` into pseudo-days (e.g., ~30 days) to simulate rolling retraining, label delay, and drift.

---

## Models / Methods

Implemented models and variations:

- Logistic Regression (LR) — fast baseline / screening model.
- XGBoost (XGB) — stronger non-linear classifier / ranking model.
- Amount/value-aware weighting and reranking variants.
- Policy layer experiments including cost-aware gating (FACL).
- Active learning / selection strategies under limited investigator budgets.

---

## Operational Metrics (Business-aligned)

The evaluation is focused on operational outcomes rather than only ML scores:

- Precision@K
  - Fraction of true frauds among the top-K investigated alerts.
- Dollar-Precision@K (DP@K)
  - Measures how much of the total investigated transaction value is fraud (money-weighted precision).
- Net Value (primary KPI)
  - A simplified daily profit metric:

  $$
  \text{NetValue} = \alpha \cdot \text{FraudEUR}_{\\text{selected}} - c \cdot N_{\\text{selected}}
  $$

  Where:
  - α = assumed recovery rate (e.g., 0.70),
  - c = investigation cost per alert (e.g., €10),
  - N_selected = number of alerts investigated that day.

Net Value directly measures the economic impact of an alerting policy and model under capacity constraints.

---

## Experiments

The notebook is structured into a progression of experiments that introduce operational realism step by step:

- **Experiment 1 — Single-day baseline comparison (snapshot)**  
  Compare LR vs XGB variants on a single day (e.g., Day 10) using top-K evaluation.

- **Experiment 2 — Rolling evaluation (retraining with time)**  
  Simulate daily operation using a rolling train window (e.g., last 10 pseudo-days) and evaluate performance stability.

- **Experiment 2b — Value-aware reranking**  
  Explore reranking that incorporates transaction amount/value to improve ROI-oriented metrics.

- **Experiment 3 — Label delay (e.g., 3-day delay)**  
  Train models using only labels available up to day d - delay and evaluate how delay impacts top-K and Net Value.

- **Experiment 4 — Weak proxy labels (weak supervision)**  
  Evaluate the risk of using heuristic proxy labels as ground truth and how they can bias models and degrade ROI.

- **Experiment 5 — Active learning (cost-aware variants)**  
  Simulate selection strategies (greedy / uncertainty / random) under limited investigator budgets; measure ROI impact.

- **Experiment 6 — Cost-Aware Gating (FACL)**  
  Add a gating layer that selects alerts based on expected financial value and throttles investigations when expected ROI is low (especially effective under drift).

---

## Key Takeaways (High-level)

- Operational metrics matter: global ML metrics can appear good while ROI is poor.
- Proxy labels can be dangerous: heuristics are useful as features but not as replacement ground truth.
- Cost-aware gating and value-aware policies can improve resilience and profitability under drift and weak signals.

---

## How to Run

### Option A — Google Colab (recommended)

1. Open the notebook in Colab.
2. Run all cells.
3. Ensure you have Kaggle credentials configured if the notebook downloads the dataset from Kaggle (the notebook uses `kagglehub`).

### Option B — Local environment

1) Create a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\\Scripts\\activate
pip install -U pip
```

2) Install dependencies:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn xgboost kagglehub notebook
```

3) Launch Jupyter and open the notebook:

```bash
jupyter notebook
# then open Operational_Resilience_in_Fraud_Detection.ipynb
```

---

## Reproducibility Notes

- Results may vary due to:
  - random seeds,
  - XGBoost version and hyperparameters,
  - environment differences.
- For stricter reproducibility:
  - set explicit random seeds in the notebook,
  - pin package versions and log them (e.g., in a `requirements.txt` or using `pip freeze`).

---

## Author

Aya Amarass  
Master’s student — CY Cergy Paris Université (CY Tech), Cergy, France  
Research topic: Operational resilience in fraud detection (delay, drift, budget constraints, ROI)  
GitHub: [ayagit1](https://github.com/ayagit1)

---

## License

This repository contains research work developed as part of a Master’s thesis at CY Cergy Paris Université (CY Tech). The code, experiments, and results are intended for academic, educational, and research purposes only.

Commercial use, deployment in production systems, or redistribution for profit is not permitted without explicit permission from the author. Proper citation is required if this work is reused in academic publications, reports, or derivative research.

© 2025 — Aya Amarass
