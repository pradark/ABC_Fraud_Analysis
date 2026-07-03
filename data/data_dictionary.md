# Data dictionary

Field definitions for the CSV files in this folder. Observation window: **Jun–Dec 2022**.

## `businesses.csv`

One row per business. This file is the **label source** for fraud analysis.

| Field | Type | Description |
|-------|------|-------------|
| `business_token` | string | Unique anonymized business identifier. Join key to `transactions.csv`. |
| `is_bad` | boolean | `TRUE` / `FALSE` — whether the business was determined to be fraudulent (`1` = bad, `0` = good in analysis). |

**Note:** 328 businesses are labeled here; 323 appear in `transactions.csv` (5 labeled businesses have no transaction rows).

---

## `transactions.csv`

One row per transaction. **12,658 rows**, 20 columns.

### Core fields (all entry types)

| Field | Type | Description |
|-------|------|-------------|
| `business_token` | string | Unique business identifier; joins to `businesses.csv`. |
| `transaction_date` | timestamp | When the transaction occurred. Used for account span, day-0 velocity, and sequencing. |
| `entry_type` | string | Channel / rail. See values below. |
| `direction` | string | Fund flow relative to the ABC account: `to` = inbound, `from` = outbound. |
| `amount_cents` | integer | Positive amount in USD cents (divide by 100 for dollars). |
| `transaction_status` | string | Final status. Common values: `SETTLED`, `DECLINED`, `RETURNED`, `VOIDED`, `CANCELED`. |
| `internal_failure_reason` | string | Why a **card** transaction was declined (empty if not applicable). Examples: `INSUFFICIENT_FUNDS`, `NO_DOMESTIC_CARD_PRESENT_TRANSACTIONS`, `SARDINE_DECLINE_VERY_HIGH`, `LIMIT_EXCEEDED`. |
| `vendor_name` | string | Counterparty name (merchant on card txns; hashed/tokenized on some ACH rows). |

### `entry_type` values

| Value | Description |
|-------|-------------|
| `card` | ABC debit card purchase |
| `ach_external` | ACH transfer initiated by an external financial institution |
| `ach_internal` | ACH transfer initiated by ABC |
| `atm` | ABC debit card ATM withdrawal |
| `check` | Check deposit |
| `linkfound_to_found_transfer` | Transfer between two ABC accounts |
| `instant_debit` | Account funding via an external debit card |

### Card-only fields

Present on card (and some ATM) transactions; otherwise typically empty.

| Field | Type | Description |
|-------|------|-------------|
| `mcc` | integer | ISO merchant category code (e.g. `4829` money orders/wire, `9402` postal, `5621` clothing). |
| `mcc_description` | string | Human-readable MCC label. |
| `merchant_country` | string | Merchant country (e.g. `USA`, `POL`). Drives international-fraud rules. |
| `pos_data_type` | string | Point-of-sale type (e.g. `POS_TERMINAL`, `ECOMMERCE`, `ATM`, `PHONE`). |
| `token_wallet_type` | string | Digital wallet if tokenized (e.g. `APPLE_PAY`); empty otherwise. |
| `pos_data_attended` | boolean | Whether the POS was attended at transaction time. |
| `pos_data_on_premise` | boolean | Whether the POS was on-premise. |
| `card_entry_mode` | string | Card present vs not: `PRESENT`, `NOT_PRESENT`. |
| `pan_entry_mode` | string | How card data was entered: `CONTACTLESS`, `ICC`, `KEY_ENTERED`, `MAGNETIC_STRIPE`, `UNKNOWN`, etc. |
| `pin_entered` | boolean | Whether PIN was entered. |
| `card_network` | string | Card network (e.g. `MASTERCARD`, `MAESTRO`, `INTERLINK`). |
| `avs_result` | string | Address verification result: `FAIL`, `MATCH`, `MATCH_ZIP_ONLY`, etc. |

### Analysis conventions used in this project

- **Business-level labels:** `is_bad` from `businesses.csv` — a business is evaluated once, not per transaction.
- **Outgoing spend:** `direction = from` for dollar severity and bust-out patterns.
- **Incoming funding:** `direction = to` for ACH-in and deposit analysis.
- **Returned ACH:** `entry_type = ach_external`, `direction = to`, `transaction_status = RETURNED` — issuer clawback signal.
