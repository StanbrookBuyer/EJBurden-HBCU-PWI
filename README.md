================================================================================
DATA AND CODE README
================================================================================

Title:    Environmental injustice in the shadow of the ivory tower: Cumulative
          EJ burden in communities surrounding Historically Black Colleges and
          Universities compared to primarily-white institutions

Authors:  Stanbrook-Buyer, R., Labady, R.

EJScreen version used: EJScreen v2.3 (accessed April 2026)
R version used:        R 4.5.1

================================================================================
FILE INVENTORY
================================================================================

CODE
────
hbcu_pwi_reproducible_pipeline.qmd
    Quarto document containing the complete, self-contained analysis pipeline.
    Structured in three parts:
      Part 1 — Data Acquisition (EJScreen API fetch; skippable if using CSVs)
      Part 2 — Data Processing (population-weighted aggregation, wide format)
      Part 3 — Statistical Analysis (all tables, figures, and tests in paper)
    Set USE_ARCHIVED_DATA <- TRUE (the default) to reproduce published results
    exactly using the archived CSV files below. Set to FALSE to re-fetch from
    the live EJScreen API (see Reproducibility Notes below).

DATA FILES
──────────
HBCU-DATA2.0.csv
    Raw HBCU institutional dataset. Columns:
      school        — Institution name (character)
      Location      — City, State (character)
      Latitude      — Campus centroid latitude, decimal degrees WGS84 (numeric)
      Longitude     — Campus centroid longitude, decimal degrees WGS84 (numeric)
      year_founded  — Year of institutional founding (integer)
    Source: Curated HBCU institutional dataset, cross-validated against federal
    Title III designation records. 101 rows (raw, before coordinate cleaning).

hbcu_list.csv
    Cleaned HBCU campus list, output of prepare_hbcu_list.R. 87 institutions
    after removal of records with missing or out-of-bounds coordinates. Same
    columns as HBCU-DATA2.0.csv. This is the definitive HBCU sample used in
    all analyses.

pwi_list.csv
    PWI comparison group. 86 institutions assembled manually from publicly
    available federal data. Same columns as hbcu_list.csv.
    Selection criteria: (i) four-year, degree-granting, accredited institutions
    classified as primarily-white under federal Title III definitions; (ii)
    located in a state also represented in the HBCU sample; (iii) comparable
    undergraduate enrollment; (iv) no specialist institutional mission.

hbcu_ej_wide.csv
    Analysis-ready HBCU environmental justice data. One row per institution
    (n = 87). Contains campus-level population-weighted mean EJ Summary Index
    national (USA) percentile values for 13 environmental indicators, derived
    from EPA EJScreen v2.2 via ArcGIS REST API (1-mile buffer; April 2025).
    See DATA DICTIONARY below for column definitions.

pwi_ej_wide.csv
    Analysis-ready PWI environmental justice data. One row per institution
    (n = 86). Identical structure to hbcu_ej_wide.csv.

matched_hbcu_pwi.csv
    Post-matching dataset: 70 HBCU–PWI pairs (140 rows) resulting from 1:1
    nearest-neighbour propensity-score matching on Census region and founding
    year (MatchIt package, R). This is the dataset used in all primary
    inferential analyses (Sections 8–13 of the pipeline).

================================================================================
DATA DICTIONARY — hbcu_ej_wide.csv and pwi_ej_wide.csv
================================================================================

school
    Institution name. Character. Primary key; matches hbcu_list.csv /
    pwi_list.csv.

Location
    City and state abbreviation, e.g. "Nashville, TN". Character.

Latitude, Longitude
    Campus centroid coordinates, decimal degrees, WGS84. Numeric.

year_founded
    Year of institutional founding. Integer.

The following 13 columns are campus-level population-weighted mean EJ Summary
Index national (USA) percentile scores. Each combines a raw environmental
indicator with demographic vulnerability factors (low income, race/ethnicity,
linguistic isolation, age) to produce a cumulative EJ burden score expressed
as a national percentile (0–100). Higher values indicate greater cumulative
EJ burden relative to the US population.

Source: https://pedp-ejscreen.azurewebsites.net (accessed April 2026).
Buffer: 2-mile circular buffer around campus centroid.
Aggregation: population-weighted mean across all intersecting census block
groups (weighted by ACSTOTPOP, ACS total resident population).

  Particulate Matter Summary Index_pct_usa
      PM2.5 EJ Index, national percentile. EJScreen field: P_EJDPM25SUP.

  Ozone Summary Index_pct_usa
      Ozone EJ Index, national percentile. EJScreen field: P_EJOZONESUP.

  Nitrogen Dioxide (NO2) Summary Index_pct_usa
      NO2 EJ Index, national percentile. EJScreen field: P_EJNO2SUP.

  Diesel Particulate Matter Summary Index_pct_usa
      Diesel PM EJ Index, national percentile. EJScreen field: P_EJDSLPMSUP.

  Toxic Releases to Air Summary Index_pct_usa
      Air toxics cancer risk EJ Index, national percentile.
      EJScreen field: P_EJCANCR.

  Traffic Proximity and Volume Summary Index_pct_usa
      Traffic EJ Index, national percentile. EJScreen field: P_EJPTRAF.

  Lead Paint Summary Index_pct_usa
      Lead paint EJ Index, national percentile. EJScreen field: P_EJLDPNT.

  Superfund Proximity Summary Index_pct_usa
      Superfund proximity EJ Index, national percentile.
      EJScreen field: P_EJPNPL.

  RMP Proximity Summary Index_pct_usa
      RMP facility proximity EJ Index, national percentile.
      EJScreen field: P_EJPRMP.

  Hazardous Waste Proximity Summary Index_pct_usa
      Hazardous waste (TSDF) proximity EJ Index, national percentile.
      EJScreen field: P_EJPTSDF.

  Underground Storage Tanks Summary Index_pct_usa
      UST EJ Index, national percentile. EJScreen field: P_EJUST.

  Wastewater Discharge Summary Index_pct_usa
      Wastewater discharge EJ Index, national percentile.
      EJScreen field: P_EJPWDIS.

  Drinking Water Non-Compliance Summary Index_pct_usa
      Drinking water non-compliance EJ Index, national percentile.
      EJScreen field: P_EJDWATERNC.

================================================================================
REPRODUCIBILITY NOTES
================================================================================

To reproduce published results exactly
───────────────────────────────────────
1. Place all CSV files from this deposit in a single working directory.
2. Open hbcu_pwi_reproducible_pipeline.qmd in RStudio (version >= 2023.09)
   or run `quarto render hbcu_pwi_reproducible_pipeline.qmd` from the terminal.
3. Ensure USE_ARCHIVED_DATA <- TRUE (default). The pipeline will load the
   archived CSVs and skip the API fetch steps.
4. Install required R packages (listed in Section 1 of the QMD, or run the
   install.packages() call provided there).

To re-fetch data from the EPA EJScreen API
────────────────────────────────────────────
Set USE_ARCHIVED_DATA <- FALSE. The pipeline will query the EJScreen ArcGIS
FeatureServer API using the campus coordinates in hbcu_list.csv and
pwi_list.csv. Note:

  - EJScreen data are updated annually by the EPA. Re-fetching will use the
    current EJScreen release, which may differ from v2.2 (April 2025).
    Numerical results may differ from published values.
  - The fetch step queries ~173 campuses with a rate-limiting pause between
    calls. Total runtime is approximately 15–20 minutes with a stable internet
    connection.
  - The EJSCREEN_VERSION parameter in the QMD specifies which service version
    to query. To replicate the original data exactly, the EPA would need to
    host an archived v2.2 endpoint; if unavailable, the current service will
    be queried automatically.

Software versions
──────────────────
R version:     4.5.1 (2025-06-13)
Quarto:        >= 1.4
Key packages:  tidyverse 2.0.0, MatchIt 4.5.5, httr 1.4.7, sf 1.0-16,
               gt 0.10.1, ggridges 0.5.6, car 3.1-2

================================================================================
LICENSE
================================================================================

Code (hbcu_pwi_reproducible_pipeline.qmd): CC BY 4.0
Data (all CSV files): CC0 1.0 (public domain dedication)

When reusing these data or code, please cite the associated publication.


================================================================================
