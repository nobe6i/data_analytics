
BAN 612: E-Commerce Fulfillment Data Pipeline & AI-Assisted Analytics Audit

This repository houses the end-to-end data validation engineering, text standardization, and exploratory diagnostic pipelines designed to process multi-channel transactional databases.

Team Authors

• Hong Ngoc Dinh
• Nora Cook
• Hmwe Pyant Thin Chit

Project Objective

When corporate strategy teams evaluate operational health (such as product return rates, delivery latency, or customer feedback), relying blindly on automated summaries can lead to flawed management conclusions. Real-world event logs frequently arrive with duplicate indexes, unit-of-measure mismatches, and structural data exceptions.
This project implements a reproducible Python data quality wrapper that transforms raw logistics files into a type-safe master matrix. It features a Human-in-the-Loop Validation Workflow that establishes a clean evidence footprint, uses Generative AI to form testable business hypotheses, and rigorously audits AI claims using Pandas validation queries before finalizing executive recommendations.

Enterprise Data Specifications

The processing stack evaluates a core structural matrix (Project_Data.csv) consisting of 2,205 original rows and 13 variables:
• order_id / customer_id – Primary and foreign relational keys.
• order_date – Historical timestamp tracking purchase cycles.
• channel – Categorical sales indicators (Store, Online).
• product_category – Segment groupings (Electronics, Apparel, Home, Grocery).
• customer_type – Commercial scale brackets (Consumer, Business, Enterprise).
• order_value – Continuous financial transaction size.
• discount_pct – Applied fractional discount rates.
• promised_days / actual_days – Scheduled versus realized shipping intervals.
• shipping_cost – Associated logistical transport fees.
• customer_rating – Direct post-purchase sentiment metrics (1-5 scale).
• returned – Binary logical indicator representing item return status (1 = Returned, 0 = Retained).

Data Cleaning & Quality Engineering

The data engineering layer executes systematic adjustments to fix corporate data anomalies:

1. Structural De-duplication

• Identifies and purges 5 exact duplicate rows where order_id values and financial parameters were identically mirrored, preventing double-counting inflation.

2. Unit-of-Measure & Format Rectification

• The Discount Unit Leak: Uncovers critical input noise in the discount_pct column where select rows recorded values as absolute integers (e.g., 15.0, 20.0) instead of continuous decimals. The pipeline applies conditional masking logic to scale these down (val / 100).
• The Chronological Anomaly: Catches a completely corrupted calendar entry (2025-13-05). Rather than inventing data, this is coerced to a Not-a-Time (NaT) pointer.

3. Outlier Isolation & Value Masking

• Illegal Financial Balances: Isolates rows containing an impossible negative transport balance (shipping_cost = -1.21), converting it to a standard null vector.
• Scale Violations: Detects a system filler flag (999) inside the customer_rating array, systematically casting these 7 corrupted rows to safe NaN pointers to protect statistical calculations from distortion.

Feature Construction Layer

Beyond raw fields, the script constructs operational performance metrics to evaluate corporate friction:
• Shipping Delay Vector (delay_days): Measures realized fulfillment deltas relative to original contract constraints (actual_days - promised_days).
• Fulfillment Performance Status (delivery_status): Applies conditional logic mapping to group records into absolute operational buckets:
\(\text{Delivery\ Status}=\begin{cases}\text{Early},&\text{if\ delay\_days}<0\\ \text{On\ time},&\text{if\ delay\_days}=0\\ \text{Late},&\text{if\ delay\_days}>0\end{cases}\)

The AI-Assisted Validation Workflow

This workflow uses a safe, structured prompting strategy that treats AI outputs strictly as testable hypotheses, which must be cross-checked programmatically with Pandas before being included in executive briefs:
text
  [Raw File Ingestion] ──> Data Quality Assessment ──> Cleaned Evidence Packet
                                                                │
  [Executive Action]   <── [Pandas Audit Verification]  <── AI Hypothesis Engine
Use code with caution.

Prompting Strategy & LLM Constraints

The generative agent is provided with an aggregate profile matrix and restricted by specific guardrails:
1. Must base hypotheses only on the provided data packet.
2. Must isolate Safe Claims (directly provable) from Unsafe Claims (causal leaps without testable controls).
3. Must generate precise programmatic verification parameters.

Programmatic Claim Audit

During the run, the AI generated a plausible correlation heatmap and suggested evaluating return rates across distribution nodes. However, a manual code audit revealed that the AI failed to detect the discount_pct integer scaling error, which would have heavily skewed the correlation baseline. Running a custom Pandas check caught this error, allowing the script to correct the baseline prior to compiling the final report.

Strategic Operational Insights

Analyzing the cleaned database (2,200 rows, 15 columns) reveals three core operational insights:

1. Delivery Latency Strongly Impacts Customer Sentiment

• Descriptive Metric: Approximately 50.0% of all validated orders suffer from post-promise fulfillment delays.
• The Sentiment Drop: Orders arriving early or on-time secure high customer satisfaction averages (4.39 and 4.34 out of 5 respectively). However, as delivery latency increases, sentiment drops sharply, with orders arriving 3 days late falling to a low average rating of 3.02.

2. Electronics Drives High Logistical Return Volume

• Descriptive Metric: The platform maps an overall baseline product return rate of 14.45% (318 returned items out of 2,200 total orders).
• The Category Spike: Electronics represents a severe product drain, with a return rate of 23.29%, followed by Apparel at 15.35%. Conversely, Grocery and Home remain highly stable at 4.15% and 6.79% respectively.

3. Post-Return Dissatisfaction Metrics

• Descriptive Metric: Reviewing customer evaluations based on return state reveals a clear divergence in sentiment.
• The Drop: Retained orders maintain a solid baseline rating of 4.11. Once a transaction triggers a return state, the average rating falls to 3.14, indicating that product issues or returns friction damage brand equity.

Technical Stack & Dependencies

The pipeline runs on the open-source Python scientific computing stack:
bash
pip install numpy pandas matplotlib seaborn
Use code with caution.

Directory Organization

text
your-repository-root/
│
├── Project_Data.csv            # Original, immutable raw source file
├── final_project_pipeline.ipynb # Primary Jupyter notebook containing data wrangling and EDA code
└── README.md                   # Repository documentation homepage
