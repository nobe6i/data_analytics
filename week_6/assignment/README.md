
California Market Expansion Index & Sensitivity Matrix (Assignment 4)

This repository houses a reproducible data engineering and strategic analytics pipeline designed to evaluate and rank California municipal markets for commercial expansion opportunities. The workflow fuses federal demographic profiles with external macroeconomic real estate indices to construct a dynamic, weighted Market Attractiveness Index.

Team Authors

• Hong Ngoc Dinh
• Nora Cook
• Hmwe Pyant Thin Chit

Project Overview

When organizations evaluate regional expansions, relying on isolated metrics (like raw population or median household income alone) can lead to highly distorted conclusions. A market with soaring household incomes might suffer from extreme real estate costs, while a populous hub might exhibit low labor market engagement.
This project solves this multi-variable problem by programmatically integrating data across four pillars—Economic Sizing, Human Capital Availability, Macro Affordability, and Labor Market Engagement—to evaluate 455 California cities across 5 distinct corporate strategy scenarios.

Data Engineering & Pipeline Architecture

The automated ingestion pipeline integrates information across public API networks and external data files:

1. Programmatic Ingestion Layer

• Federal Census Ingestion (census_df): Fetches regional profiles from the 2024 American Community Survey (ACS) 5-Year Data Profile API, mapping abstract variable keys to structural data series.
• Zillow Macro Ingestion (zillow_df): Ingests city-level home valuation cycles from the Zillow Home Value Index (ZHVI) to establish a macro real estate baseline.

2. Schema Remediation & Name Standardization

• Key Coercion: Standardizes column labels, handles missing numeric entries using pd.to_numeric(errors="coerce"), and strips out trailing formatting spaces.
• Geographic Fuzzy Trapping: Municipal naming conventions frequently diverge across federal and private systems. The pipeline intercepts and corrects string variations (e.g., mapping El Paso de Robles (Paso Robles) to Paso Robles, or San Buenaventura (Ventura) down to Ventura) to achieve a 98.7% successful merge rate.

3. Missingness Auditing & Value Masking

• The Anomaly Purge: The pipeline systematically audits records for negative placeholder flags (such as the default Census unavailable marker -666666666) or impossible data boundaries (like labor metrics outstripping the working-age population), purging 243 corrupted rows to ensure clean analysis boundaries.

Feature Construction Layer

Beyond raw attributes, the script engineers specialized metrics to track local economic vitality:
1. Labor Force Participation Rate: Captures active workforce capacity relative to working-age limits:
\(\text{Labor\ Force\ Participation\ \%}=\left(\frac{\text{Labor\ Force}}{\text{Population\ 16+}}\right)\times 100\)
2. Labor Force-to-Population Density: Tracks the proportion of the entire municipal population that is economically active, serving as a proxy for localized spending power:
\(\text{Labor\ Force-to-Population\ \%}=\left(\frac{\text{Labor\ Force}}{\text{Total\ Population}}\right)\times 100\)
3. Non-Poverty Buffer Vector: Inverts default poverty frequencies into a positive attribute tracking household independence:
\(\text{Non-Poverty\ Rate\ \%}=100-\text{Poverty\ Rate}\)
4. Housing Affordability Score: Compares typical home value against median household income, then applies MinMax normalization so that high-affordability (low-ratio) zones score closest to 100.

Multi-Scenario Weighting Matrix

To account for shifting corporate investment criteria, the model dynamically computes composite attractiveness profiles across 5 distinct strategy frameworks:

Evaluation Dimension	Base Case	Equal Weights	Pro-Affordability	Pro-Talent	Pro-Economy
Economic Stability (Non-Poverty %)	35%	25%	20%	20%	50%
Human Capital (Education %)	25%	25%	15%	50%	20%
Labor Engagement (Workforce Density)	20%	25%	15%	15%	15%
Macro Affordability (ZHVI Affordability Score)	20%	25%	50%	15%	15%


Strategic Leaderboard Findings

Running the sensitivity matrix against the cleaned database reveals how target location rankings shift based on corporate priorities:
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

• The Dominance of Emeryville: Emeryville ranks 1st under the Base Case, Equal Weights, and Pro-Affordability frameworks. This is due to its exceptional labor force-to-population rate (54.5%) and highly optimized home-value-to-income performance (3.75).
• The Talent and Wealth Pivot: When the model shifts focus to high education (Pro-Talent) or raw economic stability (Pro-Economy), Orinda takes the lead. This shift highlights how affluent enclaves excel in talent and wealth metrics but drop in ranking when affordability constraints are heavily weighted.
• Strategic Alternatives: While major hubs like Fremont, Sunnyvale, and Irvine remain historically strong, applying macro affordability filters highlights Dublin and Eastvale as exceptionally robust options. Both offer strong median household incomes ($214k and $160k respectively) paired with highly favorable home value-to-income profiles.

Technical Stack & Verification

To execute this pipeline locally, verify that your virtual python kernel contains the following package installations:
bash
pip install numpy pandas matplotlib requests tabulate
Use code with caution.

Script Directory Organization

text
your-repository-root/
│
├── City_zhvi_uc_sfrcondo_tier_0.33_0.67_sm_sa_month.csv # Inbound Zillow home-value metrics file
├── market_expansion_pipeline.ipynb                      # Primary script execution notebook
└── README.md                                            # Repository homepage
