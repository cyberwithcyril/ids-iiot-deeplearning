# IDS-IIoT-DeepLearning

**Deep Learning vs. Classical ML for Intrusion Detection in IIoT Networks**

> M.S. Cybersecurity & Digital Forensics — SUNY Albany | Spring 2026  
> Author: Cyril Thomas ([@cyberwithcyril](https://github.com/cyberwithcyril))

---

## Overview

This project benchmarks six machine learning models — three classical and three deep learning — for binary intrusion detection on Industrial Internet of Things (IIoT) network traffic. Models are trained and evaluated on the **DataSense: CIC IIoT Dataset 2025** published by the Canadian Institute for Cybersecurity (UNB).

The central research question: *Do deep learning architectures outperform classical ML for intrusion detection on window-aggregated IIoT network traffic features?*

**Short answer: Not on this dataset.** XGBoost outperformed all three deep learning models on every metric.

---

## Results

| Rank | Model | F1 | AUC-ROC | Precision | Recall | Accuracy |
|------|-------|----|---------|-----------|--------|----------|
| 1 | **XGBoost** | **0.9679** | **0.9914** | 0.9890 | 0.9476 | 0.9753 |
| 2 | Random Forest | 0.9592 | 0.9866 | 0.9766 | 0.9424 | 0.9685 |
| 3 | MLP | 0.9554 | 0.9875 | 0.9844 | 0.9281 | 0.9660 |
| 4 | 1D-CNN | 0.9537 | 0.9867 | 0.9863 | 0.9232 | 0.9648 |
| 5 | BiLSTM | 0.9473 | 0.9815 | 0.9822 | 0.9149 | 0.9601 |
| 6 | Logistic Regression | 0.8933 | 0.9505 | 0.9550 | 0.8392 | 0.9213 |

**Key findings:**
- XGBoost achieved a clean sweep — best F1, AUC-ROC, precision, recall, and accuracy
- Deep learning models were competitive (F1: 0.9473–0.9554) but did not surpass classical ML
- MLP was the strongest deep learning model
- BiLSTM underperformed even after extending to 200 epochs, confirming recurrent architectures are suboptimal for aggregated tabular IIoT features
- SHAP analysis identified top features: `network_packets_dst_count`, `network_packet-size_min`, `network_time-delta_min` — all consistent with known volumetric attack signatures

---

## Dataset

**DataSense: CIC IIoT Dataset 2025**  
Canadian Institute for Cybersecurity, University of New Brunswick  
https://www.unb.ca/cic/datasets/iiot-dataset-2025.html

| Category | Attack Variants | Log Records |
|----------|----------------|-------------|
| Benign | Normal traffic | 72,554 |
| DDoS | 14 | 298,576 |
| DoS | 14 | 227,553 |
| Reconnaissance | 9 | 689,041 |
| Web Attacks | 5 | 56,242 |
| Brute Force | 2 | 37,675 |
| MITM | 3 | 166,831 |
| Mirai Malware | 2 | 152,534 |

- 50 features extracted via CICFlowMeter
- 5-second sliding window aggregation
- Binary label: Benign (0) vs. Attack (1)
- Train/Val/Test split: 70/15/15

---

## Project Structure

```
ids-iiot-deeplearning/
├── data/                        # Processed datasets (X_train, X_test, y_train, y_test)
├── models/                      # Saved model files (.pkl, .h5)
├── notebooks/
│   ├── 01_eda.ipynb             # Exploratory data analysis
│   ├── 02_preprocessing.ipynb   # Feature engineering and preprocessing
│   ├── 03_logistic_regression.ipynb
│   ├── 04_random_forest.ipynb
│   ├── 05_xgboost.ipynb
│   ├── 06_mlp.ipynb
│   ├── 07_lstm.ipynb            # BiLSTM (extended to 200 epochs)
│   ├── 08_shap_analysis.ipynb   # SHAP feature importance on XGBoost
│   └── 09_dataset_merge.ipynb   # Cross-dataset compatibility investigation
├── results/
│   ├── model_comparison.csv     # Full metrics table
│   ├── shap_feature_importance.csv
│   ├── shap_beeswarm.png
│   └── shap_bar.png
└── README.md
```

---

## Models

### Classical ML
- **Logistic Regression** — L2 regularization, lbfgs solver, class-weighted
- **Random Forest** — 100 trees, max depth 20, class-weighted, Gini importance
- **XGBoost** — 200 estimators, lr=0.1, max_depth=6, scale_pos_weight for imbalance

### Deep Learning
- **MLP** — 3 hidden layers (256→128→64), ReLU, Dropout 0.3, early stopping (patience=10), stopped at epoch 77
- **1D-CNN** — 2× Conv1D (64/128 filters, kernel=3), GlobalMaxPool, Dense 64, trained to 100 epochs
- **BiLSTM** — 2× Bidirectional LSTM (64 units), Dense 32, extended to 200 epochs, stopped at epoch 157

---

## SHAP Analysis

SHAP (TreeExplainer) was applied to XGBoost on a 500-sample test subset.

**Top 3 features (SHAP value 1.7–2.07):**
1. `network_packets_dst_count` — high values indicate flooding attacks
2. `network_packet-size_min` — low values indicate malformed/crafted packets
3. `network_time-delta_min` — low values indicate burst/rapid-fire traffic

Features 4–10 had SHAP values of 0.27–0.80, meaning the top 3 features are 2–7× more impactful than the rest.

---

## Dataset Merge Investigation

Cross-dataset merging was investigated to expand training data:

- **CIC-IoT-2023** (UNB) — 39 features with incompatible naming conventions
- **CIC-IDS-2017** (UNB) — 79 features separating fwd/bwd traffic; DataSense uses combined flow features

**Conclusion:** Direct merging is not viable without re-extracting from raw PCAPs using a unified CICFlowMeter configuration. Cross-dataset generalization is noted as future work.

---

## Setup

```bash
git clone https://github.com/cyberwithcyril/ids-iiot-deeplearning.git
cd ids-iiot-deeplearning
python -m venv .venv
.venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

Open notebooks in order (01 → 09) in PyCharm or Jupyter.

---

## Requirements

```
pandas
numpy
scikit-learn
xgboost
tensorflow
shap
matplotlib
seaborn
joblib
```

---

## Citation

If you use this work or the dataset, please cite:

```
Canadian Institute for Cybersecurity (CIC), University of New Brunswick.
DataSense: CIC IIoT Dataset 2025.
https://www.unb.ca/cic/datasets/iiot-dataset-2025.html
```

---

## License

MIT License — see LICENSE for details.
