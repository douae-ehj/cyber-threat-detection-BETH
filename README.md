# Cyber Threat Detection with Deep Learning (BETH Dataset)

## Overview

This repository contains an implementation of a deep learning intrusion detection system trained on the **BETH dataset** : a collection of real Linux host telemetry captured via eBPF system-call tracing, enriched with honeypot attack data. The goal is binary classification: distinguishing **suspicious/malicious system-call events** (`sus_label = 1`) from **benign** ones (`sus_label = 0`).

The dataset used in this project is a preprocessed split (train / validation / test) derived from the original BETH dataset. The split was prepared by DataCamp as part of a guided project. The original BETH dataset is described in the paper [*"BETH Dataset: Real Cybersecurity Data for Anomaly Detection Research"*](https://www.gatsby.ucl.ac.uk/~balaji/udl2021/accepted-papers/UDL2021-paper-033.pdf) (Haider et al., 2021).

> **Data access:** The preprocessed CSV files are not included in this repository due to their size and redistribution constraints. You can obtain them from DataCamp's *"Detecting Cyber Threats with Deep Learning"* project, or use the original BETH dataset from the paper linked above and apply your own train/validation/test split.

---

## Dataset Characteristics

### Size and Composition

- Each row represents a single Linux system-call event.
- The dataset is **heavily imbalanced**: approximately **99.83 % benign** vs **0.17 % malicious** events.
- Pre-split into training, validation, and test sets.

### Features

The dataset contains **7 numerical features** and 1 binary target label:

| Feature             | Description                                                     |
| ------------------- | --------------------------------------------------------------- |
| `processId`       | ID of the process that triggered the event                      |
| `threadId`        | ID of the thread within the process                             |
| `parentProcessId` | ID of the parent process                                        |
| `userId`          | ID of the user who owns the process                             |
| `mountNamespace`  | Linux mount namespace identifier                                |
| `argsNum`         | Number of arguments passed to the system call                   |
| `returnValue`     | Return value of the system call (0 = success, negative = error) |
| `sus_label`       | **Target:** 0 = benign, 1 = suspicious/malicious          |

---

## Contents

### Dataset Characteristics and Class Imbalance Analysis

- Overview of dataset size across train, validation, and test splits.
- Analysis of the severe class imbalance (~600:1 ratio) and its implications for model design.

### Data Preprocessing

- Feature/label separation for each split.
- Feature normalisation using `StandardScaler` fitted **only on the training set** to prevent data leakage.
- Conversion of numpy arrays to PyTorch `float32` tensors.
- Construction of `DataLoader` objects with batching and shuffling.

### Handling Class Imbalance

- Use of `pos_weight ≈ 600` in `BCEWithLogitsLoss` to penalise false negatives (missed attacks) more heavily than false positives.
- This approach avoids resampling (e.g. SMOTE) and instead encodes the class imbalance directly into the loss function.

### Deep Learning Model

- A **3-layer Multi-Layer Perceptron (MLP)** implemented in PyTorch:
  - Architecture: `7 → 64 → 32 → 1`
  - Activations: ReLU after each hidden layer
  - Regularisation: Dropout (p=0.3) after each hidden layer
  - Output: single logit converted to probability via sigmoid
- Trained with the **Adam** optimiser (lr=1e-4) for 14 epochs using `BCEWithLogitsLoss`.

### Performance Evaluation

- Validation accuracy, precision, recall, F1-score, and confusion matrix.
- Discussion of key metrics for imbalanced security datasets (emphasis on recall for the malicious class).

---

## Key Features

- Addresses **extreme class imbalance** via loss-level weighting rather than resampling.
- Prevents **data leakage** by fitting the scaler exclusively on training data.
- Uses **`BCEWithLogitsLoss` with `pos_weight`** for numerically stable, imbalance-aware binary cross-entropy.
- Saves both the **model weights** (`model.pth`) and **fitted scaler** (`scaler.pkl`) for reproducible inference.
- Achieves near-perfect detection performance despite the severe class imbalance.

---

## Results

| Metric    | Benign (0) | Suspicious (1) |
| --------- | ---------- | -------------- |
| Precision | 1.00       | 1.00           |
| Recall    | 1.00       | 0.99           |
| F1-score  | 1.00       | 0.99           |

**Confusion Matrix (Validation Set):**

```
[[188178      3]
 [     9    777]]
```

- **True Negatives:** 188,178 benign events correctly classified
- **False Positives:** 3 benign events incorrectly flagged
- **False Negatives:** 9 attacks missed by the model
- **True Positives:** 777 attacks correctly detected

---

## Repository Structure

```
cyber-threat-detection-BETH/
│
├── cyber_threat_detection_BETH.ipynb   # Main notebook
├── requirements.txt                    # Python dependencies
├── model.pth                           # Trained model weights (generated after running notebook)
├── scaler.pkl                          # Fitted StandardScaler (generated after running notebook)
└── README.md
```

> The dataset CSV files (`labelled_train.csv`, `labelled_validation.csv`, `labelled_test.csv`) are **not included** — see the Data Access note in the Overview section above.

---

## Getting Started

### Prerequisites

- Python 3.9+
- pip

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/douae-ehj/cyber-threat-detection-BETH.git
cd cyber-threat-detection-BETH

# 2. (Recommended) Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Linux/macOS
venv\Scripts\activate           # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Place the dataset CSV files in the project root
#    labelled_train.csv | labelled_validation.csv | labelled_test.csv

# 5. Launch the notebook
jupyter notebook cyber_threat_detection_BETH.ipynb
```

---

## Dependencies

See `requirements.txt`. Core libraries:

- `torch` — model definition and training
- `torchmetrics` — accuracy metric
- `scikit-learn` — preprocessing and evaluation metrics
- `pandas` — data loading and manipulation
- `joblib` — scaler serialisation

---

## References

- Haider, S. et al. (2021). *BETH Dataset: Real Cybersecurity Data for Anomaly Detection Research*. UDL Workshop @ ICML 2021. [[PDF]](https://www.gatsby.ucl.ac.uk/~balaji/udl2021/accepted-papers/UDL2021-paper-033.pdf)
