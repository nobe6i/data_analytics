# BAN 612: Clinic Scheduling Data Analytics & Validation (Week 2)

This repository contains the Jupyter Notebook code and analytical frameworks used to parse, validate, and audit heterogeneous clinic scheduling files for a multi-clinic medical group.

## Project Objective
Before operational teams can safely analyze clinical metrics like patient no-shows, wait times, or telehealth utilization, the underlying electronic health record (EHR) data must be programmatically audited. Real-world clinic logs are often combined from disparate legacy systems and contain mixed data formats, mismatched keys, and impossible chronological sequences. 

This project implements a reproducible Python data quality pipeline that flags data integrity anomalies and builds an exportable **Data Quality Issue Log**.

---

## Dataset Architectures & Formats
The validation pipeline ingests and evaluates data across three separate storage formats:
1.  **Clinic Appointments (`clinic_appointments.csv`):** Transactional scheduling table tracking `appointment_id`, patient/provider references, booking times, and visit status strings.
2.  **Patient Master (`clinic_patients.xlsx`):** Demographic master table tracking unique patient keys, dates of birth (`date_of_birth`), assigned sex, insurance type, and postal ZIP codes.
3.  **Provider Directory (`clinic_providers.json`):** Directory mapping distinct physician keys (`provider_id`) alongside specialty fields and clinic locations.

---

## Data Validation & Engineering Pipeline

The programmatic audit pipeline is built sequentially across the following operational phases:

### 1. Ingestion & Schema Inspection
*   Evaluates matrix boundaries using `.shape` and extracts inferred data schemas via `.info()`.
*   Uncovers columns containing null factors or misclassified generic structural objects (e.g., date columns arriving as plain text strings).

### 2. Text Normalization & Standard Mapping
*   Removes leading and trailing white spaces using string modifiers (`.str.strip()`).
*   Resolves variations in case text (`.str.lower()`) and merges fragmented categorical labels into uniform fields.
*   **Status Recoding:** Consolidates ambiguous historical entries (e.g., mapping `no-show`, `no show`, and `NS` down to a single standard `No Show` target).
*   **Telehealth Mapping:** Standardizes boolean values (`y`, `yes`, `n`, `no`) into absolute `Yes` or `No` categories.

### 3. Chronological Consistency & Regex Parsing
*   Converts text-mapped strings into rigorous timestamp objects with `pd.to_datetime(errors='coerce', format='mixed')`.
*   **Format Exceptions:** Uses boolean masking arrays to catch completely corrupted date variables that fail parsing (e.g., impossible calendar inputs like `2026-13-40 25:00`).
*   **ZIP Code Pattern Auditing:** Uses regex pattern matchers (`^\\d{5}$`) to isolate malformed postal indices (e.g., corrupted 4-digit codes like `9458`).
*   **Biometric Tracking:** Subtracts birth dates from current clock cycles (`pd.Timestamp.today()`) to generate continuous age fields for anomaly tracking.

### 4. Relational Integrity Jumps
*   Executes relational checks using `.isin()` to find broken orphan keys across tables.
*   Isolates appointments tied to unknown patient keys (e.g., `P9999`) or unregistered doctor IDs (e.g., `D299`) that do not exist within the primary master tables.
*   Identifies business key anomalies, including duplicate instances of unique keys (e.g., multiple conflicting entries for `A3015`).

### 5. Multi-Column Business Rule Enforcement
The pipeline evaluates logical dependencies across multiple fields to flag operational errors:
*   **Booking Chronology:** Identifies records where the booking log time (`booked_at`) succeeds the actual physical appointment slot.
*   **Visit Duration Errors:** Cross-references checkout logs with check-in timestamps to catch impossible negative clinic durations.
*   **Conditional Completeness:** Flags completed visits (`Status == 'Completed'`) that lack mandatory operational timestamps.
*   **Status/Timestamp Conflicts:** Catches cancelled visits that incorrectly contain active check-in stamps.
*   **Clinic Grace Periods:** Utilizes conditional `np.where` arrays to evaluate check-in latency, dynamically updating statuses to `No Show` if the patient arrives past the typical 15-minute clinic grace window.

---

## Technical Stack & Dependencies

To execute this pipeline, ensure your local virtual environment contains the following dependencies:

```bash
numpy >= 1.24.0
pandas >= 1.5.0
matplotlib >= 3.6.0
openpyxl >= 3.0.0  # Required for reading Excel engines
```

---

## Output Data Quality Log
The pipeline aggregates all flagged violations into a standardized, exportable data quality report (`clinic_data_quality_issue_log.csv`) detailing the violation type, the count of affected records, and the recommended system fix:

| rule | count | action |
| :--- | :--- | :--- |
| Invalid appointment datetime | 1 | Investigate / correct source |
| Duplicate appointment ID | 2 | Investigate duplicates |
| Unknown patient ID | 1 | Resolve against patient master |
| Unknown provider ID | 1 | Resolve against provider master |
| Booking after appointment | 1 | Flag as chronology error |
| Checkout before check-in | 1 | Flag as chronology error |
| Completed visit missing timestamp | 1 | Investigate incomplete workflow |
| Cancelled visit with check-in | 1 | Investigate status/timestamp conflict |

---

## In-Class Practice Exercises Included
The bottom section of the notebook contains a series of independent coding tasks to reinforce data quality engineering concepts:
1.  **Practice 1:** Standardizing raw visit type columns.
2.  **Practice 2:** Testing patient master uniqueness constraints.
3.  **Practice 3:** Computing scheduling lead times and isolating negative deltas.
4.  **Practice 4:** Computing active visit durations in minutes and filtering extreme statistical outliers.
5.  **Practice 5:** Implementing explicit relational merges (`validate='many_to_one'`) with provider directories.
