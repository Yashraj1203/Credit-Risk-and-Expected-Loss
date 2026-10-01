# Credit Risk Model

## Findings corrected

### 1. Expected-loss rounding mismatch
The first build stored rounded PD/LGD/EAD values while the expected-loss value had been calculated from higher-precision intermediate values. V2 recalculates Expected Loss from the stored values and adds an explicit loan-level tie-out check.

### 2. PD methodology
The first build's `PD_Model` represented the latent probability used to generate the synthetic default flag. V2 fits an actual illustrative logistic-regression model using Credit Score, DTI, Delinquency Bucket and Collateral Coverage, then uses fitted model predictions for portfolio PD and EL.

### 3. Stress-test consistency
The Base scenario is explicitly reconciled to the corrected portfolio Expected Loss. Moderate and Severe Stress apply transparent shocks to PD, LGD and EAD.

## Validation
- Loans: 5,000
- Exposure: 752,291,444.01
- EAD: 806,632,235.85
- Default rate: 1.88%
- ROC-AUC: 0.669
- Brier score: 0.018
- Expected Loss: 8,235,782.57
- Maximum EL reconciliation difference: 0.000000000060

## Status
The substantive model logic is stronger and the audit controls explicitly test PD/LGD bounds, positive exposure, missing values and the PD × LGD × EAD reconciliation. It remains a synthetic portfolio model, not a production banking credit model.
