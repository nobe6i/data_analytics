
AI-Assisted SaaS Customer Risk Matrix and Upsell Pipeline (Week 7 Case Study)

This repository contains a reproducible, type-safe data engineering and predictive risk modeling pipeline built to evaluate B2B software-as-a-service (SaaS) contract risks and expansion pipelines.

Table of Contents

• Project Overview
• Enterprise Data Architectures
• Data Engineering & Ingestion Grain
• Constructed Heuristic Risk Vector
• Strategic Action Segments
• The AI-Assisted Validation Workflow
• Executive Churn vs. Upsell Findings
• Dependencies

Project Overview

In subscription business models, tracking customer churn and expansion opportunities requires processing highly fragmented inputs. Metrics span unstructured account text notes, support tickets, periodic product server hit counts, and invoice cycles.
Rather than deploying automated black-box machine learning or blindly delegating strategic decisions to generative AI, this repository implements a robust, human-in-the-loop validation pipeline. The analyst generates clean customer-level evidence packets, uses generative AI to form testable business hypotheses, and executes strict programmatic checks with Pandas to verify claims before generating management recommendations.

Enterprise Data Architectures

The analytics engine safely pulls event details across six relational files:
• customers.csv – Base demographic company registry documenting 140 firmographic observations.
• subscriptions.csv – 158 contract matrices tracking recurring revenue states and plan tiers.
• invoices.csv – 1,142 financial actions tracking balance dues, collections, and delays.
• weekly_usage.csv – 5,928 row-by-row tracking parameters mapping logins and API usage.
• support_tickets.csv – 300 customer cases capturing help-desk resolution times and CSAT scores.
• account_notes.csv – Unstructured qualitative summary notes logged by account managers.

Data Engineering & Ingestion Grain

To make the data ready for both AI indexing and human validation, raw event transactions are rolled up into an immutable Customer 360 analytical master index (one record per customer_id).

Strategic Preprocessing Steps

• Chronological Synchronization: Text date series are systematically parsed using pd.to_datetime(errors="coerce"), forcing broken time strings into safe NaT structures.
• Usage Delta Calculation: Implements rolling window aggregates to isolate sudden user engagement shifts by calculating the change percentage between the latest 8 weeks of platform logins and the preceding 8-week baseline (login_change_pct).
• Null Imputation Boundary Rules: Financial variables (total_billed), hit parameters (avg_logins), and text notes are filled with 0 where missing entries represent true inactivity. In contrast, performance factors like avg_csat and resolution times retain true null values to avoid skewing tracking dashboards.

Constructed Heuristic Risk Vector

To build a clear, auditable tracking metric, accounts are assigned a baseline Heuristic Risk Score (0 to 100) based on five operational business signals:
1. Usage Drop (+25 points): Triggered if an account's recent login activity falls more than 25% below its previous baseline (login_change_pct < -25).
2. Financial Delinquency (+20 points): Imposed if the customer holds one or more outstanding unpaid invoices.
3. Low Platform Satisfaction (+20 points): Added if help-desk feedback maps to an average customer satisfaction score under 3.5 (avg_csat < 3.5).
4. Heavy Account Support Strain (+15 points): Triggered if high-priority help-desk support submissions exceed 2 or more instances.
5. Account Manager Warning (+20 points): Applied if qualitative account manager notes contain explicit keywords indicating active account threat profiles (risk_notes > 0).

Strategic Action Segments

Accounts are grouped into management buckets based on their Value Score (the percentile rank of their contract MRR) and their Risk Score to streamline customer success workflows:
text
                  MANAGEMENT ACTION MAP (CUSTOMER 360)
     100 ┌──────────────────────────────────┬──────────────────────────────────┐
         │                                  │                                  │
         │         Monitor Segment          │         Expand Carefully         │
         │                                  │       (High Value / Low Risk)    │
  Value  │                                  │                                  │
  Score  ├──────────────────────────────────┼──────────────────────────────────┤
 (MRR %) │                                  │                                  │
         │         Monitor Segment          │             Save Now             │
         │                                  │      (High Value / High Risk)    │
         │                                  │                                  │
       0 └──────────────────────────────────┴──────────────────────────────────┘
         0                                 50                                100
                                       Risk Score
Use code with caution.
• Save now: High-value enterprise accounts (MRR ≥ 70th percentile) carrying severe operational threat indices (Risk ≥ 50).
• Expand carefully: High-value enterprise accounts (MRR ≥ 70th percentile) exhibiting stable usage and minimal risk signals (Risk < 50).
• Scaled retention: Lower-value, high-risk accounts (MRR < 70th percentile, Risk ≥ 50) managed via automated, group-level success loops.
• Monitor: Default baseline status representing low-risk, standard accounts.

The AI-Assisted Validation Workflow

This workflow enforces a strict security wall between data exploration and executive decision-making.
text
 [Customer 360 Master] ──> Extract Focus Subset ──> Prompt Engineering ──> Generative AI Draft
                                                                                 │
 [Pandas Validation]   <──  Audit Claims Matrix <──   Isolate AI Claims   <──────┘
Use code with caution.
1. The Evidence Packet: The script exports a highly summarized, anonymized profile matrix of at-risk accounts to the LLM environment, preventing data leakage.
2. The Hypothesis Generation: The AI processes the structured table to highlight risk patterns, draft retention strategies, and identify potential expansion opportunities.
3. Programmatic Audit (Crucial Step): Rather than copying the AI draft directly into reports, the analyst runs the AI claims through boolean verification scripts:python
# Programmatic audit verifying if AI recommendations match true database parameters
claim_audit = bool(((customer360.customer_id == "C1041") & (customer360.value_score >= 70) & (customer360.risk_score >= 50)).any())
Use code with caution.
4. Executive Output: The verified insights are compiled into an analyst-backed management table, and unverified AI claims are flagged as hallucinations and discarded.

Executive Churn vs. Upsell Findings

Running the validated pipeline across the account workspace isolates top corporate profiles requiring immediate administrative review:

Validated Urgent Retention Targets (Save now)

• Account C1108 (Evergreen Partners 108): An enterprise-tier account with an exceptionally high contract value (MRR: $1,712.17, 88th percentile value score). Despite active product usage (136 weekly logins), it registers a high risk index of 60 driven by critical payment collection friction (1 unpaid invoice), poor help-desk sentiment (3.33 CSAT), and a documented account manager risk note.
• Account C1019 (Summit Labs 19): A high-value business client (MRR: $710.42) that has triggered severe churn risk indicators (Risk Score: 55). Activity tracking reveals a steep 24.9% decline in active logins, alongside an escalating support burden consisting of 6 help-desk tickets, 2 high-priority issues, and poor satisfaction scores (3.33 CSAT).

Validated Upsell Opportunities (Expand carefully)

• Account C1069 (Civic Group 69): Represents the highest-value active contract in the dataset (MRR: $2,231.81). The account demonstrates stellar health indicators: a healthy 7.58% login expansion delta, solid platform satisfaction (4.25 CSAT), and a documented account manager upsell flag (upsell_notes > 0).
• Account C1032 (Nimbus Analytics 32): A top-tier enterprise account (MRR: $2,110.12) displaying stable, consistent usage profiles (-4.7% login change), a perfect platform satisfaction record (5.00 CSAT), zero unpaid invoices, and a flagged upsell discovery note.

Dependencies

The analytical transformation stack runs within Python kernels matching the following minimum requirements:
bash
numpy >= 1.24.0
pandas >= 1.5.0
matplotlib >= 3.6.0
