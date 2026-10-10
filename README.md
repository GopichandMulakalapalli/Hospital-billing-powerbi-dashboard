# Hospital Billing & Revenue Analysis (Power BI)

An end-to-end analytics project: raw hospital admissions data was cleaned in Excel, modeled as a star schema in Power BI, and turned into an interactive dashboard that shows where revenue comes from and how much billed money is still uncollected.

**Business question:** Where does the hospital earn its revenue, and how much is stuck in pending or rejected insurance claims?

**Author:** Gopichand Mulakalapalli

![Overview page](screenshots/01_overview.png)

---

## Tools

- Excel (data cleaning and validation)
- Power BI Desktop (data model, DAX, dashboard)
- DAX
- Kaggle (data source)

## Data

- **Source:** Kaggle "Healthcare Dataset" (synthetic data), 55,500 rows and 15 columns
- **Columns:** Name, Age, Gender, Blood Type, Medical Condition, Date of Admission, Doctor, Hospital, Insurance Provider, Billing Amount, Room Number, Admission Type, Discharge Date, Medication, Test Results
- **Period covered:** 8 May 2019 to 7 May 2024 (2019 and 2024 are partial years)

## Data cleaning (Excel)

| Issue found | Fix | Result |
|---|---|---|
| Mixed-case names (e.g. "LesLie TErRy") | `PROPER(TRIM())` | Consistent names |
| 534 duplicate rows | Remove Duplicates on all columns | 534 removed |
| 106 negative billing amounts | Filtered and deleted | 106 removed |
| Dates stored as text (dd-mm-yy) | Converted with a `DATE()` formula to avoid day/month swaps | Real date columns |
| Billing decimals | `ROUND(...,2)` | 2 decimal places |

**Final dataset: 54,860 records**, with no blanks, no duplicates, and no negative amounts.

Validation checks passed: discharge date is never earlier than admission date, and length of stay equals the date difference on every row.

### Columns added

| Column | Description |
|---|---|
| PatientID | Sequential ID (the source has none) |
| LengthOfStay | Discharge date minus admission date (1 to 30 days) |
| AgeGroup | 0-17, 18-35, 36-50, 51-65, 66+ |
| BillBand | Low (under 15,000), Medium (15,000 to 35,000), High (over 35,000) |
| AdmitYear, AdmitMonth | Taken from the admission date |
| **ClaimStatus** (simulated) | Approved about 70%, Pending about 15%, Rejected about 15% |
| **AmountPaid** (simulated) | Approved: full bill, Pending: 50%, Rejected: 0 |
| **OutstandingAmount** (simulated) | Billing amount minus AmountPaid |

> **Note:** The source data has no claims information. ClaimStatus, AmountPaid, and OutstandingAmount were generated for analysis practice and do not reflect real claim outcomes.

## Data model

A star schema with one fact table, six dimension tables, and a measures table.

- **Fact table:** `FactAdmissions`
- **Dimensions:** `DimDate`, `DimCondition`, `DimInsurer`, `DimAdmissionType`, `DimClaimStatus`, `DimAgeGroup`
- **Relationships:** six one-to-many, single-direction (DimDate connects to AdmitDate only)
- **Measures table:** `_Measures`

![Data model](screenshots/model_view.png)

## DAX measures

Total Revenue, Total Admissions, Total Patients, Avg Bill, Avg Length of Stay, Total Paid, Outstanding Amount, Collection Rate %, Rejected Amount, Rejection Rate %, Emergency Share %, Revenue LY, Revenue YoY %, Revenue YTD

```DAX
Total Revenue = SUM(FactAdmissions[Billing])

Outstanding Amount = SUM(FactAdmissions[OutstandingAmount])

Collection Rate % = DIVIDE([Total Paid], [Total Revenue])

Rejection Rate % =
DIVIDE(
    CALCULATE([Total Admissions], FactAdmissions[ClaimStatus] = "Rejected"),
    [Total Admissions]
)

Revenue YoY % =
DIVIDE([Total Revenue] - [Revenue LY], [Revenue LY])
```

## Key figures (all years)

| Metric | Value |
|---|---|
| Total revenue | 1,404,121,600 |
| Total admissions | 54,860 |
| Average bill | 25,594.63 |
| Average length of stay | 15.5 days |
| Amount paid | 1,087,249,093 (77.4% collection rate) |
| Outstanding amount | 316,872,507 (about 22.6% of billed) |
| Rejection rate | 15.13% |
| Emergency admissions | 32.94% |

Amounts are shown without a currency because the source data does not specify one.

## Dashboard pages

1. **Overview (built):** KPI cards, monthly revenue trend, revenue by insurer, condition, and admission type, with slicers for year, admission type, and condition
2. **Claims and Collections (planned):** claim status split, claims by insurer, rejection-rate heat map, outstanding amount by condition and year
3. **Clinical Insights (planned):** admissions by age group and gender, average bill by condition, medication usage, test results
4. **Detail and drill-through (planned):** record-level table with drill-through by medical condition

## Insights

- Revenue is steady from 2020 to 2023, at about 277M to 284M per year.
- Revenue and average bill are spread almost evenly across the six conditions (about 230M to 237M each), the five insurers (277M to 284M), and the three admission types (462M to 473M).
- About 23% of the billed amount is outstanding, made up of pending and rejected claims.
- Insurer rejection rates range only from 14.5% to 15.9%. Because claim status is simulated, these differences are noise and not a real finding.

## Limitations

- The data is synthetic, so patterns are flat and do not represent a real hospital.
- Claim status and payment fields are simulated.
- 2019 and 2024 are partial years, so year-over-year comparisons use 2020 to 2023.
- Hospital and doctor names have about 40,000 distinct values each, so they were not used as slicers.
- The dataset has no invoice or due dates, so aging analysis was not possible.

## Repository structure

```
├── README.md
├── Hospital_Billing_Revenue_Dashboard.pbix
├── data/
│   ├── healthcare_dataset.csv          (original Kaggle file)
│   └── healthcare_clean.xlsx           (cleaned file used in Power BI)
├── theme/
│   └── Hospital_Dashboard_Theme.json
└── screenshots/
    ├── 01_overview.png
    └── model_view.png
```

## How to open

1. Install Power BI Desktop (Windows).
2. Open `Hospital_Billing_Revenue_Dashboard.pbix`.
3. If Power BI asks for the data source, point it to `data/healthcare_clean.xlsx` (Home > Transform data > Data source settings).
4. To apply the color theme: View > Themes > Browse for themes > select `theme/Hospital_Dashboard_Theme.json`.

## Data source

Kaggle: Healthcare Dataset (synthetic). Add the dataset link here.
