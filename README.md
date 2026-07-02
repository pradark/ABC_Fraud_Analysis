# ABC Fraud Analysis

Fraud risk analysis on business banking accounts using labeled transaction history to design interpretable prevention rules. Covers pattern detection, dollar-severity ranking, and a two-tier production policy for ABC (Jun–Dec 2022).

**[Interactive report →](https://pradark.github.io/ABC_Fraud_Analysis/)**

## Overview

The dataset includes **328 businesses** with transaction history (**276 confirmed fraud, 52 good**) and **12,658 transactions** over Jun–Dec 2022. All metrics are computed at the **business level** (`is_bad`), with **outgoing spend** (`direction = from`) used for dollar severity.

### Findings

- Dominant pattern: **short-lived, card-funded bust-out** — ACH funding in, heavy spend on high-risk MCCs (apparel, quasi-cash, prepaid-style swipes), cash out within days.
- Strongest signals: **clothing/accessory MCCs**, **quasi-cash MCCs** (4829, 6051, 9402), **contactless + AVS-fail** abuse, **day-0 velocity**, and **returned ACH**.
- **11-rule production policy** (OR logic):
  - **Tier 1 — Immediate (5 rules):** early velocity, bust-out, gift-card-style swipes, contactless + AVS-fail, international decline attempts.
  - **Tier 2 — Confirm (6 rules):** quasi-cash MCC + high ticket, apparel cluster, international settled spend, Sardine very-high decline, returned ACH.
- Policy performance on the labeled sample: **~89.8% precision, ~92.4% recall**, 29 false positives.

### Method

1. **Exploratory notebook** (`ABC_Risk.ipynb`) — EDA, business-level bad rates, IV/WOE feature ranking, decision tree (test AUC ~0.89), initial rule prototypes.
2. **Pattern deep-dives** — postal vs. apparel vs. velocity; severity-ranked MCC analysis; end-to-end transaction reconstructions.
3. **Rule library** — 37 atomic rules evaluated; curated 11-rule stack for automated deployment.

## Repository contents

| File | Description |
|------|-------------|
| [docs/fraud_analysis.html](docs/fraud_analysis.html) | Interactive HTML report: approach, production decision tree, rule glossary, money-flow chart, worked examples. |
| [ABC_Risk.ipynb](ABC_Risk.ipynb) | Jupyter notebook — exploratory analysis, feature engineering, IV/WOE ranking, decision tree, initial rules. |
| [businesses.csv](businesses.csv) | Labeled businesses: `business_token`, `is_bad`. |
| [transactions.csv](transactions.csv) | Transaction ledger: entry type, direction, amount, status, MCC, card/ACH fields, vendor. |

## Data note

Five businesses have an `is_bad` label but no rows in `transactions.csv` (4 fraud, 1 good). They are excluded from rule evaluation because there is no activity to score.
