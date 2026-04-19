# COMPAS Adversarial Security Audit

**DNSC 6330: Responsible Machine Learning**  
Individual Homework 5

> **Generative AI Disclosure:** Generative AI tools were used as a learning aid during the development of this work — specifically for brainstorming code structure, debugging implementations, and reviewing outputs for accuracy. All AI-generated content was critically reviewed, validated, and integrated as the author's own intellectual product. This disclosure is made in accordance with GW's Generative AI Use Policy.

## Purpose

This repository contains the Individual Homework 5 submission for DNSC 6330, extending the Lecture 05 live-coding lab on ML security. It implements an applied adversarial audit of two classifiers trained on the ProPublica COMPAS recidivism dataset — a Logistic Regression (LR) and a Gradient Boosted Tree (GBT) — across three attack vectors and a structured reflection.

The workflow covers:

1. **Baseline setup** — Replicate the Lecture 04/05 data pipeline; establish clean-model FPR and AIR by race before any attack.
2. **Task 1: PGD Evasion Audit** — Run Projected Gradient Descent (Madry et al., 2018) on both LR (exact gradient via sign(**w**)) and GBT (vectorised finite-difference gradient estimation) for ε ∈ {0.25, 0.5, 1.0, 2.0}. Report FPR by race, AIR, and ε at which AIR crosses 0.80 for each model. Visualize LR vs. GBT on the same axes. Write a paragraph on comparative model vulnerability.
3. **Task 2: Poisoning Loop with Fairness Monitoring** — Extend the Lecture 05 label-flip loop to target both African-American and Caucasian defendants. Plot AUC and AIR degradation for both race variants on the same axes. Identify the stealth zone (AUC drop ≤ 2pp while AIR outside [0.80, 1.25]). Implement PSI on input features and model output scores; assess whether a feature-based drift monitor would detect either attack.
4. **Task 3: Membership Inference (Depth)** — Run Shokri et al. (2017) shadow-model MI attack on both GBT and LR. Plot side-by-side confidence-gap histograms. Test whether generalization gap predicts MI AUC. Sweep L2 regularization (C ∈ {0.01, 0.1, 1.0, 10.0}) and plot MI AUC vs. C; discuss the privacy–utility tradeoff.
5. **Task 4: Reflection** — Identify the single highest-risk finding (GBT membership inference vulnerability); propose one proactive mitigation (regularize/substitute LR) and one reactive mitigation (disaggregated output score PSI monitoring); quantify effects using experimental results; discuss disparate impact on racial groups.

## Python Libraries Used

| Library | Purpose |
|---|---|
| `pandas` | Data manipulation and analysis |
| `numpy` | Numerical computations |
| `matplotlib` | Data visualization |
| `scikit-learn` | Preprocessing, LogisticRegression, GradientBoostingClassifier, DecisionTreeClassifier, metrics |

No external adversarial ML library is required — all attack pipelines are implemented from scratch using standard scikit-learn and numpy.

## Reproducing the Results

### Prerequisites

- Python 3.9 or later
- `pip` package manager
- Internet connection (the notebook downloads the dataset automatically)

### Steps

1. **Clone or download this repository:**
   ```bash
   git clone <repository-url>
   cd <repository-name>/HW5
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run the Jupyter Notebook:**
   ```bash
   jupyter notebook DNSC-6330-Responsible-ML-homework-05.ipynb
   ```
   Alternatively, upload to Google Colab and run all cells sequentially
   (Runtime → Run all).

4. **Data source:** The notebook downloads the dataset automatically from the
   ProPublica GitHub repository — no manual download is required.

> **Note on runtime:** GBT training with 200 estimators (~30s) plus the finite-difference
> PGD attack on GBT (20 iterations × 4 epsilon values × ~1.5s per call ≈ 2–3 min), the
> shadow model pipelines (20 shadow models total, ~2 min), and the L2 sweep (4 C values
> × 10 shadow models each ≈ 3–5 min). Total expected runtime: **10–15 minutes** on a
> standard laptop CPU. Google Colab GPU/CPU is recommended for faster execution.

## Repository Structure

```
HW5/
├── README.md                                      # This file
├── requirements.txt                               # Python dependencies
├── DNSC-6330-Responsible-ML-homework-05.ipynb    # Main homework notebook
├── Lecture_05_security.ipynb                     # Original lecture lab (reference)
├── DNSC_6330_Lecture-05.pdf                      # Lecture slides and assignment spec
└── homework_checklist.md                         # Self-audit checklist
```

## Data Source

Broward County COMPAS scores and two-year recidivism outcomes from the
[ProPublica Machine Bias investigation](https://www.propublica.org/article/machine-bias-risk-assessments-in-criminal-sentencing) (2016).

- Dataset: [`compas-scores-two-years.csv`](https://raw.githubusercontent.com/propublica/compas-analysis/master/compas-scores-two-years.csv)
- Original R analysis: [ProPublica/compas-analysis](https://github.com/propublica/compas-analysis)

## Key Findings Summary

- **Task 1 (PGD):** Both LR and GBT begin with pre-existing racial disparity (AIR > 1.5 at ε = 0). LR's AIR remains stable under white-box PGD because the uniform sign(**w**) perturbation affects both racial groups proportionally. GBT's finite-difference PGD results are reported in the notebook. Neither model's AIR crosses 0.80 within the tested epsilon range.

- **Task 2 (Poisoning):** AA-targeted label-flip poisoning drives AIR higher (above 1.25); CA-targeted poisoning reduces Caucasian FPR, also pushing AIR upward. A feature-based PSI monitor is completely blind to label-flip attacks (feature distributions are unchanged; all PSI = 0.0). Model output score PSI provides partial detection at higher poison rates.

- **Task 3 (MI):** The GBT's large generalization gap (+0.080: Train AUC 0.798 vs. Test AUC 0.718) produces significantly higher MI AUC than the LR, which generalizes consistently. L2 regularization on LR compresses the generalization gap and reduces MI AUC toward 0.50, with C = 0.1 offering a strong privacy–utility tradeoff.

- **Task 4 (Reflection):** GBT membership inference is the single highest risk. Proactive mitigation: substitute LR (or heavily regularize GBT). Reactive mitigation: disaggregated output score PSI monitoring by race group after each retraining cycle.

## References

- Vassilev et al. (2024). *NIST AI 100-2e2023.* https://doi.org/10.6028/NIST.AI.100-2e2023
- Madry et al. (2018). *Towards Deep Learning Models Resistant to Adversarial Attacks.* ICLR 2018.
- Shokri et al. (2017). *Membership Inference Attacks Against Machine Learning Models.* IEEE S&P.
- ProPublica (2016). *Machine Bias.*
