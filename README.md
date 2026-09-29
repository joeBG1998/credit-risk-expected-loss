## Credit Risk Modelling & Expected Loss Framework

End-to-end credit risk modelling project using Lending Club consumer-loan data. The project estimates borrower-level Probability of Default (PD), calibrates predicted probabilities using out-of-time validation, derives a recovery-based Loss Given Default (LGD), calculates Exposure at Default (EAD), and translates these components into portfolio Expected Loss (EL) and stress scenarios.

## Key results

- **LightGBM test ROC-AUC:** 0.7071
- **Test KS statistic after calibration:** 0.3000
- **Calibrated mean test PD:** 21.74%
- **Realised test default rate:** 21.29%
- **Mature historical LGD:** 89.62%
- **Base portfolio expected loss:** approximately $706.9m
- **Severe-stress expected loss:** approximately $1.118bn
- **Severe-stress increase vs. base:** approximately 58.16%

## Project objective

The project answers a practical credit-risk question:

> Can borrower and loan characteristics available at origination be used to estimate default risk and translate those estimates into portfolio expected credit losses?

The core framework is:

**Expected Loss = PD × LGD × EAD**

## Methodology

### 1. Target definition

Resolved Lending Club loan outcomes are used for supervised modelling.

**Default**
- Charged Off
- Default
- Legacy credit-policy charged-off status

**Non-default**
- Fully Paid
- Legacy credit-policy fully-paid status

Current, late, grace-period and otherwise unresolved loans are excluded from the supervised target.

### 2. Out-of-time validation

A chronological split is used instead of a random train/test split:

- **Train:** 2007–2014
- **Validation:** 2015–2016
- **Test:** 2017–2018

This is intended to provide a more realistic assessment of performance on later lending cohorts and reveal temporal drift in both discrimination and calibration.

### 3. Feature engineering and leakage control

The PD model uses borrower and loan information available at or near origination. Post-origination variables such as later payments, recoveries, remaining principal and settlement information are excluded from the PD feature set.

Key transformations include:

- midpoint FICO score
- ordinal employment length
- median numerical imputation
- categorical imputation and one-hot encoding
- standardisation for Logistic Regression

Lending Club `grade` and `sub_grade` are excluded from the primary model because they embed the lender's own underwriting assessment.

### 4. PD models

Two models are compared:

- **Logistic Regression** — interpretable baseline
- **LightGBM** — nonlinear gradient-boosting model

Evaluation metrics include:

- ROC-AUC
- PR-AUC
- Kolmogorov-Smirnov statistic
- Brier score

LightGBM is selected as the champion model based on stronger out-of-time performance.

### 5. Probability calibration

Later cohorts exhibit higher realised default rates than the training period. Isotonic regression is fitted on the validation sample and applied to the untouched test sample.

This preserves borrower ranking while improving absolute PD estimation.

### 6. Model interpretation

SHAP values are used to identify major drivers of predicted default risk. Important drivers include:

- interest rate
- loan term
- annual income
- recent account openings
- loan amount
- debt-to-income ratio
- FICO score
- revolving-credit utilisation

Interest rate is interpreted cautiously because Lending Club pricing partly embeds its own underwriting assessment.

### 7. LGD and EAD

For defaulted loans:

**EAD at default ≈ Funded Amount − Principal Repaid**

**Net Recovery = Recoveries − Collection Recovery Fees**

**LGD = 1 − Net Recovery / EAD at Default**

To reduce recovery-window censoring, the base LGD assumption uses mature historical defaults through 2015.

For portfolio expected-loss calculations at origination:

**EAD = Funded Amount**

### 8. Portfolio expected loss

Loan-level expected loss is calculated as:

**ELᵢ = PDᵢ × LGD × EADᵢ**

The test portfolio is segmented into five PD-based risk bands from Very Low to Very High risk, and realised default rates increase materially across these bands.

### 9. Stress testing

Illustrative scenarios apply simultaneous shocks to PD and LGD:

- Base
- Moderate downturn
- Severe downturn
- Extreme stress

These scenarios are sensitivity analyses, not regulatory stress scenarios.

## Repository structure

```text
credit-risk-expected-loss/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
└── notebooks/
    └── credit_risk_expected_loss.ipynb
```

## Data

The raw Lending Club CSV is **not included** in this repository because of its size.

Place the following file in the project root, or update `DATA_PATH` in the notebook:

```text
accepted_2007_to_2018Q4.csv
```

See `data/README.md` for details.

## How to run

1. Clone or download this repository.
2. Create a Python environment.
3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Place the Lending Club CSV where the notebook expects it, or update `DATA_PATH`.
5. Open:

```text
notebooks/credit_risk_expected_loss.ipynb
```

6. Run all cells from top to bottom.

## Main libraries

- pandas
- NumPy
- scikit-learn
- LightGBM
- SHAP
- SciPy
- Matplotlib

## Limitations

- The 2017–2018 test sample contains only loans with resolved outcomes, which may create maturity-selection bias.
- LGD is reconstructed from Lending Club repayment and recovery fields rather than directly observed bank accounting exposure at default.
- Interest rate partly embeds Lending Club's own underwriting assessment.
- Stress scenarios are illustrative sensitivity tests rather than regulatory macroeconomic scenarios.
- Lending Club is unsecured consumer credit, so the results should not be directly generalised to corporate or wholesale credit portfolios.

## Career relevance

This project demonstrates practical skills relevant to:

- Credit Risk
- Quantitative Risk
- Model Risk
- Risk Analytics
- Portfolio Risk
- Financial Data Analytics
