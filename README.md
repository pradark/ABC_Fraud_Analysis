# ABC Fraud Analysis

Take-home case study: analyze labeled business accounts and transaction history to recommend fraud prevention rules for ABC (Jun–Dec 2022 sample).

## Primary deliverable

**[View the interactive report →](https://pradark.github.io/ABC_Fraud_Analysis/)**

Standalone HTML report with production policy, rule evaluation, charts, and worked case studies.

- **Live report (GitHub Pages):** https://pradark.github.io/ABC_Fraud_Analysis/
- **Direct link:** https://pradark.github.io/ABC_Fraud_Analysis/fraud_analysis.html
- [Source file on GitHub](https://github.com/pradark/ABC_Fraud_Analysis/blob/main/fraud_analysis.html) (code view only — use the links above to read the report)

## What was analyzed

The sample contains **328 businesses** with transaction history (**276 confirmed fraud, 52 good**) and **12,658 transactions** over Jun–Dec 2022. Analysis was done at the **business level** (`is_bad`), with **outgoing spend** (`direction = from`) used for dollar severity.

### Key findings

- Dominant pattern: **short-lived, card-funded bust-out** — ACH funding in, heavy spend on high-risk MCCs (apparel, quasi-cash, prepaid-style swipes), cash out within days.
- Top signal categories: **clothing/accessory MCCs**, **postal / quasi-cash MCCs** (4829, 6051, 9402), **contactless + AVS-fail** card-not-present abuse, **day-0 velocity**, and **returned ACH**.
- Recommended **11-rule production policy** (OR logic):
  - **Tier 1 — Immediate (5 rules):** early velocity, bust-out, gift-card-style swipes, contactless + AVS-fail, international decline attempts.
  - **Tier 2 — Confirm (6 rules):** quasi-cash MCC + high ticket, apparel cluster, international settled spend, Sardine very-high decline, returned ACH.
- Sample performance: **~89.8% precision, ~92.4% recall**, 29 false positives.

### Approach

1. **Exploratory notebook** (`ABC_Risk.ipynb`) — EDA, business-level bad rates, IV/WOE feature ranking, decision tree (test AUC ~0.89), first hand-built rules.
2. **Hypothesis-driven probing** — deep dives on postal vs. apparel vs. velocity patterns; severity-ranked MCC analysis; end-to-end case reconstructions.
3. **Production rule library** — 37 atomic rules evaluated; curated 11-rule stack for automation at scale.

## Files in this repository

| File | Description |
|------|-------------|
| [fraud_analysis.html](fraud_analysis.html) | **Main deliverable.** Full analysis report: data description, approach, production decision tree, rule glossary, money-flow chart, case studies. |
| [ABC_Risk.ipynb](ABC_Risk.ipynb) | Jupyter notebook with exploratory analysis, feature engineering, IV/WOE ranking, decision tree, and initial rules. |
| [Instructions.pdf](Instructions.pdf) | Original assignment brief (company name anonymized to ABC). |
| [businesses.csv](businesses.csv) | Labeled businesses: `business_token`, `is_bad`. |
| [transactions.csv](transactions.csv) | Transaction ledger: entry type, direction, amount, status, MCC, card/ACH fields, vendor. |

## Note on data

Five businesses have an `is_bad` label but no rows in `transactions.csv` (4 fraud, 1 good). They are excluded from rule evaluation because there is no activity to score.
