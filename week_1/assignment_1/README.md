# BAN 612: From Raw Data to Trustworthy Evidence (Assignment 1)

This repository houses the Python notebooks, data cleaning scripts, and structural quality assurance configurations for the initial lecture demo and Assignment 1 processing pipelines. 

## Project Objective
Real-world enterprise data is rarely analysis-ready. It arrives with structural bugs, format variations, input noise, and broken relational keys. The primary objective of this project is to implement robust, programmatically auditable data preprocessing steps that transition raw datasets into validated, trustworthy analytical environments.

---

## Dataset Architectures & Raw Input Problems

The data pipeline processes four core transactional and metadata tables:

### 1. Messy Orders Table (`orders_raw`)
*   **Attributes:** `order_id`, `customer_id`, `order_date`, `amount`, `channel`, `state`.
*   **Identified Issues:** Extraneous string variables (`$`, `,`), unparsed multi-format date metrics (e.g., `8/2/2026`, `2026/08/03`, `Aug 4 2026`), mixed casing variations (`online`, `STORE `), duplicate keys, and critical missing entries or illegal negative balances.

### 2. Customers Table (`customers`)
*   **Attributes:** `customer_id`, `customer_name`, `age`, `signup_date`, `segment`, `email`, `state`.
*   **Identified Issues:** Input string inconsistencies (`SMALL BUSINESS`, `consumer `), null fields across critical customer dimensions (`age`, `email`), text-mapped chronological records, and significant biometric anomalies (e.g., a customer record showing an age of `220.0`).

### 3. Store Metadata Table (`store`)
*   **Attributes:** `store_id`, `store_name`, `city`, `state`, `region`, `square_feet`, `opened_date`.
*   **Identified Issues:** String fields mismapped as structural objects, empty string indices within categorical columns (missing regions), and comma-formatted integers mimicking text parameters.

### 4. Transactions Ledger Table (`transactions`)
*   **Attributes:** `transaction_id`, `customer_id`, `store_id`, `transaction_date`, `product_category`, `quantity`, `unit_price`, `discount_pct`, `payment_method`, `state`.
*   **Identified Issues:** Structural foreign key deviations (e.g., padding mismatches in customer strings relative to the `customers` dimensional master), row duplication, and missing entries.

---

## Data Cleaning & Quality Engineering Pipeline

The data engineering pipeline performs programmatic fixes across three foundational layers:

### Layer 1: Structural Cleanse & Cast
*   **Text Stripping & Standardizing:** Strips trailing whitespaces and enforces systematic capitalization protocols (`.str.strip().str.title()`, `.str.upper()`) across all channel and region categories.
*   **Numerical Correction:** Eliminates custom currency regex variables (`$`, `,`), maps blank strings to systemic null pointers (`np.nan`), and casts object series to continuous floats or integers (`pd.to_numeric`).
*   **Chronological Standardization:** Parses inconsistent multi-format text values into formal timestamp indices using `pd.to_datetime`.

### Layer 2: Business Rule Validation & Outlier Mitigation
*   **Audit Flag Generation:** Generates multi-column logical validation tables to explicitly track structural bugs (`duplicate_order_id`, `missing_amount`, `negative_amount`) rather than silently deleting rows.
*   **Demographic Boundaries:** Employs relational bounding arrays to purge non-viable age metrics, keeping only valid adult customers between the ages of 18 and 80.
*   **Primary/Foreign Key Alignment:** Normalizes inconsistent index tracking metrics (e.g., matching short `C001` keys to standard padded `C0001` fields) using string zero-padding (`zfill(4)`), enabling secure database joins.

### Layer 3: Relational Verification & Merging
*   **Referential Integrity Checks:** Implements strict relational constraint validation checks during tabular merges (`validate="many_to_one"`) to guard against structural explosions.
*   **Mismapped Join Audits:** Leverages programmatic join indicators (`indicator=True`) to run validation aggregations, quickly highlighting broken orphan keys that fail to cross-reference with master dimensions.

---

## Technical Stack & Dependencies

The processing stack is executed within Python-based Jupyter Notebooks using standard libraries:

```bash
numpy >= 1.24.0
pandas >= 1.5.0
matplotlib >= 3.6.0
seaborn >= 0.12.0
```
