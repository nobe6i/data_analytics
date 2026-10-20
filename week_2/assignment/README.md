
EHR Clinic Scheduling Analytics & Performance Audit (Homework Assignment 2)

This folder contains a reproducible data quality and validation pipeline engineered to audit multi-clinic medical group scheduling logs. The primary workflow isolates, logs, and resolves system anomalies to produce data credible enough for strategic business decision-making.

Team Authors

• Hong Ngoc Dinh
• Nora Cook
• Hmwe Pyant Thin Chit

Project Objective

Before clinic management can reliably evaluate healthcare parameters—such as patient no-show behavior, telehealth utilization, or provider wait times—the source Electronic Health Record (EHR) data must be validated. Raw transactional streams from merged scheduling platforms frequently contain dirty fields, broken referential records, and impossible chronological events.
This project implements an automated validation wrapper that cleans system variables, enforces cross-column constraints, and outputs a permanent Data Quality Issue Log alongside a comparative Provider Performance Audit.

EHR Data Architectures & Ingestion

The auditing engine simultaneously ingests and mirrors data across three structural medical files:
1. Clinic Appointments (clinic_appointments.csv): Transactional table tracking appointment_id, patient/provider reference strings, booking times, and visit state descriptors.
2. Patient Master (clinic_patients.xlsx): Relational master file cataloging unique patient keys (patient_id), dates of birth, assigned sex codes, insurance carriers, and geographic ZIP codes.
3. Provider Directory (clinic_providers.json): System directory mapping unique provider keys (provider_id) alongside primary physician specialties and clinic locations.

Data Cleaning & Quality Remediation

The pipeline executes systematic fixes across three main architectural layers:

Layer 1: Structural Cleansing & Normalization

• Whitespace & Case Optimization: Strips surrounding string noise using .str.strip() and normalizes variables down to uniform lower case profiles to facilitate exact key matching.
• Categorical Harmonization: Unifies historical variations in status indicators (no-show, No Show, and NS are collapsed into a single No Show domain) and maps conflicting telehealth text values (Y, y, yes) to absolute categorical labels.

Layer 2: Chronological Verification

• Datetime Parsing: Converts multi-format textual objects into true timestamps using pd.to_datetime(errors='coerce', format='mixed').
• Malformed Entry Trapping: Isolates corrupt chronological structures (such as 2026-13-40 25:00) by forcing them to Not-a-Time (NaT) null pointers to prevent runtime calculation exceptions.

Layer 3: Constraint Auditing & Referential Key Padding

• Database Key Integrity: Enforces relational constraint validation using string padding matching algorithms (zfill(4)), systematically correcting shorter patient index markers (e.g., matching a short string like C1 or C001 to a standard padded master label like C0001).

Multi-Column Business Rule Architecture

Enterprise-grade validation requires auditing intersecting logic dependencies across distinct fields. The workspace evaluates nine absolute clinical business rules:
1. Invalid Datetime Stamp: Flags raw chronological values that completely fail standard date-time parsing parameters.
2. Duplicate Key Verification: Isolates instances of identical appointment_id keys assigned across distinct patient events.
3. Patient Referential Integrity: Flags transactions containing a patient_id (e.g., P9999) that does not map to any profile within the Master patient register.
4. Provider Referential Integrity: Identifies rows referencing a doctor ID (e.g., D299) missing from the official Master Provider directory.
5. Booking Chronology Verification: Traps erroneous transactions where the internal booking timestamp (booked_at_dt) occurs after the scheduled appointment block (appointment_at_dt).
6. Visit Duration Check: Cross-references checkout stamps against check-in stamps to expose impossible negative clinic durations.
7. Conditional Completeness Rule: Isolates files flagged as Completed that lack mandatory clinical check-in or checkout activity stamps.
8. Cancelled Activity Validation: Flags cancelled appointments that incorrectly contain active check-in timestamps.
9. No-Show Activations Check: Flags records marked as a No Show that contradictory contain active checkout check-in activity times.

Data Anomaly Issue Log

Rather than silently dropping anomalous records, the pipeline aggregates all system violations into a standardized, exportable data quality report (clinic_data_quality_issue_log.csv) detailing the violation type, count of affected records, and the recommended system fix:

rule	count	action
Invalid appointment datetime	1	Investigate / correct source
Duplicate appointment ID	2	Investigate duplicates
Unknown patient ID	1	Resolve against patient master
Unknown provider ID	1	Resolve against provider master
Booking after appointment	1	Flag as chronology error
Checkout before check-in	1	Flag as chronology error
Completed visit missing timestamp	1	Investigate incomplete workflow
Cancelled visit with check-in	1	Investigate status/timestamp conflict
No Show visit with visit time	2	Investigate / correct the status


Operational Impact: Metric Performance Distortion Audit

To verify the critical nature of data-cleansing protocols prior to constructing operational dashboards, a parallel No-Show Rate Performance Audit was run comparing Raw (Dirty) Data against the Validated (Cleaned) Data.

Analytical Parameters

• The Metric Definition: A no-show represents an appointment where the patient failed to attend.
• The Denominator Split: Formally includes only eligible appointments (Completed + No Show).
• Exclusions: Explicitly filters out Cancelled records, as well as appointments containing internally contradictory information (the 5 invalid records flagged above with impossible chronology or corrupted dates), to protect metric calculations from distortion.

Results: No-Show Rank Divergence

Recomputing performance on the validated subset dramatically changes provider rankings. This proves that evaluating performance using uncleaned data can lead to inaccurate management conclusions:
text
status_standard  raw_no_show_rate  validated_no_show_rate  raw_rank  validated_rank  rank_change
provider_id                                                                                         
D204                    37.50%                  42.86%       1.0             1.0          0.0
D205                    33.33%                  28.57%       2.0             2.0          0.0
D203                    20.00%                  25.00%       4.0             3.0          1.0
D202                    16.67%                  16.67%       5.0             4.0          1.0
D201                    25.00%                  14.29%       3.0             5.0         -2.0
Use code with caution.
• Key Anomaly Found: Provider D201 appeared to have a mediocre 25.0% no-show rate in the raw data (Rank 3). Once impossible timestamps and data-entry errors were scrubbed, their true validated no-show rate dropped to 14.29% (Rank 5), making them the top-performing provider in the network.

Technical Stack & Dependencies

To execute this validation matrix, ensure your local virtual kernel contains the following modules:
bash
numpy >= 1.24.0
pandas >= 1.5.0
matplotlib >= 3.6.0
seaborn >= 0.12.0
openpyxl >= 3.0.0  # Required for processing corporate Excel engines
