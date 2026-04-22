# Network Intrusion Detection System

A machine learning pipeline for detecting network intrusions using statistical analysis and Naïve Bayes classification, built on the [KDD Cup 1999 dataset](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html).

## Overview

This project was developed across three milestones, progressing from exploratory data analysis to a fully implemented anomaly detection classifier. The goal is to classify network traffic as **normal** or **anomaly** based on 41 features extracted from raw TCP/IP connections.

## Project Structure

```
Intrusion-Detection/
├── Milestone_1.py          # Exploratory Data Analysis (EDA)
├── Milestone_2.py          # Statistical modelling & Z-score anomaly detection
├── Milestone_3.py          # Naïve Bayes classifiers
├── PCA_MS_3_EXTRA.py       # Extra: PCA dimensionality reduction + NB variants
├── Train_data.csv          # Training dataset
├── Test_data.csv           # Testing dataset
├── Milestone 1.pdf         # Milestone 1 report
├── Milestone 2&3.pdf       # Milestones 2 & 3 report
├── Random_MS_3_Report.pdf  # Additional Milestone 3 analysis report
└── Project Description.docx
```

## Dataset

The dataset contains 41 features per network connection record, including:

| Category | Features |
|---|---|
| Basic | `duration`, `protocol_type`, `service`, `flag`, `src_bytes`, `dst_bytes` |
| Content | `hot`, `num_failed_logins`, `logged_in`, `num_compromised`, `root_shell`, `su_attempted` |
| Traffic | `count`, `srv_count`, `serror_rate`, `rerror_rate`, `same_srv_rate`, `diff_srv_rate` |
| Host-based | `dst_host_count`, `dst_host_srv_count`, `dst_host_same_srv_rate`, etc. |
| **Label** | `class` — `normal` or `anomaly` |

The train/test split used throughout is **70% train / 30% test**.

## Milestones

### Milestone 1 — Exploratory Data Analysis (`Milestone_1.py`)

Performs a thorough statistical survey of the dataset:

- **Data profiling** — field names, data types, missing/infinite value detection, unique category counts
- **Descriptive statistics** — per-column max, min, mean, and variance
- **Quartile analysis** — per-column quartile breakdowns with statistics for each quartile
- **Distribution plots** — PDF (KDE) for continuous features, PMF (bar chart) for categorical features
- **CDF plots** — cumulative distribution for every feature
- **Conditional distributions** — PDF/PMF conditioned on attack class
- **Scatter plots** — random pairs of numeric features
- **Joint distributions** — joint PMF (heatmap) and joint PDF (KDE) for random column pairs
- **Conditional joint distributions** — joint PDFs/PMFs conditioned on each attack type
- **Correlation heatmap** — full correlation matrix across numeric features
- **Attack correlation** — fields most correlated with each attack type (one-hot encoded)

> Most plot calls are commented out by default to allow selective execution. Uncomment the desired function call to generate the corresponding plot. Note: `plot_joint_cond_pdf_pmf` has an estimated runtime of ~10 minutes.

### Milestone 2 — Statistical Modelling & Z-Score Detection (`Milestone_2.py`)

Introduces statistical modelling and a first anomaly detector:

**Z-Score Anomaly Detector (Task 1)**
- Correlates each numeric feature with the binary attack label to derive feature weights
- Computes a *weighted Z-score* per row across all valid features
- Classifies a connection as anomaly if the weighted score exceeds a threshold
- Threshold study: `{1.5, 2.0, 2.5, 3.0}` — threshold of **0.5** yielded the best accuracy and recall

**Distribution Fitting (Task 2)**
- Fits 21 SciPy distributions to each numeric feature using MSE minimisation
- Identifies the best-fitting distribution per column under three conditions: original, normal-only, anomaly-only
- Plots conditional PDFs with the best-fit curve overlaid (KDE + dashed best-fit line)
- Documents PMF for low-variance/categorical columns

**Performance Metrics**
```
performance_metrics(actual_labels, predictions)
→ Accuracy, Precision, Recall (computed from confusion matrix)
```

### Milestone 3 — Naïve Bayes Classifiers (`Milestone_3.py`)

Implements a custom Naïve Bayes classifier and benchmarks it against sklearn variants:

**Custom Naïve Bayes (Task 1)**
- Fits best-distribution parameters separately for anomaly-conditioned and normal-conditioned data
- At prediction time, computes log-PDF/PMF likelihoods for each feature under both conditions
- Classifies by comparing `P(anomaly | row)` vs `P(normal | row)`
- Uses a small epsilon (`1e-10`) to prevent log-zero errors

**sklearn Naïve Bayes Benchmark (Task 2)**
- One-hot encodes categorical features (shared encoder for train/test)
- Trains and evaluates three sklearn models: `GaussianNB`, `MultinomialNB`, `BernoulliNB`
- Reports accuracy, precision, and recall for each

### Extra — PCA + Naïve Bayes (`PCA_MS_3_EXTRA.py`)

Extends Milestone 3 with dimensionality reduction:
- Applies `StandardScaler` + `PCA` (default: 10 components) before classification
- Shifts PCA output to be non-negative for `MultinomialNB` compatibility
- Compares all three NB variants on PCA-reduced features

## Requirements

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn
```

## Usage

All scripts expect `Train_data.csv` (and `Test_data.csv` for some tasks) to be present in the **same directory** as the scripts.

```bash
# Run EDA (uncomment the desired plot calls first)
python Milestone_1.py

# Run statistical modelling & Z-score detector
python Milestone_2.py

# Run Naïve Bayes classifiers
python Milestone_3.py

# Run PCA + Naïve Bayes (extra)
python PCA_MS_3_EXTRA.py
```

> `Milestone_3.py` imports helper functions from `Milestone_2.py`, so both files must be in the same directory.

