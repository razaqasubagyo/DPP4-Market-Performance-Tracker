# Data Documentation

This folder documents the data source, analytical scope, market definition, preparation workflow, and key quality-assurance considerations used in the **DPP-4 Market Performance Tracker**.

The original NHS prescribing files are not stored directly in this repository because of their size.

---

## Data Source

The project uses publicly available data from the:

**NHS Business Services Authority (NHSBSA) — English Prescribing Dataset**

The dataset contains prescribing activity recorded in England and includes fields such as:

- Prescribing month
- Chemical substance code
- Chemical substance description
- BNF presentation code
- Presentation description
- Prescription items
- Total quantity
- Net Ingredient Cost (NIC)
- Actual Cost
- SNOMED code

The data is used to analyse DPP-4 category performance at molecule and presentation level.

---

## Analytical Period

The final analytical dataset covers:

> **July 2024 – July 2026**

This provides 25 monthly periods for trend analysis.

The primary headline comparison used in the dashboard is:

> **January–July 2026 vs January–July 2025**

This ensures that year-on-year metrics compare equivalent seven-month periods.

Historical trend visuals use the full available period:

> **July 2024 – July 2026**

---

## Market Definition

The primary analytical market consists of five major **single-agent DPP-4 molecules**:

- Sitagliptin
- Linagliptin
- Alogliptin
- Saxagliptin
- Vildagliptin

Combination products were excluded from the primary market definition.

This means that measures such as:

- DPP-4 Market Items
- Market Growth
- Volume Share
- Share Change
- Growth Gap
- Contribution to Net Category Change
- NIC Share

are calculated using only the five defined single-agent molecules.

Therefore:

> **Market share in this project refers to prescription-item share within the defined single-agent DPP-4 category.**

---

## Combination Products

The NHS prescribing data also contains DPP-4-containing combination products.

Examples may include combinations such as:

- DPP-4 inhibitor + metformin
- DPP-4 inhibitor + SGLT2 inhibitor

These were not included in the primary analysis.

The exclusion was deliberate in order to maintain a clearer and more consistent competitive denominator.

The project therefore focuses on:

> **single-agent molecule-level DPP-4 prescribing performance**

rather than the entire universe of DPP-4-containing medicines.

---

## Analytical Grain

The original source data is available at GP-practice level.

Because this project does not include geographic analysis, the data was aggregated to:

> **one row per presentation per month**

The final grouping structure includes:

- YearMonth
- ChemicalCode
- Molecule
- PresentationCode
- PresentationName
- SNOMEDCode

The following measures were aggregated using sums:

- Items
- TotalQuantity
- NIC
- ActualCost

This reduces model size while preserving the level of detail needed for:

- monthly market trends;
- molecule-level comparisons;
- market-share analysis;
- growth analysis;
- recorded cost analysis; and
- presentation-level drill-down.

---

## Final Fact Table

The final `Fact_Prescribing` table contains:

- `YearMonth`
- `ChemicalCode`
- `Molecule`
- `PresentationCode`
- `PresentationName`
- `Items`
- `TotalQuantity`
- `NIC`
- `ActualCost`
- `SNOMEDCode`

---

## Dimension Tables

### `Dim_Calendar`

Contains:

- YearMonth
- Year
- MonthNumber
- MonthName
- Quarter
- MonthYear
- YearMonthSort

The table supports:

- monthly trend analysis;
- chronological sorting;
- year-on-year comparison; and
- time-intelligence calculations.

---

### `Dim_Chemical`

Contains:

- ChemicalCode
- Molecule
- TherapyClass
- DPP4Flag

The table contains one row per DPP-4 molecule.

Expected molecule count:

> **5**

---

### `Dim_Presentation`

Contains:

- PresentationCode
- PresentationName
- ChemicalCode
- Molecule
- SNOMEDCode
- Formulation
- Strength
- PresentationGroup

The table contains one row per unique presentation code.

Presentation attributes were simplified into commercially readable groupings where possible.

Examples include:

- Tablet 25 mg
- Tablet 50 mg
- Tablet 100 mg
- Oral Solution

---

# Data Preparation Workflow

The data-preparation process was completed in Power Query.

The main workflow included:

1. Importing monthly NHS prescribing files.
2. Separating files into compatible source-schema groups.
3. Harmonising column names across source structures.
4. Standardising field names.
5. Converting month fields into proper date values.
6. Correcting numeric data types for cost fields.
7. Filtering to the five defined DPP-4 molecules.
8. Excluding combination products from the primary market.
9. Appending monthly datasets.
10. Aggregating practice-level records to monthly presentation level.
11. Creating dimension tables.
12. Validating relationships and analytical grain.
13. Performing quality-assurance checks before building DAX measures.

---

## Source Schema Harmonisation

The monthly source files did not all use an identical schema.

Different source groups were therefore harmonised before append.

Examples of fields requiring standardisation included:

- prescribing month;
- chemical-substance code;
- chemical-substance description;
- BNF presentation code;
- presentation description;
- NIC;
- Actual Cost; and
- SNOMED code.

The final analytical naming convention uses:

```text
YearMonth
ChemicalCode
Molecule
PresentationCode
PresentationName
Items
TotalQuantity
NIC
ActualCost
SNOMEDCode
```

---

## YearMonth Transformation

Monthly source values stored in `YYYYMM` format were converted into proper date fields.

Example Power Query logic:

```powerquery
#date(
    Number.FromText(Text.Start(Text.From([YearMonth]), 4)),
    Number.FromText(Text.End(Text.From([YearMonth]), 2)),
    1
)
```

Example:

```text
202407
```

becomes:

```text
01/07/2024
```

This allows the use of Power BI time intelligence and chronological trend analysis.

---

## MonthYear and Sort Fields

A readable month-year label was created using:

```powerquery
Date.ToText([YearMonth], "MMM yyyy")
```

Example:

```text
Jul 2024
```

A numeric sort field was also created:

```powerquery
Date.Year([YearMonth]) * 100 + Date.Month([YearMonth])
```

Example:

```text
202407
```

`MonthYear` is sorted by `YearMonthSort` in the Power BI model.

---

## Cost Field Data-Type Correction

An important data-quality requirement was ensuring that:

- `NIC`
- `Actual Cost`

were imported and retained as decimal numeric values.

These fields were parsed using decimal numeric types and appropriate locale handling.

Example Power Query typing pattern:

```powerquery
{"NIC", type number},
{"ACTUAL_COST", type number}
```

with locale handling applied during conversion.

This avoids losing decimal precision.

Fixed divisors were not used because source cost values can contain different decimal lengths.

The correction therefore occurs at the data-type level rather than by manually dividing values.

---

## Final Data Types

The final fact table uses:

| Column | Data Type |
|---|---|
| YearMonth | Date |
| ChemicalCode | Text |
| Molecule | Text |
| PresentationCode | Text |
| PresentationName | Text |
| Items | Whole Number |
| TotalQuantity | Decimal Number |
| NIC | Decimal Number |
| ActualCost | Decimal Number |
| SNOMEDCode | Text |

---

# Aggregation Logic

After append and cleaning, the fact table was grouped by:

- `YearMonth`
- `ChemicalCode`
- `Molecule`
- `PresentationCode`
- `PresentationName`
- `SNOMEDCode`

The following fields were summed:

- `Items`
- `TotalQuantity`
- `NIC`
- `ActualCost`

The target analytical grain is:

> **one row per presentation per month**

---

# DPP-4 Molecule Mapping

The final market contains:

| Molecule | Therapy Class |
|---|---|
| Sitagliptin | DPP-4 |
| Linagliptin | DPP-4 |
| Alogliptin | DPP-4 |
| Saxagliptin | DPP-4 |
| Vildagliptin | DPP-4 |

The analysis remains at molecule level rather than manufacturer or brand level.

This is important because public prescribing data may contain branded and generic presentations of the same chemical substance.

---

# Quality Assurance

The final dataset was checked before analytical measures were developed.

Key QA checks included:

- complete monthly coverage from Jul 2024 to Jul 2026;
- five expected DPP-4 molecules;
- valid `YearMonth` values;
- no unexpected blank molecule mappings;
- unique presentation codes in `Dim_Presentation`;
- correct one-to-many relationships;
- realistic NIC and Actual Cost values;
- market-share totals summing to 100%;
- NIC-share totals summing to 100%;
- value-volume gaps summing to approximately 0 percentage points at total-market level;
- prior-year measures behaving correctly; and
- headline Jan–Jul 2026 comparisons matching Jan–Jul 2025.

---

## Example 2026 YTD QA Results

For Jan–Jul 2026:

- **DPP-4 prescription items:** approximately 3.96M
- **DPP-4 YoY growth:** -0.61%
- **Recorded NIC:** approximately £62.84M
- **NIC YoY growth:** -18.57%
- **NIC per Item:** approximately £15.86

The molecule-level analysis also identified Sitagliptin as the largest molecule by prescription-item volume.

---

# Commercial Measure Definitions

## Prescription Items

Prescription Items are used as the primary volume measure.

They represent prescribing activity.

They do not represent unique patients.

---

## Net Ingredient Cost

NIC is used as the primary recorded cost measure.

It should not be interpreted as:

- manufacturer revenue;
- net sales;
- commercial profit;
- realised selling price; or
- profitability.

---

## NIC per Item

NIC per Item is calculated as:

> **Recorded NIC / Prescription Items**

It is used as a diagnostic measure to understand the relationship between recorded cost and prescribing activity.

It should not be interpreted as a manufacturer selling price.

---

## Volume Share

Volume Share is calculated as:

> **Molecule Prescription Items / Total DPP-4 Prescription Items**

The measure therefore represents:

> **item-based prescribing share**

rather than patient share.

---

## NIC Share

NIC Share is calculated as:

> **Molecule NIC / Total DPP-4 NIC**

This represents the proportion of recorded ingredient cost associated with each molecule.

---

## Value-Volume Gap

The project defines:

> **Value-Volume Gap = NIC Share − Volume Share**

A positive gap indicates that a molecule represents a larger proportion of recorded NIC than of prescription-item volume.

A negative gap indicates the opposite.

This should not be interpreted as a measure of profitability.

---

## Absolute Growth

Absolute Growth measures:

> **Current prescription items − prior-year prescription items**

This answers:

> How much volume was added or lost?

---

## YoY Growth

YoY Growth measures:

> **(Current − Prior Year) / Prior Year**

This answers:

> How fast did the market or molecule change?

Absolute growth and percentage growth should therefore not be interpreted as the same concept.

---

## Growth Gap

Growth Gap compares molecule growth with category growth:

> **Molecule YoY Growth − DPP-4 Market YoY Growth**

A positive gap indicates that a molecule outperformed the category.

A negative gap indicates that it underperformed the category.

---

## Share Change

Share Change is calculated as:

> **Current Volume Share − Prior-Year Volume Share**

The result is expressed in:

> **percentage points**

rather than percentage growth.

---

## Contribution to Net Category Change

Contribution to Net Category Change is calculated as:

> **Molecule Absolute Growth / DPP-4 Absolute Growth**

The result may:

- exceed 100%;
- be negative; or
- appear unusually large

when positive and negative molecule-level changes offset each other.

This is particularly important when the total category records only a small net change.

---

# Important Interpretation Notes

## Items Are Not Patients

Prescription items reflect prescribing activity.

They should not be translated into unique patient numbers.

For example, the analysis should say:

> Sitagliptin generated additional prescription items.

It should not say:

> Sitagliptin treated additional patients.

---

## Market Share Is Item-Based

Different DPP-4 molecules may have different prescribing patterns.

Therefore, prescription-item share should not be interpreted as:

- patient share; or
- therapeutic-equivalent-unit share.

---

## NIC Is Not Revenue

NIC is a recorded prescribing-cost measure.

A molecule with a larger NIC share should not automatically be described as:

- more profitable;
- higher revenue;
- more commercially valuable.

---

## Value-Volume Differences Require Careful Interpretation

Differences between volume share and NIC share may be associated with:

- NIC per item;
- presentation mix;
- prescribing mix; and
- differences in recorded ingredient cost across molecules.

The project does not attempt to establish causal attribution between these factors.

---

# Data Limitations

## England Scope

The dataset reflects prescribing activity captured in England.

The results should not be interpreted as total UK pharmaceutical consumption.

---

## Combination Products

Combination products are excluded from the primary DPP-4 market.

The findings therefore relate specifically to the defined single-agent category.

---

## No Patient-Level Data

The dataset does not contain longitudinal patient-level information.

No conclusions are made about:

- patient switching;
- adherence;
- persistence; or
- individual treatment pathways.

---

## No Manufacturer Financial Data

The dataset does not provide:

- manufacturer revenue;
- net realised price;
- rebates;
- discounts;
- margin; or
- profitability.

---

## No Causal Inference

The dashboard identifies patterns and associations in prescribing and recorded cost data.

It does not establish that one observed factor caused another.

---

# Why Raw Data Is Not Included

The original NHS prescribing files are large and are not stored directly in this repository.

Instead, this repository documents:

- data scope;
- market definition;
- transformation logic;
- analytical grain;
- quality-assurance approach; and
- key interpretation constraints.

Users wishing to reproduce the analysis should download the relevant monthly English Prescribing Dataset files from the official NHSBSA public data source.

---

# Reproducibility Workflow

To reproduce the dataset:

1. Download monthly NHSBSA English Prescribing Dataset files covering Jul 2024–Jul 2026.
2. Filter to Sitagliptin, Linagliptin, Alogliptin, Saxagliptin and Vildagliptin.
3. Exclude combination products from the primary market.
4. Harmonise source schemas.
5. Standardise field names.
6. Convert prescribing month into a proper date.
7. Parse NIC and Actual Cost as decimal numeric fields.
8. Append monthly files.
9. Aggregate records to monthly presentation level.
10. Create `Dim_Calendar`, `Dim_Chemical` and `Dim_Presentation`.
11. Validate molecule counts, monthly coverage and cost values.
12. Build the DAX measures documented in the main project README.
13. Recheck headline market totals and share measures before dashboard publication.

---

# Disclaimer

This project was developed independently for educational and portfolio purposes using publicly available NHS prescribing data.

It is not affiliated with, endorsed by, or produced on behalf of the NHS, any pharmaceutical manufacturer, or any other commercial organisation.

The analysis is intended to demonstrate data preparation, data modelling, Power BI, DAX, commercial analytics, and analytical interpretation.

It should not be interpreted as clinical, financial, investment, or commercial advice.
