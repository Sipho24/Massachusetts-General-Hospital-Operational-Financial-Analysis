# Healthcare Operations & Financial Intelligence Dashboard

> **AnalystLab Africa — Data Analytics Internship**  
> An interactive healthcare operations dashboard built in Power BI and Excel to help decision-makers understand encounter patterns, costs, payer coverage, and procedure utilisation at Massachusetts General Hospital (Boston, MA).

---

## Table of Contents

- [Project Overview](#project-overview)
- [Dataset Description](#dataset-description)
- [Project Structure](#project-structure)
- [Task 1: Data Preparation & Modelling](#task-1-data-preparation--modelling)
- [Task 2: Healthcare Operations Analysis](#task-2-healthcare-operations-analysis)
- [Task 3: Dashboard Build](#task-3-dashboard-build)
- [Task 4: Validation & Verification](#task-4-validation--verification)
- [Task 5: Business Recommendations](#task-5-business-recommendations)
- [KPI Definitions](#kpi-definitions)
- [Key Findings](#key-findings)
- [Data Limitations](#data-limitations)
- [Tools & Technologies](#tools--technologies)
- [How to Reproduce](#how-to-reproduce)

---

## Project Overview

A healthcare organisation wants to improve its understanding of hospital operations, encounter costs, payer coverage, procedure utilisation, and patient return visits. This project investigates these areas using historical healthcare datasets and delivers:

- A cleaned, well-modelled star schema in Power BI
- An interactive single-page dashboard with 7 KPIs
- A documented analysis of 8 operational questions
- Independent verification of key calculations
- Three evidence-based business recommendations

**Hospital:** Massachusetts General Hospital, Boston, MA  
**Data Period:** January 2011 – February 2022 (2022 is partial — January 1 to February 5 only)  
**Total Records:** 27,890 encounters | 47,701 procedures | 974 patients | 10 payers | 1 organisation

---

## Dataset Description

| Table | Rows | Columns | Role | Key Fields |
|---|---|---|---|---|
| **Encounters** | 27,890 | 14 | Central fact table | Id, START, STOP, PATIENT, PAYER, ENCOUNTERCLASS, TOTAL_CLAIM_COST, PAYER_COVERAGE, REASONCODE |
| **Patients** | 974 | 20 | Dimension table | Id, BIRTHDATE, DEATHDATE, GENDER, RACE, ETHNICITY, MARITAL, CITY, COUNTY, ZIP |
| **Procedures** | 47,701 | 9 | Secondary fact table | START, STOP, PATIENT, ENCOUNTER, CODE, DESCRIPTION, BASE_COST, REASONCODE |
| **Organizations** | 1 | 8 | Dimension table | Id, NAME, ADDRESS, CITY, STATE, LAT, LON |
| **Payers** | 10 | 7 | Dimension table | Id, NAME, ADDRESS, CITY, STATE_HEADQUARTERED, PHONE |
| **Data Dictionary** | — | 3 | Reference | Table, Field, Description |

### Relationships

```
Organizations (1) ──< Encounters (M) >── Patients (1)
                          │
                    Payers (1) ──<
                          │
                    Procedures (M) ──> Encounters (1)
                          │
                    Procedures (M) ──> Patients (1) [INACTIVE]
```

- **Encounters → Patients:** Many-to-One, Single direction, Active
- **Encounters → Organizations:** Many-to-One, Single direction, Active
- **Encounters → Payers:** Many-to-One, Single direction, Active
- **Procedures → Encounters:** Many-to-One, Single direction, Active
- **Procedures → Patients:** Many-to-One, Single direction, **Inactive** (resolves ambiguity)
- **Date_Table → Encounters:** One-to-Many, Single direction, Active
- **Date_Table → Procedures:** One-to-Many, Single direction, Active

---

## Project Structure

```
healthcare-analysis/
│
├── README.md                                    # This file
│
├── Task1_Data_Preparation_and_Modelling.docx     # Data quality, relationships, model, transformations
├── Task2_Healthcare_Operations_Analysis.docx     # 8 operational questions with metrics & interpretations
├── Task3_Dashboard_Build_Guide.docx              # Single-page dashboard build guide & KPI documentation
├── Task4_Validation_and_Verification.docx        # Independent verification, filter tests, discrepancies
├── Task5_Business_Recommendations.docx           # Three evidence-based recommendations
│
├── HEALTHCARE_DATA_ANALYSIS.md                    # Full data analysis report (Python/pandas output)
├── TASK1_BUSINESS_PROBLEM.md                     # Business problem statement
│
└── dashboard_mockups/
    └── single_page_dashboard.png                # Visual reference for the dashboard
```

---

## Task 1: Data Preparation & Modelling

**Deliverable:** `Task1_Data_Preparation_and_Modelling.docx`

### Data Quality Issues Identified

| # | Issue | Table | Count | Action |
|---|---|---|---|---|
| 1 | PAYER_COVERAGE = $0 with insurance listed | Encounters | 4,779 | Flagged; coverage analysis focused on reliable payers |
| 2 | TOTAL_CLAIM_COST = $0 | Encounters | 89 | Created Zero_Cost_Flag; excluded from cost averages |
| 3 | Encounters after patient death date | Encounters | 68 | Merged DEATHDATE; created After_Death_Flag |
| 4 | Extreme encounter durations | Encounters | ~5+ | Created Duration_Category to flag anomalies |
| 5 | Near-zero private insurer coverage | Encounters | 4,500+ | Documented as data reliability limitation |

### Transformations Performed (Power Query)

**Encounters:**
- Data type corrections (SNOMED codes → Text, dates → DateTime, costs → Decimal)
- Null replacements (REASONCODE → "N/A", REASONDESCRIPTION → "No specific diagnosis")
- Calculated columns: Zero_Cost_Flag, Duration_Total_Minutes, Duration_Hours, Duration_Days, Out_of_Pocket, Coverage_Rate, Year, Quarter, Month, Year_Complete, After_Death_Flag, Duration_Category
- DAX calculated columns: Days_Since_Last_Encounter, Readmission_30Day

**Patients:**
- ZIP code fix: float → text with leading zeros (Text.PadStart to 5 chars)
- Null replacements (MARITAL → "U", SUFFIX → "None", MAIDEN → "N/A")
- Calculated columns: Deceased_Flag, Age, Age_Group

**Payers:**
- ZIP code fix (same as patients)
- Null replacements for NO_INSURANCE row (→ "N/A")
- Calculated column: Payer_Type (Government / Private / No Insurance)

**Procedures:**
- Data type corrections (SNOMED codes → Text)
- Null replacements (REASONCODE → "N/A", REASONDESCRIPTION → "Routine / No specific diagnosis")
- Calculated columns: Procedure_Duration_Minutes, Cost_Category, Year, Month

**Organizations:**
- Minimal transformation (1 row); ZIP fix; technical columns hidden

**Date Table:**
- Created from scratch in Power Query (Blank Query with M code)
- Spans 1 January 2011 to current date (DateTime.LocalNow())
- Columns: Date, Year, Quarter, Month_Number, Month_Name, Year_Month, Day_of_Week
- Marked as date table in Power BI

---

## Task 2: Healthcare Operations Analysis

**Deliverable:** `Task2_Healthcare_Operations_Analysis.docx`

Eight operational questions investigated (requirement: at least four):

| # | Question | Key Finding |
|---|---|---|
| 1 | How have encounter volumes changed over time? | Grew from 1,336 (2011) to 3,885 (2014), stabilised ~2,400, surged to 3,530 in 2021 |
| 2 | Which encounter classes account for the largest proportion? | Ambulatory 44.9% (12,537); inpatient 4.1% (1,135) |
| 3 | What proportion of encounters lasted > 24 hours? | 75 encounters (0.3%) — 58 inpatient, 17 anomalies |
| 4 | How do total claim costs vary across encounter classes? | Ambulatory $36.3M total; inpatient $7,958 avg; max single $641,883 (Sepsis) |
| 5 | How much payer coverage is recorded? | 30.6% coverage rate; 48.7% have $0 coverage; $70.4M out-of-pocket |
| 6 | Which procedures occur most frequently? | Health assessment (4,596), hospice care (4,098), depression screening (3,614) |
| 7 | Which procedures have the highest average base cost? | Hemodialysis $29,300 avg (n=27); cardioversion $25,903 avg (n=1,383) |
| 8 | How do average total claim costs vary by payer? | Medicaid $6,205 avg; NO_INSURANCE $5,593 avg; Dual Eligible $1,696 avg |

Each question includes: metric, calculation method, business interpretation, and data tables.

---

## Task 3: Dashboard Build

**Deliverable:** `Task3_Dashboard_Build_Guide.docx`  
**Visual Reference:** `dashboard_mockups/single_page_dashboard.png`

### Dashboard Layout (Single Page, 16:9)

| Row | Left | Right |
|---|---|---|
| Title bar | "Massachusetts General Hospital — Healthcare Intelligence Dashboard" | — |
| KPI cards | 7 cards in a horizontal row | — |
| Row 3 | Line chart: Encounter volume by year | Donut: Encounter class distribution |
| Row 4 | Bar: Total cost by encounter class | Bar: Coverage rate by payer |
| Row 5 | Bar: Top 10 most frequent procedures | Column: Cost by category + Donut: Payer mix |
| Row 6 | Column: Out-of-pocket distribution | Stacked bar: Volume by class and year + Procedure coverage card |
| Left column | 5 slicers (Year, Class, Payer, Gender, Age Group) | — |

### Filters Applied

| Filter | Level | Value | Reason |
|---|---|---|---|
| Year_Complete | Page | "Complete Year" | Excludes 220 partial 2022 encounters |
| Zero_Cost_Flag | Visual (cost charts) | "No" | Excludes 89 $0-cost encounters from averages |
| Payer_Name | Visual (coverage chart) | != "NO_INSURANCE" | NO_INSURANCE is always 0%, distorts scale |
| Duration_Category | Visual (duration chart) | != "Long Stay (>30 days) - Anomaly" | Excludes data quality outliers |
| Procedure_Description | Visual (top 10) | Top N = 10 | Limits to 10 bars for readability |

---

## Task 4: Validation & Verification

**Deliverable:** `Task4_Validation_and_Verification.docx`

### Independent Verification Results

| Calculation | DAX Result | Independent Result | Match? |
|---|---|---|---|
| Total Encounters | 27,891 | 27,891 (3 methods) | ✅ Yes |
| Avg Cost per Encounter | $3,655.12 | $3,655.12 (2 methods) | ✅ Yes |
| Payer Coverage Rate | 30.6% | 30.6% (2 methods) | ✅ Yes |
| 30-Day Readmission Rate | 61.9% | 60.7% (corrected) | ⚠️ No — discrepancy found |
| Avg Procedures/Encounter | 3.25 | 3.25 (2 methods) | ✅ Yes |

### Discrepancy Found

The readmission DAX formula (`days_since_last <= 30`) includes 322 same-day encounters (days = 0) as "readmissions." These are concurrent visits, not readmissions. The corrected rate is 60.7%.

**Fix:** Change DAX to `days_since_last > 0 AND days_since_last <= 30`.

### Filter Behaviour Confirmed

All filters tested and verified:
- Year_Complete correctly excludes 220 partial 2022 encounters ✅
- Zero_Cost_Flag correctly excludes 89 zero-cost encounters ✅
- Slicers (Encounter Class, Payer) produce correct sub-totals ✅
- Combined filters stack correctly (27,584 with both) ✅

---

## Task 5: Business Recommendations

**Deliverable:** `Task5_Business_Recommendations.docx`

| # | Recommendation | Key Finding | Potential Impact | Limitation |
|---|---|---|---|---|
| 1 | Investigate $70.4M uninsured/under-covered burden | 48.7% of encounters have $0 coverage; $63.1M uncovered | Recover $10–25M in coverage | Requires billing/claims data not in dataset |
| 2 | Care coordination for high-frequency return patients | 60.7% 30-day return rate; median 14 encounters per patient | Reduce unplanned readmissions by 10–15% | No planned/unplanned flag; no outcome data |
| 3 | Review procedure recording process | 47.4% of encounters have no linked procedures | Potential $10–15M in unrecorded costs | Requires EHR audit to determine cause |

---

## KPI Definitions

All KPIs are documented so another analyst can reproduce every calculation.

| # | KPI | Definition | DAX Formula | Filters |
|---|---|---|---|---|
| 1 | Total Encounters | Count of all encounter records | `COUNTROWS(Encounters)` | Year_Complete = Complete Year |
| 2 | Avg Cost per Encounter | Mean total claim cost (excl. zero-cost) | `DIVIDE(SUM(Encounters[Total_Claim_Cost]), COUNTROWS(Encounters), 0)` | Zero_Cost_Flag = No; Year_Complete = Complete Year |
| 3 | Payer Coverage Rate | Total coverage / total claim cost | `DIVIDE(SUM(Encounters[Payer_Coverage]), SUM(Encounters[Total_Claim_Cost]), 0)` | Year_Complete = Complete Year |
| 4 | 30-Day Readmission Rate | % encounters within 30 days of prior visit for same patient | `DIVIDE(CALCULATE(COUNTROWS(Encounters), Encounters[Readmission_30Day]="Yes"), COUNTROWS(Encounters), 0)` | Year_Complete = Complete Year |
| 5 | Total Out-of-Pocket | Sum of (claim cost − coverage) | `SUM(Encounters[Out_of_Pocket])` | Year_Complete = Complete Year |
| 6 | Uninsured Rate | % encounters where payer = NO_INSURANCE | `DIVIDE(CALCULATE(COUNTROWS(Encounters), Payers[Payer_Name]="NO_INSURANCE"), COUNTROWS(Encounters), 0)` | Year_Complete = Complete Year |
| 7 | Avg Procedures per Encounter | Mean procedures per encounter (with procedures) | `DIVIDE(COUNTROWS(Procedures), DISTINCTCOUNT(Procedures[Encounter_ID]), 0)` | Year_Complete = Complete Year |

**Note:** All DAX formulas use `DIVIDE()` instead of the `/` operator to prevent divide-by-zero errors. The third argument (`0`) returns 0 when the denominator is blank or zero.

---

## Key Findings

### Operations
- **27,890 total encounters** across 11+ years (average 2,516/year for complete years)
- **Ambulatory visits dominate** at 44.9% of all encounters; inpatient is only 4.1% but the most expensive per visit
- **99.7% of encounters are same-day** — the hospital is overwhelmingly an outpatient operation

### Costs
- **$101.5M in total encounter costs**; mean $3,651 but median only $279 (heavily right-skewed)
- **Inpatient encounters average $7,958** per visit — 3× the overall average
- **The single most expensive encounter** cost $641,883 (Sepsis, emergency, uninsured)

### Payer Coverage
- **30.6% overall coverage rate** — for every $1 billed, payers cover $0.31
- **48.7% of encounters have $0 coverage** (13,586 encounters)
- **$70.4M out-of-pocket burden** — the dominant financial characteristic of the dataset
- **5 private insurers show near-0% coverage** (Aetna, Anthem, Cigna, Humana, UHC) — data quality concern

### Procedures
- **47,701 total procedures** costing $105.5M
- **Top procedures are assessments and screenings**, not surgical interventions
- **47.4% of encounters have no linked procedures** — ambiguous (consultation-only vs under-recording)
- **77.5% of procedures have no reason code** — limits cost-by-diagnosis analysis

### Readmissions
- **60.7% 30-day return rate** (corrected; DAX reports 61.9% due to same-day inclusion)
- **79.2% of patients** had at least one 30-day return visit
- **Median 14 encounters per patient**; max 1,381 for a single patient
- **Elderly chronic-care population** (median age 79.7, CHF, hyperlipidemia, Alzheimer's)

---

## Data Limitations

| Limitation | Impact |
|---|---|
| **Single hospital** | All data from Massachusetts General Hospital; no multi-site comparison |
| **Synthetic patient data** | Patient names include numbers (e.g., "Nikita578"); likely Synthea-generated |
| **Partial 2022 data** | Only 220 encounters (Jan 1 – Feb 5); excluded from annual comparisons |
| **Private insurer coverage unreliable** | 5 payers show near-0% coverage; may be incomplete data, not genuine non-coverage |
| **No clinical outcome data** | No recovery, complication, or mortality data; readmission is a proxy only |
| **No medication/immunisation data** | Total claim cost may not capture full cost of care |
| **No planned/unplanned flag** | Readmission rate includes planned follow-ups; cannot distinguish clinically |
| **Geographic homogeneity** | All patients in Massachusetts; no regional comparison |
| **No provider/department data** | No physician, department, or care team information |
| **Data dictionary discrepancies** | "Middle" and "FIPS County Code" fields defined but not present in data |

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Data modelling, DAX measures, dashboard visualisation |
| **Power Query** | Data cleaning, transformation, calculated columns |
| **Excel** | Data preparation support, pivot tables |
| **Python (pandas)** | Independent data analysis and verification on raw CSV files |
| **DAX** | Calculated columns and measures for KPIs and time intelligence |

---

## How to Reproduce

### Prerequisites
- Power BI Desktop (latest version)
- The 5 healthcare CSV files (Encounters, Patients, Procedures, Organizations, Payers)
- The Data Dictionary CSV

### Steps

1. **Load Data:** Import all 5 CSV files into Power BI via Get Data → Text/CSV → Transform Data
2. **Clean in Power Query:** Follow the transformation steps documented in `Task1_Data_Preparation_and_Modelling.docx` (Section 4)
3. **Create Date Table:** Follow the M code in Task 1, Section 4.6
4. **Build Model:** Create relationships in Model View per Task 1, Section 3.2
5. **Create Measures:** Add all DAX measures from `Task3_Dashboard_Build_Guide.docx` (Section 4)
6. **Build Dashboard:** Follow the single-page layout in Task 3, Section 5
7. **Add Filters:** Apply page-level and visual-level filters per Task 3, Section 6
8. **Verify:** Cross-check KPI values against `Task4_Validation_and_Verification.docx`

### Reproducing the Python Analysis

```bash
# Install dependencies
python3 -m venv venv
source venv/bin/activate
pip install pandas

# Run analysis (adjust file paths as needed)
python3 analysis_script.py
```

The Python analysis scripts perform:
- Data quality assessment (missing values, duplicates, referential integrity)
- All 8 operational question calculations
- Independent verification of 5 key calculations
- Filter behaviour testing
- Discrepancy investigation

---

## Author

**Sipss**  
AnalystLab Africa — Data Analytics Internship  
Healthcare Intelligence Project Team

---

## Acknowledgements

- AnalystLab Africa for the internship programme and project brief
- Massachusetts General Hospital (synthetic data representation)
- Synthea (likely data generation tool)

---

*All figures in this project were computed directly from the provided datasets using Excel and PowerBIand verified independently. No values were estimated or imputed. Data limitations are documented transparently throughout all deliverables.*

