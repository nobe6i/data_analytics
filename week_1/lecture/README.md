# BAN 612 Lecture 1 Demo: From Raw Data to Trustworthy Evidence

This repository contains the interactive demo notebook used for the introductory lecture of **BAN 612**. 

## Objective
The notebook serves as a compact Python/Pandas refresher designed to introduce the data analytics mindset. Rather than starting with a perfectly prepared dataset, students work through a synthetic, intentionally "messy" orders dataset to master the iterative processes of:
1. **Inspection** – Systematically examining raw tables before making alterations.
2. **Validation** – Setting and tracking flags for data integrity issues.
3. **Cleaning** – Stripping string anomalies, correcting formatting errors, and parsing dates.
4. **Integration** – Safely merging tables while enforcing specific relational schemas (e.g., `many_to_one`).
5. **Summarization & Interpretation** – Formulating safe, data-backed claims versus making unsafe assumptions.

---

## Technical Stack & Libraries
The notebook runs entirely on standard open-source Python libraries:
* **Pandas:** For structural matrix manipulation and grouping.
* **NumPy:** For managing numerical null pointers (`np.nan`).
* **Matplotlib:** For rendering descriptive business charts.

---

## Notebook Structure & Workflow

### 1. Ingestion & Raw Data Creation
Generates a small, synthetic transactional dataset containing hidden errors common to real-world corporate systems (e.g., raw currency strings, mixed date structures, casing issues, duplicate keys, and missing or negative amounts).

### 2. Systematic Data Inspection
Demystifies the data checking process by evaluating matrix bounds using `.shape`, `.info()`, `.describe()`, `.isna()`, and `.duplicated()`.

### 3. Structural Cleansing
Removes syntactic bugs: strips white spaces, harmonizes categorical strings (`.str.title()`, `.str.upper()`), handles custom values as NaN markers, strips currency characters, and normalizes date configurations using `pd.to_datetime`.

### 4. Integrity Flagging & Validation
Creates boolean validation frameworks to explicitly flag non-conforming rows (`negative_amount`, `missing_date`, `duplicate_order_id`) rather than quietly throwing data out.

### 5. Data Aggregation & Analytical Merges
Enforces strict relationship checks during data merges (`validate="many_to_one"`) with a secondary customer segmentation table, leveraging merge indicators to spot broken foreign keys.

### 6. Interpretation: The Claim Ladder
Concludes with a lesson on data interpretation, comparing a descriptive **safe claim** (supported directly by the evidence) with an **unsafe claim** (assuming causal linkings without structural experimental designs).

---

## Mini-Exercise Included
The notebook ends with an actionable student challenge designed to modify the code blocks and address targeted business questions regarding duplicates, missing data audits, merge exceptions, and safe data claim formulations.
