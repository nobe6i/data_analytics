
B2B SaaS Customer-Month Analytics and Churn-Risk Data Pipeline

This repository houses the end-to-end data transformation and feature engineering pipeline designed to convert raw transactional event logs into a type-safe, analysis-ready customer-month master table.

Table of Contents

• Project Overview
• EHR Inbound Tables
• Wrangling Decisions & Processing Grain
• Constructed Business Features
• Data Pipeline Architecture
• Managerial Reporting: Customer Churn Anomaly Risk Audit
• Dependencies

Project Overview

For subscription-based software-as-a-service (SaaS) businesses, tracking churn risk and product health requires looking at customer signals over time. However, operational metrics like weekly server hits, invoice events, and help-desk tickets arrive at varying intervals and granularities.
This pipeline acts as a data engineering layer. It cleans outbound tables, reshapes dimensional matrices, summarizes transactional histories, and aligns all metrics to a shared customer-calendar-month unit of analysis. The output is a robust table optimized for predictive modeling and executive dashboards.

EHR Inbound Tables

The data pipeline safely ingests and cross-examines six structural business tables:
1. Customer Master (customers.csv): Inbound firmographic profiles tracking unique customer_id rows alongside industry tags, corporate size brackets, regional blocks, and original marketing acquisition pathways.
2. Plan Subscriptions (subscriptions.csv): Dimensional contract register capturing starting timestamps, termination markers, contract types (Monthly vs. Annual), Monthly Recurring Revenue (mrr), and historical upgrade/churn indicators.
3. Financial Invoices (invoices.csv): Transactional billing logs tracking unique invoice IDs, total amounts due, balances received, payment dates, discount promotional codes, and processing refund amounts.
4. Weekly Product Usage (weekly_usage.csv): Granular tracking table tracking active account users, monthly system logins, generated metric reports, and external API requests.
5. Help-Desk Support Tickets (support_tickets.csv): Customer service ledger documenting ticket categories, organizational priority flags, operational resolution deltas, and customer satisfaction (csat) scores.
6. Usage Spreadsheets (usage_wide_demo.csv): Cross-tabulated legacy spreadsheet layout used to test programmatic unpivoting arrays (.melt()).

Wrangling Decisions & Processing Grain

To prevent relational duplication and inflated financial counts, every source table is systematically transformed to a standardized customer-month grain (one record per customer_id per calendar month).

Time Horizon Boundaries

• Active Boundary: Evaluates continuous data points from January 2025 through December 2025.
• Chronological Standardization: Converts date strings into clear, chronological blocks using .dt.to_period('M') to strip day/time variance and make database aggregations more reliable.
• The Skeleton Safeguard: To ensure inactive accounts are not dropped from risk metrics, the pipeline creates an explicit customer_month_skeleton using cross-joins (pd.period_range). This grid serves as the master merge index for all metrics, guaranteeing that zero-activity months are caught.

Constructed Business Features

The pipeline builds targeted indicators by applying logic across multiple variables:
• Customer Lifetime Age: Computes current operational age (customer_age_months) by measuring the chronological distance from the sign-up date to the active tracking month.
• Service Interaction Intensity: Normalizes support tickets against usage frequency to create a standardized scale:
\(\text{Support\ Intensity}=100\times \left(\frac{\text{Monthly\ Ticket\ Count}}{\text{Monthly\ Logins}}\right)\)
• The Churn-Risk Engagement Flag: Dynamically sets a boolean flag (low_engagement=True) if an account registers fewer than 10 logins in a month, excluding early-stage onboarding accounts under 2 months old.
• The Payment Delinquency Risk Flag: Evaluates financial collections and locks a high-risk indicator (payment_risk=True) if the payment ratio drops below 80% (pct_paid < 0.8) or if the average payment latency exceeds a 15-day grace period.

Data Pipeline Architecture

The transformation architecture uses Python method chaining to build readable, end-to-end code flows:
text
 Inbound Event Logs       Grain Standardization            Analytical Join             Type Coercion
   [Invoices.csv]    ──>   .groupby('invoice_month')  ──┐
   [Weekly_Usage]    ──>   .groupby('usage_month')    ──┼─> [Customer_Month Skeleton] ─> .fillna(Clear Rules)
   [Support_Tickets] ──>   .groupby('ticket_month')   ──┘
Use code with caution.

Strategic Null Value Rules

Replacing missing observations requires careful business judgment rather than blind automation. The pipeline applies targeted imputation rules:
• True Zero Alignments: Missing logins, zero-dollar invoices, or empty help-desk columns are safely set to 0 since a missing entry indicates no activity occurred.
• Biometric Preservations: Average satisfaction (avg_csat) and payment latency (avg_days_to_pay) columns retain true nulls (NaN). Imputing missing values with zero would distort performance scores by indicating zero satisfaction or instantaneous payments.

Managerial Reporting: Customer Churn Anomaly Risk Audit

Aggregating metrics across the validated customer-month master table reveals structural behavior patterns between corporate account segments over time:
text
      month             size_segment  customers  avg_logins  pct_low_engagement  pct_payment_risk  total_net_collected
20  2025-11  Mid-Market / Enterprise         51  189.215686            0.156863          0.058824             25723.15
21  2025-11                      SMB         89  159.123596            0.168539          0.033708             31680.83
22  2025-12  Mid-Market / Enterprise         51  273.274510            0.156863          0.058824             21537.82
23  2025-12                      SMB         89  186.617978            0.179775          0.011236             31870.27
Use code with caution.

Risk Audit Takeaways

• The Small-Business Churn Divergence: Small and medium-sized business (SMB) accounts consistently exhibit higher engagement risk compared to Enterprise accounts. By December, low engagement peaks at 17.97% within SMB spaces, compared to 15.68% in Enterprise circles.
• The Late-Quarter Revenue Drift: While mid-market engagement rates stabilized or improved in December (averaging 273 logins), their total monthly revenue collections dropped from $25.7k to $21.5k, indicating potential contract renegotiations or invoicing friction.

Dependencies

The processing stack runs inside a Python environment configured with the following package baselines:
bash
numpy >= 1.24.0
pandas >= 1.5.0
openpyxl >= 3.0.0  # Required for processing inbound Excel data files
Use code with caution.
