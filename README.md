# Credit Risk & Expected Loss Analytics

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://www.python.org/)
[![Excel](https://img.shields.io/badge/Model-Excel-217346.svg)](Model/)
[![SQL](https://img.shields.io/badge/SQL-Analytics-informational.svg)](SQL/)
[![Data](https://img.shields.io/badge/Data-Synthetic-lightgrey.svg)](#-data--important-disclaimer)

> An end-to-end credit-risk analytics framework that estimates **PD, LGD, EAD and Expected Loss**, validates a logistic-regression PD model, measures portfolio concentration and rating migration, and quantifies the impact of stressed credit conditions.

**Portfolio at a glance:** 5,000 loans · $752.3M exposure · $806.6M EAD · 1.88% observed default rate · **$8.24M expected loss (1.02% of EAD)**

---

## 📌 Project Overview

This project mirrors the workflow of a bank's credit-risk analytics team, moving from a loan-level data tape to a management-ready view of portfolio loss:

```
Loan Portfolio
   ↓
Risk Segmentation (grade, product, region, industry, delinquency, vintage)
   ↓
PD Model (logistic regression, validated on hold-out data)
   ↓
LGD (recovery-based, product-level)  +  EAD (drawn balance + CCF × undrawn)
   ↓
Expected Loss  =  PD × LGD × EAD
   ↓
Rating Migration Matrix
   ↓
Stress Testing (Base / Moderate / Severe)
   ↓
Management Dashboard & Model Audit Controls
```

**Core question:** *How much credit loss should this portfolio be expected to generate, where is the risk concentrated, and how sensitive is that loss to an adverse economic environment?*

---

## 🧪 Data & Important Disclaimer

> **All data in this repository is synthetic.** No real borrowers, banks, or proprietary information are used. Results illustrate methodology and are **not** a production credit model, a regulatory submission, or investment or lending advice.

The loan tape (`Data/Credit_Risk_Portfolio_Analytics.csv`, 5,000 records) contains:

| Group | Fields |
|---|---|
| **Identifiers & segments** | `Loan_ID`, `Product`, `Region`, `Industry`, `Vintage_Year` (2022–2026) |
| **Borrower & loan features** | `Annual_Income`, `Credit_Score`, `DTI`, `Tenure_Months`, `Collateral_Coverage`, `Exposure`, `Delinquency_Bucket` |
| **Risk outputs** | `Risk_Grade` (AA, A, B, C, D, E), `Default_Flag`, `PD_Model`, `LGD`, `EAD`, `Expected_Loss` |

**Products:** Auto Loan · Credit Card · Retail Term Loan · SME Loan
**Regions:** North · South · East · West · Central
**Industries:** Manufacturing · Technology · Healthcare · Services · Retail · Other

---

## 📊 Key Results

| Metric | Value |
|---|---|
| Loans | 5,000 |
| Total exposure | $752,291,444.01 |
| Total EAD | $806,632,235.85 |
| Observed default rate | 1.88% |
| EAD-weighted average PD | ~1.91% |
| EAD-weighted average LGD | ~45.7% |
| **Portfolio Expected Loss** | **$8,235,782.57** |
| **EL / EAD** | **~1.02%** |
| PD model ROC-AUC | 0.669 |
| Gini coefficient (2 × AUC − 1) | ~0.34 |
| Brier score | 0.018 |
| Max EL reconciliation difference | 0.00000000006 |

### Expected loss by product

| Product | Loans | Exposure | Expected Loss |
|---|---:|---:|---:|
| Retail Term Loan | 1,804 | $266.6M | $4.09M |
| Credit Card | 1,206 | $182.6M | $3.45M |
| Auto Loan | 1,245 | $192.7M | $0.42M |
| SME Loan | 745 | $110.4M | $0.27M |

> Credit Card and Retail Term Loan (both **unsecured**) account for ~91% of portfolio expected loss despite being ~60% of exposure, which is the central concentration insight: loss is driven by collateral and product structure, not just by balance.

---

## 🔬 Methodology

### 1. Probability of Default (PD)

A binary **logistic regression** (12-month default flag) is fitted on a 70:30 stratified split using four drivers:

| Driver | Coefficient (β) | Interpretation |
|---|---:|---|
| Credit Score | −0.009688 | Higher score → lower PD |
| DTI | +0.032063 | Higher leverage → higher PD |
| Delinquency Bucket | +0.263326 | Past dues sharply raise PD |
| Collateral Coverage | −0.145200 | Higher collateral → lower PD |

Validation includes ROC-AUC, Gini, Brier score, and a **decile calibration table** (predicted vs. actual default rate, calibration ratio, cumulative capture) to confirm rank-ordering. The top PD decile captures the largest share of observed defaults.

### 2. Loss Given Default (LGD)

`LGD = 1 − Recovery Rate`, set by product and collateral type.

| Product | Facility type | Collateral | Recovery | LGD |
|---|---|---|---:|---:|
| Auto Loan | Secured retail | Vehicle title | 89.32% | 10.68% |
| SME Loan | Secured commercial | Equipment & real estate | 88.84% | 11.16% |
| Credit Card | Unsecured revolving | None | 32.24% | 67.76% |
| Retail Term Loan | Unsecured amortizing | None | 32.33% | 67.67% |

Loan-level LGD reflects collateral coverage on top of these product baselines (see `Methodology/Credit_Risk_Methodology.md`).

### 3. Exposure at Default (EAD)

`EAD = Drawn Balance + CCF × Undrawn Limit`, with credit conversion factors by product (Auto 20%, Credit Card 50%, Retail Term 0%, SME 30%). Unsecured revolving lines show the largest EAD uplift, reflecting drawdown behaviour ahead of default.

### 4. Expected Loss

$$EL = PD \times LGD \times EAD$$

Computed at **loan, risk-grade, segment, and portfolio** level, with an explicit loan-level tie-out between summed loan EL and the reported portfolio figure.

### 5. Rating Migration

A 1-year transition matrix across grades AA → E plus Default, with row-sum checks and upgrade/downgrade drift metrics. The matrix is an **assumed illustrative matrix**, not estimated from observed cohorts.

---

## 📗 Excel Model Architecture

`Model/Credit_Risk_Expected_Loss_Model_Professional.xlsx` — nine formula-driven sheets:

| Sheet | Purpose |
|---|---|
| `01_Executive_Summary` | KPI cards (exposure, EAD, EL, default rate, weighted PD/LGD) and product-level breakdown |
| `02_Portfolio_Analytics` | Risk-grade profile, region and delinquency concentration |
| `03_PD_Engine` | Model specification, coefficients, odds ratios, AUC/Gini/Brier, decile calibration |
| `04_LGD_EAD_Engine` | Recovery, LGD, CCF and EAD assumptions by product |
| `05_Expected_Loss` | EL reconciliation by risk grade and portfolio total |
| `06_Credit_Migration` | 1-year transition matrix and drift metrics |
| `07_Stress_Testing` | Scenario definitions, stressed EL, PD × LGD sensitivity matrix |
| `08_Model_Audit_Controls` | Automated integrity and reconciliation checks |
| `09_Data_Tape` | 5,000-loan source data driving all sheets via live formulas |

---

## 🌪️ Stress Testing

Three scenarios apply transparent shocks to PD, LGD and EAD. The Base case is reconciled directly to the portfolio expected loss.

| Scenario | PD multiplier | LGD add-on | EAD multiplier | Stressed EL | Incremental loss | Uplift |
|---|---:|---:|---:|---:|---:|---:|
| Base | 1.00× | +0.00 | 1.00× | $8.24M | — | — |
| Moderate Stress | 1.25× | +5 pp | 1.03× | ~$11.76M | ~$3.53M | ~+43% |
| Severe Stress | 1.60× | +12 pp | 1.07× | ~$17.80M | ~$9.57M | ~+116% |

Loss rate (EL / EAD) rises from **~1.02%** in Base to **~1.46%** (Moderate) and **~2.21%** (Severe).

The sheet also includes a **PD-shock × LGD-add-on sensitivity grid** (PD 1.00×–1.60×, LGD +0 to +12 pp).

> **Method note:** Stressed EL is calculated at portfolio level as `Base EL × PD multiplier × (weighted LGD + add-on) / weighted LGD × EAD multiplier`. This is a transparent aggregate approximation; a loan-level re-scoring under shocked inputs is a natural extension.

---

## ✅ Model Controls & Audit Trail

`08_Model_Audit_Controls` runs automated checks:

| ID | Control | Criterion | Severity |
|---|---|---|---|
| CTL-01 | PD bounds | 0 ≤ PD ≤ 1 for all loans | High |
| CTL-02 | LGD bounds | 0 ≤ LGD ≤ 1 for all loans | High |
| CTL-03 | Positive exposure & EAD | Exposure > 0 and EAD > 0 | Critical |
| CTL-04 | EL tie-out | \|Σ loan EL − table EL\| < $1 | Critical |
| CTL-05 | Stress base tie-out | Base stressed EL = portfolio EL | Critical |
| CTL-06 | Data completeness | 5,000 records, no nulls | Medium |
| CTL-07 | Migration row sums | Each transition row sums to 100% | High |



---

## 🗄️ SQL Analytics

`SQL/credit_risk_queries.sql` provides reporting queries for:

- Default-rate reporting by product, region, industry and grade
- Risk-grade profile analysis
- Delinquency-bucket trends
- Exposure and sector/geography concentration
- Vintage analysis (default and loss by origination year)
- Expected-loss aggregation at loan, grade, segment and portfolio level

---

## 🐍 Python Pipeline

`Python/credit_risk_pd_model.py` covers:

- Data preparation and feature validation
- PD model training (train/test split, logistic regression)
- Validation: ROC-AUC, confusion matrix, calibration analysis, PD by risk grade
- LGD/EAD assignment and loan-level EL calculation
- Stress testing under Base / Moderate / Severe scenarios (loan-level)
- Visual analytics (ROC curve, calibration plot, concentration and stress charts)


---

## 📁 Repository Structure

```
Credit-Risk-Analytics/
│
├── README.md
├── LICENSE
│
├── Model/
│   └── Credit_Risk_Model.xlsx  
│
├── Data/
│   └── Credit_Risk_Portfolio_Analytics.csv                 
│
├── SQL/
│   └── credit_risk_queries.sql                             
│
├── Reports/
│   └── Credit_Risk_Analytics_Report.pdf                    
│                                            
│
└── Methodology/
    │ 
    └── methodology.md                      
```

**Suggested `requirements.txt`:** `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `openpyxl`

---

## ⚠️ Limitations & Scope

This is a **synthetic-data portfolio project** built to demonstrate methodology. Please read results accordingly:

- **Synthetic data.** Default behaviour, recoveries and relationships are simulated and do not reflect a real book.
- **Modest discrimination.** ROC-AUC of 0.669 is illustrative; a production PD model would require richer features, out-of-time validation and monitoring.
- **Assumption-based LGD, CCF and migration.** These are set assumptions, not calibrated to observed workout or cohort data.
- **Low default count.** With ~94 defaults, decile-level calibration is noisy.
- **Aggregate stress method in Excel.** Stress in the workbook is a portfolio-level scaling; it is not a full macro-linked, loan-level projection.
- **Not regulatory-compliant.** The framework is *inspired by* IFRS 9 / Basel and DFAST/CCAR concepts (12-month PD, PD × LGD × EAD, downturn LGD, adverse scenarios). It does not implement staging, lifetime ECL, forward-looking macro overlays, or regulatory capital.
- **Not advice.** Nothing here constitutes lending, investment, or risk-management advice.

---

## 🛠️ Skills Demonstrated

- Credit-risk modeling: PD, LGD, EAD, Expected Loss (PD × LGD × EAD)
- Logistic regression, ROC-AUC/Gini, Brier score, decile calibration
- Portfolio segmentation, concentration and vintage analysis
- Rating-migration matrices and stress-scenario design
- Formula-driven Excel modeling with reconciliation and audit controls
- Model governance: versioned findings, tie-outs, bounds and completeness checks
- Analytical SQL and Python (pandas, scikit-learn) for risk analytics
- Power BI dashboard design for management reporting

---

## 📄 License

Released under the [MIT License](LICENSE).

---

*Developed by [Yashraj1203](https://github.com/Yashraj1203) — synthetic data, for educational and portfolio purposes only.*
