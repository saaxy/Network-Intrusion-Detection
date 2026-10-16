# Network Intrusion Detection System using Machine Learning

A classical supervised machine learning project that classifies network connections as **Normal** or **Attack**, built and explained in a single Jupyter/Google Colab notebook.

## Problem

Given the characteristics of a network connection (protocol, service, bytes sent, error rates, and so on), can a model decide whether the traffic is normal or malicious? This is a binary classification problem.

## Dataset

NSL-KDD, a cleaned version of the KDD Cup 1999 intrusion dataset (duplicates removed).

- Training file: KDDTrain+ (125,973 records)
- Test file: KDDTest+ (22,544 records), saved here as `test.csv`
- 41 features: 3 categorical (`protocol_type`, `service`, `flag`) and the rest numeric
- Target: `label` (normal or a specific attack name), converted to 0 = Normal, 1 = Attack
- Attack families grouped into "Attack": DoS, Probe, R2L, U2R

The notebook downloads the training data automatically. `test.csv` is the official test file with a header row added.

Why NSL-KDD: small enough for Colab, well documented, clear labels, and easy preprocessing. CICIDS2017 is several GB and UNSW-NB15 is much messier.

## Approach

1. Load data and attach the official column names
2. Clean: no missing values or duplicates; drop `difficulty_level` (dataset metadata, not traffic) and constant columns
3. EDA: class balance, attack types, protocol, flag, `src_bytes`, correlations
4. Create the binary target
5. Stratified 80/20 train/validation split
6. One-hot encode categorical features and standardize numeric features (fitted on training data only)
7. Train Logistic Regression, Decision Tree, Random Forest, XGBoost
8. Compare with accuracy, precision, recall, F1, ROC-AUC and confusion matrices
9. Choose the final model on validation data, then evaluate on `test.csv`
10. Save the preprocessor and model with joblib

## Results

Validation set (random 20% split of the training file):

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic Regression | 0.9719 | 0.9762 | 0.9630 | 0.9696 |
| Decision Tree | 0.9981 | 0.9978 | 0.9982 | 0.9980 |
| Random Forest | 0.9990 | 0.9995 | 0.9985 | 0.9990 |
| XGBoost | 0.9993 | 0.9992 | 0.9992 | 0.9992 |

Selected model: XGBoost (highest validation F1).

Official test set (`test.csv`), XGBoost: accuracy 0.794, precision 0.969, recall 0.660, F1 0.785.

### Why the test score is much lower

The official test set contains attack types that never appear in training, and a different mix of traffic. The model catches high-volume attacks that it has seen (neptune, smurf, satan, back, portsweep, ipsweep) almost perfectly, but misses unseen types (for example snmpguess, mailbomb) and very rare ones (guess_passwd has only 53 training rows). Precision stays high, recall drops. This is a known limit of supervised detection: it recognizes attack patterns it has learned. The random-split score is optimistic, and the test.csv score is the more honest estimate.

## Limitations

- The data reflects 1999-era traffic, so this is a learning benchmark, not a modern production dataset.
- Supervised models do not detect brand-new attack types.
- No hyperparameter tuning; default-style settings were used to keep the project simple.

## Future work

- Multi-class classification (DoS, Probe, R2L, U2R)
- Cross-validation and hyperparameter tuning
- Class weights or resampling for rare attack types
- Streamlit app for live predictions

## How to run

Google Colab: upload `network_intrusion_detection.ipynb` (and optionally `test.csv`), then Runtime > Run all.

Locally:

    pip install numpy pandas matplotlib seaborn scikit-learn xgboost joblib jupyter
    jupyter notebook network_intrusion_detection.ipynb

Running the notebook creates `preprocessor.pkl`, `nids_model.pkl` and `metadata.pkl`. These are generated files, so they are not part of the repository.

## Tech stack

Python, pandas, NumPy, scikit-learn, XGBoost, matplotlib, seaborn, joblib
