# AI/ML Banking Fraud Detection System

**Built for the IBM Z Datathon**

An educational banking prototype that combines a Python-based banking workflow with a machine-learning model to estimate transaction fraud risk. The system supports account creation, generated account numbers, login, balance management, deposits, withdrawals, money transfers, transaction history, and fraud screening for withdrawals and transfers.

> **Project status:** Prototype / learning project. This is not a production banking application. Fraud predictions depend on the trained model, dataset, feature preprocessing, and evaluation results.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Proposed Solution](#proposed-solution)
- [Project Objectives](#project-objectives)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [System Architecture](#system-architecture)
- [Machine Learning Approach](#machine-learning-approach)
- [Features Used by the Model](#features-used-by-the-model)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation and Setup](#installation-and-setup)
- [Running the Application](#running-the-application)
- [Testing Fraud Detection](#testing-fraud-detection)
- [Example Workflow](#example-workflow)
- [IBM Z / LinuxONE Relevance](#ibm-z--linuxone-relevance)
- [Security and Data Handling](#security-and-data-handling)
- [Limitations](#limitations)
- [Future Enhancements](#future-enhancements)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [Disclaimer](#disclaimer)

---

## Overview

Digital banking makes financial transactions convenient, but fraudulent transactions can cause financial losses for customers and financial institutions. This project demonstrates how machine learning can be used to screen transactions for potential fraud.

The application combines a simple command-line banking system with a saved machine-learning model. When a user attempts a supported transaction, the application constructs a feature row, sends it to the model, obtains a fraud-risk score when probability prediction is supported, and compares the score with a configurable threshold. A transaction classified as high risk is flagged and blocked in the prototype; a transaction that passes screening continues through the simulated banking workflow.

The project was developed as a prototype for the **IBM Z Datathon**, exploring how AI/ML-based fraud screening could complement secure, scalable enterprise computing environments such as IBM Z and LinuxONE.

## Problem Statement

Banking systems need to identify suspicious transactions quickly without unnecessarily interrupting legitimate activity. Rules based only on a transaction amount may miss complex patterns, while excessive alerts can create friction for genuine customers.

The project explores a machine-learning approach that evaluates transaction attributes and produces a risk score to help identify potentially fraudulent withdrawals and transfers.

## Proposed Solution

The prototype uses a trained classification model to evaluate transactions. The banking application collects or calculates transaction details, formats them into the feature schema expected by the model, and obtains a prediction. The result is compared with a configurable threshold:

- **Below threshold:** the transaction passes the model's screening step.
- **At or above threshold:** the transaction is flagged and blocked by the prototype.
- **Prediction error or incompatible model:** the transaction is cancelled rather than proceeding without screening.

A passing score does **not** guarantee that a transaction is legitimate. It only means that the model did not exceed the configured threshold.

## Project Objectives

- Demonstrate a basic banking workflow in Python.
- Generate a unique 10-digit account number for each account created during a program session.
- Support login, balance checks, deposits, withdrawals, transfers, and transaction history.
- Apply ML-based fraud screening to withdrawals and transfers.
- Display a fraud-risk score and flag high-risk transactions.
- Explore the integration of machine learning with enterprise infrastructure concepts relevant to IBM Z / LinuxONE.
- Identify the data, evaluation, and security requirements needed before any real-world deployment.

## Key Features

### Banking workflow
- Create an account with a username and four-digit PIN.
- Generate a 10-digit account number.
- Log in and log out.
- Check the current balance.
- Deposit money.
- Withdraw money after fraud screening.
- Transfer money using the receiver's account number.
- View transaction history for the logged-in user.

### Fraud screening
- Load a previously trained model from `fraud_model.pkl`.
- Prepare a transaction feature row using pandas.
- Call `predict_proba()` when the model supports it.
- Find the probability associated with fraud class label `1`.
- Compare the score with the configured threshold (default: `0.70`).
- Flag and block transactions whose score meets or exceeds the threshold.
- Cancel a transaction if model inference fails.

### Important implementation details
- Account data and transaction history are stored in memory only.
- Account numbers are unique within the current program session; they are not persisted or guaranteed unique across separate runs.
- The example code uses `float` for amounts, which can introduce rounding issues. Real financial software should use integer minor units (such as paise) or a suitable decimal arithmetic strategy.
- The model input columns and preprocessing must match the model's training pipeline exactly.

## How It Works

1. **Account creation:** the application accepts a username and PIN, generates an account number, and initializes the balance.
2. **Login:** the user enters their username and PIN.
3. **Transaction request:** the user chooses deposit, withdrawal, or transfer.
4. **Basic validation:** the application validates the amount, receiver, and available balance.
5. **Feature preparation:** for supported withdrawals and transfers, the application prepares transaction features such as amount and before/after balances.
6. **Model inference:** the saved classifier estimates the transaction's fraud probability, if supported.
7. **Risk threshold:** the score is compared with the configured threshold.
8. **Decision:** a high-risk transaction is flagged and blocked; a transaction passing screening proceeds in the simulated application.
9. **History:** the application records successful transactions and flagged attempts in its in-memory transaction history.

## System Architecture

```text
                 +---------------------------+
                 |     User / CLI Menu       |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 |    Banking Application    |
                 | Accounts, PIN, Balances   |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 | Transaction Validation    |
                 | Amount / Receiver / Funds  |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 | Feature Preparation       |
                 | pandas DataFrame          |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 | Trained ML Classifier     |
                 | fraud_model.pkl           |
                 +-------------+-------------+
                               |
                 +-------------v-------------+
                 | Fraud Risk Score          |
                 | Threshold Comparison      |
                 +-------------+-------------+
                               |
                +--------------+---------------+
                |                              |
        +-------v--------+            +--------v---------+
        | Flag / Block   |            | Allow Prototype  |
        | High-Risk      |            | Transaction       |
        +----------------+            +------------------+
```

## Machine Learning Approach

### Task type

Fraud detection is typically formulated as a **supervised binary classification** problem when labelled examples of legitimate and fraudulent transactions are available.

- `0`: legitimate / non-fraudulent transaction
- `1`: fraudulent transaction

The code expects fraud to be represented by class label `1`. If the training dataset uses different labels, update the inference logic accordingly.

### Model algorithm

The banking application loads a trained model from `fraud_model.pkl`; the exact algorithm depends on the model you trained and saved. For example, if the training code used Random Forest, then the model is a Random Forest classifier. Do not describe the project as using Random Forest, XGBoost, Logistic Regression, or another specific algorithm unless that is the algorithm actually present in the saved model.

### Typical training pipeline

The model-training process for a labelled transaction dataset usually includes:

1. Load the dataset with pandas.
2. Inspect columns, data types, missing values, duplicates, and target labels.
3. Identify the target column that marks fraud.
4. Separate the target label from input features.
5. Split data into training and test sets. For time-dependent transaction data, consider a chronological split to avoid leakage.
6. Preprocess numeric and categorical columns consistently.
7. Train a suitable classification algorithm.
8. Evaluate it on held-out data.
9. Save the complete fitted pipeline or model with `joblib`.
10. Load the saved artifact in the banking application and pass the same feature schema during inference.

The training script and actual metrics should be added to this repository if available. The runtime banking script alone does not train the model.

### Evaluation metrics to report

Fraud datasets are often imbalanced, so accuracy alone can be misleading. Evaluate and report metrics on held-out data, including:

- **Precision:** of the transactions flagged as fraud, how many were fraud?
- **Recall:** of the actual fraud cases, how many did the model detect?
- **F1-score:** balance between precision and recall.
- **Confusion matrix:** true positives, false positives, true negatives, and false negatives.
- **PR-AUC:** useful for evaluating ranking performance on imbalanced datasets.
- **False-positive rate:** how often legitimate transactions are incorrectly flagged.

Do not add metric values to the README until they have been measured on a clearly identified test set.

## Features Used by the Model

The example inference code is designed around the following PaySim-style feature names:

| Feature | Meaning in the example |
|---|---|
| `step` | Time step from the source transaction schema. In PaySim, this represents a simulation time unit, not simply the number of transactions made. |
| `type` | Transaction category, such as `TRANSFER` or `CASH_OUT`, if these are the exact categories used during training. |
| `amount` | Transaction amount. |
| `oldbalanceOrg` | Sender/origin balance before the transaction. |
| `newbalanceOrig` | Sender/origin balance after the proposed transaction. |
| `oldbalanceDest` | Receiver/destination balance before the transaction. |
| `newbalanceDest` | Receiver/destination balance after the proposed transaction. |

### Feature compatibility is essential

These names are not universal banking fields. They must match the exact training dataset and preprocessing pipeline. If the model was trained on a different dataset with columns such as `TransactionAmount`, `AccountBalance`, `TransactionType`, `Channel`, or `DeviceID`, the example PaySim-style input will not automatically work.

The application must also use the correct categorical values, column order or named-column contract, data types, encoders, and scaling steps. If preprocessing was used during training, save and load the complete fitted pipeline whenever possible.

### Balance assumptions in the prototype

- For transfers, the sender's and receiver's current in-memory balances are used to calculate proposed balances.
- For cash withdrawals, the example uses `CASH_OUT` and zero destination balances. These are modelling assumptions, not universal facts about ATM or cash withdrawals.
- The `step` value in a basic prototype is not a realistic time source unless it is derived according to the training data's definition.

These assumptions should be revisited before interpreting the risk scores as meaningful.

## Technology Stack

| Technology / Library | Role |
|---|---|
| Python 3 | Application logic and model inference |
| pandas | Creating a DataFrame for transaction features |
| joblib | Loading the saved model artifact |
| scikit-learn | Common ML estimator interface and model support, if used by the saved estimator |
| Command-line interface | User interaction for the banking prototype |
| IBM Z / LinuxONE | Target enterprise-platform context explored for the datathon; deployment should only be claimed after it has been tested there |

### Python concepts demonstrated
- Classes and objects (object-oriented programming)
- Dictionaries and lists for in-memory data
- Methods and instance attributes
- Input validation and exception handling
- Loops and conditional statements
- Model loading and inference
- Basic transaction workflow and risk-threshold logic

## Project Structure

A minimal runtime folder can look like this:

```text
bank-fraud-detection/
├── bank.py
├── fraud_model.pkl
├── requirements.txt
├── README.md
└── data/
    └── README.md
```

Optional files you may add as your project grows:

```text
bank-fraud-detection/
├── bank.py
├── train_model.py
├── evaluate_model.py
├── fraud_model.pkl
├── requirements.txt
├── README.md
├── data/
│   └── README.md
├── reports/
│   └── model_metrics.md
└── tests/
    └── test_transactions.py
```

Only list files that actually exist in your repository. Do not commit private customer data, credentials, or datasets whose licences do not permit redistribution.

## Prerequisites

- Python 3.10 or another Python version supported by your installed dependencies.
- `pip`.
- The trained model file `fraud_model.pkl`.
- A model compatible with the feature schema passed by `bank.py`.

## Installation and Setup

### 1. Clone the repository

Replace the placeholder URL with your actual GitHub repository URL:

```bash
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
cd YOUR-REPOSITORY
```

### 2. Create a virtual environment (recommended)

Linux / macOS:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install pandas joblib scikit-learn
```

If your saved model uses other libraries (for example, XGBoost), install the appropriate dependency version too.

### 4. Place the model file

Ensure the model artifact is in the same directory as `bank.py`:

```text
bank-fraud-detection/
├── bank.py
└── fraud_model.pkl
```

If the model file has a different name or location, update the `joblib.load(...)` path in the code.

### 5. Create `requirements.txt`

A basic dependency file for the example is:

```text
pandas
joblib
scikit-learn
```

For reproducibility, pin the versions actually used to train and run the model. Do not assume that these unpinned dependencies will reproduce the original training environment.

## Running the Application

```bash
python bank.py
```

The main menu lets a user create an account, log in, or exit.

Suggested demo sequence:

1. Create two accounts.
2. Record each generated account number.
3. Log in to the first account.
4. Deposit demo funds.
5. Try a withdrawal.
6. Log out and log in to the second account if needed.
7. Return to the first account and transfer funds using the second account's account number.
8. Review the transaction history.

**Prototype note:** Accounts, balances, and transaction history are stored in memory. They reset when the program exits.

## Testing Fraud Detection

### Basic functional tests

- Invalid username and duplicate username.
- PIN with fewer or more than four digits.
- Invalid, zero, or negative transaction amount.
- Withdrawal or transfer with insufficient funds.
- Unknown receiver account number.
- Attempt to transfer to the same account.
- A normal transaction that passes the model threshold.
- A transaction that exceeds the threshold.
- Model loading or inference failure.

### ML-specific tests

Use a held-out test set and examples whose labels are known. Compare predictions with the actual labels and report precision, recall, F1-score, confusion matrix, and PR-AUC as appropriate. Verify that the inference preprocessing is identical to training preprocessing.

Do not assume that a large amount will always be classified as fraud. The model's score depends on the learned patterns and all input features.

### Safe demonstration

For a live demo, use a separate test dataset or a clearly labelled simulation. You can show the risk score, threshold, and final decision. Do not fabricate fraud scores or present a manually forced alert as a genuine model prediction.

## Example Workflow

A transfer in the prototype follows this simplified sequence:

```text
User chooses Transfer
        |
Enter receiver account number
        |
Validate receiver and amount
        |
Check available sender balance
        |
Calculate proposed sender/receiver balances
        |
Build model input features
        |
Run trained model
        |
Compare fraud score with threshold
        |
   +----+----+
   |         |
High risk   Below threshold
   |         |
Flag/block  Update balances
             |
        Record transaction
```

The actual score is produced by the loaded model. It is not a guarantee that the transaction is fraudulent or legitimate.

## IBM Z / LinuxONE Relevance

This project was created for the **IBM Z Datathon** and explores an AI/ML fraud-screening use case relevant to financial services.

IBM Z and LinuxONE are enterprise computing platforms associated with security, reliability, availability, and scalable transaction workloads. A future deployment could investigate running the Python service and its supported ML dependencies in a Linux environment on IBM Z or LinuxONE, with suitable controls for identity, data protection, audit logging, and monitoring.

**Current scope:** this README does not claim that the prototype has been deployed on IBM Z or LinuxONE. Mention a completed deployment, platform-specific optimisation, or benchmark only after it has actually been implemented and verified.

## Security and Data Handling

This is a learning prototype, not a secure banking service.

Current limitations include:

- PINs are stored in plain text in memory.
- No persistent database or durable audit log is implemented.
- Random account numbers are only checked against current in-memory accounts.
- `float` can produce rounding issues for monetary values.
- There is no rate limiting, lockout policy, multi-factor authentication, or session timeout.
- There is no concurrency control or atomic database transaction for simultaneous transfers.
- The model's probability output may not be calibrated and may produce false positives or false negatives.
- The fraud threshold is a configurable demo setting, not a validated banking policy.
- `joblib` model files should only be loaded from trusted sources because pickle-based formats can execute code when deserialized.

Never use real customer information or production credentials in the demo. Do not use the prototype to hold, move, or make decisions about real money.

## Limitations

1. **Model dependency:** the saved model must exist and be compatible with the installed libraries.
2. **Feature mismatch:** inference columns and values must match the model's training pipeline.
3. **Simplified transaction types:** `CASH_OUT` is used as an example for withdrawals and may not represent the training data correctly.
4. **Time step:** the example uses a placeholder time step; PaySim's `step` is time-based.
5. **In-memory state:** accounts and history are lost when the program exits.
6. **Evaluation:** model quality cannot be established from a risk score alone; held-out evaluation is required.
7. **Not production-ready:** authentication, persistence, monitoring, security, and operational controls are incomplete.

## Future Enhancements

- Store accounts and transactions in a database with transaction-safe updates.
- Hash PINs with a suitable password-hashing algorithm and add rate limits.
- Use integer paise or decimal arithmetic for money.
- Add a web dashboard with fraud alerts, risk scores, and transaction filters.
- Add a reproducible model-training and evaluation pipeline.
- Save preprocessing and model together in a fitted scikit-learn pipeline.
- Validate the feature schema before inference.
- Use a time-aware source for `step` if the model expects it.
- Evaluate threshold choices based on precision, recall, false-positive cost, and missed-fraud cost.
- Add automated unit tests and integration tests.
- Add structured logs and audit trails without exposing sensitive information.
- Explore containerised deployment and compatibility testing on IBM Z / LinuxONE.
- Monitor model drift and retrain with appropriately governed, representative data.

## Troubleshooting

### `FileNotFoundError: fraud_model.pkl`

Place `fraud_model.pkl` next to `bank.py`, or update the model path.

### `ModuleNotFoundError`

Activate your virtual environment and install the missing library, for example:

```bash
pip install pandas joblib scikit-learn
```

### Model feature-name or column mismatch

Check the training script. The feature names, data types, categorical encodings, and preprocessing at inference must match training. If the model was trained on a dataset with different columns, adapt the input-building code to that dataset.

### `predict_proba()` is unavailable

Some estimators do not implement probability prediction. Use a compatible classifier or deliberately design the application around the estimator's supported prediction interface; do not treat an arbitrary decision score as a probability.

### Fraud class `1` not found

Inspect the training target labels and the loaded model's `classes_`. Update the fraud-label mapping if the training dataset uses different labels.

### Model loads but predictions look wrong

Confirm that the model was trained on the same type of transaction, that the inference features are realistic, that preprocessing is identical, and that performance was evaluated on a held-out dataset. Do not assume that a score is meaningful merely because the code runs.

## Contributing

Contributions and suggestions are welcome. Useful areas include model evaluation, tests, database persistence, improved security, documentation, and IBM Z / LinuxONE compatibility experiments.

1. Fork the repository.
2. Create a feature branch.
3. Make a focused change.
4. Run relevant tests.
5. Open a pull request describing the change and its validation.

## Disclaimer

This project is an educational prototype created for the **IBM Z Datathon**. It is intended to demonstrate a possible machine-learning workflow for transaction fraud screening. It is not financial advice, a production banking platform, or a guarantee of fraud detection. Predictions may be wrong, and any real deployment would require rigorous validation, security review, privacy controls, and compliance with applicable requirements.
