
Telecom Customer Cancellation & Churn Analytics (Homework 5)

This repository contains an end-to-end data preprocessing and exploratory data analysis (EDA) pipeline designed to audit, clean, and profile subscriber metadata to identify critical risk drivers associated with account cancellations.

Team Authors

• Hong Ngoc Dinh
• Nora Cook
• Hmwe Pyant Thin Chit

Project Overview

For subscription-based business models, predicting and preventing customer churn is vital for protecting revenue. However, corporate database exports frequently arrive with structural anomalies, duplicate entries, inconsistent strings, and illegal values that distort performance tracking.
This project implements a reproducible Python data cleaning framework that transforms a messy customer ledger into a type-safe analytical master matrix. By cross-examining demographics, account tenure, financial spend, and support loads, the pipeline isolates high-risk customer segments to guide strategic retention workflows.

Technical Stack & Packages

• Data Manipulation: pandas, numpy
• Data Visualization: matplotlib, seaborn

Data Quality Assessment & Cleaning Pipeline

The raw dataset (HW4_customer_cancellations.csv) initially contained 2,508 records across 10 evaluation columns. Programmatic profiling uncovered several technical data quality leaks, which were resolved across three main engineering layers:

1. Duplicate Mitigation

• Issue: Identified 8 exact duplicate rows where both customer_id and transactional features were identical.
• Fix: Dropped duplicate records using .drop_duplicates(), retaining exactly one unique record per customer.

2. Numerical Anomaly Masking & Imputation

• Financial Leak: Uncovered anomalous negative balances within the monthly_spend array (e.g., -77.39), likely caused by misclassified system refunds or data-entry errors. These were programmatically masked to Not-a-Number (NaN) null pointers to prevent regression skews.
• Biometric Scaling Error: Detected a extreme scale violation in satisfaction_rating where some entries were recorded as 999 instead of staying inside the valid 1–5 range. These out-of-bounds metrics were coerced to NaN variables.
• Missing Value Strategy: The final cleaned dataset contains 3 missing values in monthly_spend, 3 missing values in total_spend, and 6 missing values in satisfaction_rating. Because these gaps affect < 0.3% of the total volume, rows were retained to protect data density.

3. Categorical Text Standardization

• Service Plan Alignment: Capitalized mismatched entries to merge the lowercase string basic into the standard Basic plan domain.
• Payment Method Unification: Consolidated various string formats by mapping text variations like auto pay into a single pythonic categorical label (AutoPay).

Feature Engineering Layer

To support strategic dashboards and corporate reporting, the script constructs four transparent business indicators:
1. Tenure Groups: Discretizes the continuous tenure_months field into six uniform 12-month cohort brackets using pd.cut().
2. Low Satisfaction Vector: Generates a boolean flag (low_satisfaction = True) for any property file indicating a 1 or 2 on the satisfaction scale.
3. New Customer Indicator: Flags accounts with 12 months or less of continuous platform life (tenure_months <= 12).
4. High Service Contact Flag: Flags accounts requiring high support intensity, defined as having 3 or more support interactions (customer_service_contacts >= 3).

Core Exploratory Findings & Business Insights

The final cleaned master matrix consists of 2,500 rows and 14 columns, reflecting an overall baseline cancellation rate of 20.0% (500 churned accounts vs. 2,000 retained).
Grouping the validated data highlights three customer segments with severe risk divergence:

1. The Low-Satisfaction Segment

• Finding: Customers scoring a 1 or 2 on the satisfaction scale exhibit an alarming 41.85% cancellation rate, more than doubling the platform's baseline average.
• Managerial Meaning: Low customer satisfaction is the single strongest indicator of contract risk. This group should be prioritized for immediate service recovery workflows.

2. The New-Customer Cohort (First-Year Churn)

• Finding: Accounts within their first 12 months of service experience a 32.93% cancellation rate. Once a customer crosses the 1-year threshold, risk drops significantly and stabilizes between 15%–19% across older cohorts.
• Managerial Meaning: The first year represents a high-friction onboarding zone. Management must deploy proactive engagement and product-adoption check-ins during the first 180 days to protect customer acquisition costs.

3. The High Support Load Segment

• Finding: Accounts with 0–2 customer-service contacts maintain low cancellation rates (16%–18%). However, once an account hits 3 contacts, the churn rate jumps to 27.74%, climbs to 41.10% at 4 contacts, and peaks at 63.64% at 5 contacts.
• Managerial Meaning: Repeated help-desk outreach is a clear warning sign of customer frustration or unresolved technical friction. Management should establish automated system alerts that trigger retention review workflows as soon as an account logs its third support ticket.

Directory Organization

text
your-repository-root/
│
├── HW4_customer_cancellations.csv   # Original, immutable raw source file
├── Churn_Analysis_Pipeline.ipynb     # Primary Jupyter notebook containing cleaning & EDA code
└── README.md                         # Repository homepage documentation
