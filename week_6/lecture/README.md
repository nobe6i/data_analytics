
B2B SaaS Customer Experience & Exploratory Churn-Risk Data Matrix (Week 6)

This repository houses the exploratory data analysis (EDA), segment profiling, and risk indexing pipelines built on top of the customer-month master tables.

Table of Contents

• Project Overview
• EHR Inbound Data Specifications
• Exploratory Analysis & Core Questions
• Customer Behavioral Value Segmentation
• Exploratory Risk Index Framework
• Managerial Reporting: Executive Risk Summary Matrix
• Dependencies

Project Overview

While data engineering and row-pivoting establish the structural grain of a database, Exploratory Data Analysis (EDA) translates those raw pipelines into operational evidence.
The primary objective of this project is to build an auditable analytics script that visualizes Monthly Recurring Revenue (MRR) distributions, evaluates product usage spikes over time, profiles customer support burdens, and develops an interpretable Exploratory Churn Risk Index to prioritize customer success follow-ups.

EHR Inbound Data Specifications

The analysis framework cross-examines data across five master files:
• customers.csv – Firmographic customer attributes tracking 140 unique corporate observations.
• subscriptions.csv – 158 contract history tiers tracking billing cycles and active values.
• weekly_usage.csv – 5,928 row-by-row event logs logging user activity flags.
• invoices.csv – 1,142 billing actions documenting collection history and payment lags.
• support_tickets.csv – 300 support cases capturing help-desk priority tiers and satisfaction scores.

Exploratory Analysis & Core Questions

The EDA notebook evaluates five specific relationship dynamics to support data-backed business decisions:

1. Revenue Skew & Concentration

• Univariate Audit: Analyzes the concentration of standard Monthly Recurring Revenue (MRR).
• Finding: Metrics show highly right-skewed revenue properties (the mean is significantly higher than the median). A small tier of high-value accounts generates most of the platform's recurring revenue.

2. Time-Series Volatility

• Trend Tracking: Tracks monthly login and active user curves.
• Finding: Product usage exhibits predictable mid-quarter spikes followed by end-of-quarter dips, which must be accounted for before flagging real account drops.

3. Usage vs. Financial Value

• Correlation Evaluation: Uses scatterplots to evaluate active user counts against monthly revenue streams.
• Finding: While there is a strong positive correlation between higher tier spend and usage volume, a small segment of expensive accounts remains completely inactive, highlighting significant immediate contract risk.

4. Support Interaction Load

• Service Burden Audit: Normalizes ticket volume against product usage frequency to calculate an exact support load metric:
\(\text{Support\ Load}=1000\times \left(\frac{\text{Help-Desk\ Ticket\ Count}}{\text{Total\ Monthly\ Logins}}\right)\)
• Finding: Higher-value enterprise plans produce more support tickets overall, but their usage-adjusted support load is much lower than entry-tier accounts, which create heavy support noise relative to their platform activity.

Customer Behavioral Value Segmentation

To shift from general averages to targeted customer insights, accounts are bucketed into a 3x3 classification grid based on their financial tier (MRR terciles) and product activity level (average logins per active user):
text
                  CUSTOMER COUNT SEGMENT MATRIX
   High Value  │      [ 14 ]      │      [ 15 ]      │      [ 18 ]      │
               ├──────────────────┼──────────────────┼──────────────────┤
    Mid Value  │      [ 18 ]      │      [ 12 ]      │      [ 16 ]      │
               ├──────────────────┼──────────────────┼──────────────────┤
    Low Value  │      [ 15 ]      │      [ 19 ]      │      [ 13 ]      │
               └──────────────────┴──────────────────┴──────────────────┘
                  Low Engagement     Mid Engagement     High Engagement
Use code with caution.

Exploratory Risk Index Framework

Rather than rushing into black-box modeling, the pipeline builds a transparent, multi-variable Exploratory Risk Score (0 to 9) to flag accounts needing customer success intervention:
• Low Engagement Intensity (+2 points): Assigned if an account's average usage falls into the lowest 25% of the platform baseline.
• Payment Collection Friction (+2 points): Assigned if the invoice collection rate drops below 98% (payment_realization < 0.98).
• Negative Help-Desk Review (+1 point): Triggered if an account's average customer satisfaction score drops below a 3.5 rating (avg_csat < 3.5).
• Heavy Support Burden (+1 point): Added if total help-desk ticket submissions land in the highest 25% of the platform volume.
• Contract Cancellation (+3 points): Imposed if the primary subscription state changes to Cancelled.

Managerial Reporting: Executive Risk Summary Matrix

Grouping the final dataset by value and engagement segments provides leadership with a clear view of platform revenue risks:
text
   value_segment    engagement_segment  customers  median_mrr  avg_risk_score  avg_csat  median_payment_realization
0     High value       High engagement         18      310.25        1.250000  4.120000                        1.00
1     High value        Mid engagement         15      289.20        2.133333  3.800000                        1.00
2     High value        Low engagement         14      253.28        4.784313  2.950000                        0.88
3      Low value        Low engagement         15       89.45        5.157303  2.120000                        0.82
Use code with caution.

Risk Audit Findings

• The High-Value Danger Zone (Row 2): Identifies 14 key corporate accounts flagged as high-revenue but low-engagement. They average an elevated risk score of 4.78, low satisfaction (2.95 CSAT), and poor financial collection rates (88% payment realization). This group represents the primary revenue risk for the business.
• The Low-Value Support Drain (Row 3): Entry-level, low-engagement accounts generate the worst satisfaction scores (2.12 CSAT) and require disproportionate help-desk attention, suggesting that simple onboarding changes could reduce this operational burden.

Dependencies

The analytics stack runs within Python environments using standard scientific computing tools:
bash
numpy >= 1.24.0
pandas >= 1.5.0
matplotlib >= 3.6.0
Use code with caution.
