
California Market Expansion Index & Sensitivity Matrix (Homework 4)

This repository houses a reproducible data engineering and strategic analytics pipeline engineered to evaluate and rank California municipal markets for expansion opportunities. The workflow merges official federal census data profiles with external macroeconomic real estate indicators to construct a dynamic Market Attractiveness Index.

Team Authors

• Hong Ngoc Dinh
• Nora Cook
• Hmwe Pyant Thin Chit

Project Objective

When organizations consider regional expansions, relying on isolated metrics (like raw population or median income alone) can create highly distorted conclusions. A market with soaring household incomes might suffer from crippling real estate values or localized labor force constraints.
This project solves this multi-variable problem by programmatically loading data across multiple dimensions—Economic Sizing, Human Capital Availability, Macro Affordability, and Labor Market Engagement—to rank 455 California cities across 5 corporate expansion strategy scenarios.

Data Engineering & Pipeline Architecture

The automated ingestion pipeline integrates information across public API networks and corporate storage files:

1. Programmatic Ingestion Layer

• Federal Census Ingestion (census_df): Executes stateful GET handshakes against the 2024 American Community Survey (ACS) 5-Year Data Profile API, mapping abstract variable keys to structural data series.
• Zillow Macro Ingestion (zillow_df): Materializes cross-sectional housing vectors from the Zillow Home Value Index (ZHVI) to measure typical single-family home value cycles.

2. Schema Remediation & Name Standardization

• Key Coercion: Standardizes string labels by stripping whitespace and replacing spaces with pythonic underscores. Text fields are converted to explicit integers and floating-point parameters.
• Geographic Fuzzy Trapping: Legacy spreadsheet conventions frequently mismatch relational join anchors. The script intercepts and corrects string variations (e.g., standardizing El Paso de Robles (Paso Robles) to Paso Robles, or San Buenaventura (Ventura) down to Ventura) to achieve a 98.7% successful join match rate across systems.

3. Missingness Auditing & Value Masking

• The Anomaly Purge: The pipeline systematically audits data boxes for negative placeholder flags (such as the default Census unavailable marker -666666666) or impossible mathematical parameters (like labor numbers outstripping working-age brackets), stripping 243 corrupted rows to safeguard analytical boundaries.

Feature Construction Layer

Beyond raw attributes, the script engineers specialized behavioral metrics to track true commercial vitality:
1. Labor Force Participation Rate: Captures active workforce capacity relative to working-age limits:
\(\text{Labor\ Force\ Participation\ \%}=\left(\frac{\text{Labor\ Force}}{\text{Population\ 16+}}\right)\times 100\)
2. Labor Force-to-Population Density: Tracks the proportion of the entire municipal population that is economically active, serving as a proxy for localized disposable income density:
\(\text{Labor\ Force-to-Population\ \%}=\left(\frac{\text{Labor\ Force}}{\text{Total\ Population}}\right)\times 100\)
3. Non-Poverty Buffer Vector: Inverts default poverty frequencies to construct a clear metric tracking household spending power:
\(\text{Non-Poverty\ Rate\ \%}=100-\text{Poverty\ Rate}\)
4. Housing Affordability Score: Measures real estate values against income metrics, then normalizes the result using MinMax transformations so that high-affordability (low-ratio) zones safely score closest to 100.

Multi-Scenario Weighting Matrix

To account for shifting corporate investment criteria, the model dynamically computes composite attractiveness profiles across 5 distinct strategy frameworks:

Evaluation Dimension	Base Case	Equal Weights	Pro-Affordability	Pro-Talent	Pro-Economy
Economic Stability (Non-Poverty)	35%	25%	20%	20%	50%
Human Capital (Education %)	25%	25%	15%	50%	20%
Labor Engagement (Workforce Density)	20%	25%	15%	15%	15%
Macro Affordability (ZHVI Normalized)	20%	25%	50%	15%	15%


Strategic Leaderboard Findings

Running the weighting scenarios against the cleaned database reveals how target location rankings shift based on corporate priorities:
text
=== TOP 10 CITIES ACROSS ALL SCENARIOS ===
        Base Case                    Equal Weights                Pro-Affordability            Pro-Talent                   Pro-Economy  
Rank 1  Emeryville city (81.2)       Emeryville city (81.0)       Emeryville city (86.7)       Orinda city (81.5)           Orinda city (85.1)  
Rank 2  Orinda city (81.0)           Orinda city (77.6)           Amador City city (82.0)      Piedmont city (81.0)         Lafayette city (83.4)  
Rank 3  Sausalito city (79.5)        Sausalito city (77.4)        Dublin city (81.9)           Lafayette city (78.0)        Dublin city (83.2)  
Rank 4  Dublin city (79.5)           Dublin city (77.1)           Clayton city (79.9)          Sausalito city (77.8)        Piedmont city (83.2)  
Rank 5  Lafayette city (79.1)        Lafayette city (75.8)        Folsom city (79.5)           Emeryville city (77.6)       Sausalito city (82.4)  
Rank 6  Piedmont city (79.1)         San Ramon city (75.5)        Orinda city (79.4)           Albany city (77.2)           San Ramon city (82.4)  
Rank 7  San Ramon city (78.4)        Albany city (75.5)           Ridgecrest city (79.1)       Cupertino city (76.9)        Emeryville city (82.3)  
Rank 8  Albany city (78.3)           Piedmont city (75.4)         La Canada Flintridge (78.6)  Del Mar city (76.5)          Foster City city (81.9)  
Rank 9  Mountain View city (77.9)    Mountain View city (75.0)    Rocklin city (78.2)          Los Altos city (76.0)        Albany city (81.8)  
Rank 10 Foster City city (77.6)      Foster City city (74.5)      Hercules city (78.2)         Mountain View city (76.0)    Mountain View city (81.7)  
Use code with caution.

Core Analytical Insights

• The Dominance of Emeryville: Emeryville ranks 1st under the Base Case, Equal Weights, and Pro-Affordability frameworks due to its exceptional labor force metrics and highly balanced housing-to-income performance.
• The Talent and Wealth Pivot: When the model shifts focus to high education (Pro-Talent) or raw economic stability (Pro-Economy), Orinda takes the lead. This shift highlights how luxury enclaves excel in talent and wealth metrics but drop in ranking when affordability constraints are strictly applied.
• Strategic Expansion Expansion Picks: While cities like Fremont and Sunnyvale remain strong, adding affordability metrics highlights Dublin and Eastvale as highly attractive alternatives. They offer high median incomes ($214k and $160k respectively) paired with exceptional housing-to-income affordability ratios.

Core Technical Stack & Verification

To execute this validation matrix locally, verify that your virtual python kernel contains the following package installations:
bash
pip install numpy pandas matplotlib requests tabulate
Use code with caution.

Script Directory Organization

text
your-repository-root/
│
├── City_zhvi_uc_sfrcondo_tier_0.33_0.67_sm_sa_month.csv # Core Zillow single-family metrics file
├── market_expansion_pipeline.ipynb                      # Primary script execution file
└── README.md                                            # Repository homep
