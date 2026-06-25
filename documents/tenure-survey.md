The Employee Tenure dataset is collected biennially (every two years, typically in January) as a specialized supplement to the **Current Population Survey (CPS)**. Unlike monthly tracking indices, this dataset evaluates the long-term stability and shifts in the labor market by measuring how long individuals have been with their current employer.

The survey targets **employed wage and salary workers** specifically (the self-employed and unpaid family workers are excluded). The data schema isolates demographic and professional traits (dimensions) from median and distributional timelines (measurements).

---

## 1. Core Measurement Table (Fact Table Layout)

This structural table records the final statistical calculations compiled from the CPS household microdata records during each biennial release window.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`survey_year`** | INT | Composite PK | The 4-digit calendar year the supplement was fielded. | `2024`, `2026` |
| **`survey_month`** | CHAR(2) | Attribute | The exact collection month. This is structurally pinned to January. | `01` |
| **`series_id` / `group_id**` | VARCHAR(20) | Foreign Key | Relational link mapping to the demographic and structural dimensions. | Unique intersection key string |
| **`measure_code`** | CHAR(2) | Foreign Key | Links directly to the tenure calculation type. | `MT` (Median Tenure), `D1` (Distribution Tier 1) |
| **`value`** | NUMERIC(5,2) | **Measurement** | The actual measured result. Expressed as **either a number of years** or as a **percentage (%)**. | `3.9` (3.9 median years) or `22.0` (22% of workforce) |

---

## 2. Demographic Dimension Attributes

Because tenure patterns are heavily influenced by population traits, the dataset relies heavily on cross-sectional demographic variables.

| Field Name | Data Type | Attribute Categories | Description / Allowed Values |
| --- | --- | --- | --- |
| **`age_group`** | VARCHAR(15) | Age Cohorts | Breaks down the population into intervals. Tenure trends scale upward significantly with age buckets `16–19`, `20–24`, `25–34`, `35–44`, `45–54`, `55–64`, `65+` |
| **`sex`** | CHAR(1) | Gender | Tracks variances between male and female retention habits. • `M` (Men)  • `F` (Women) • `A` (All combined) |
| **`race_ethnicity`** | VARCHAR(25) | Racial & Ethnic Identity | Maps the employee baseline to major demographic definitions. • `White`, `Black or African American`, `Asian`, `Hispanic or Latino` |
| **`education_level`** | VARCHAR(40) | Educational Attainment | Filter applied strictly to workers **ages 25 and over**. `Less than a high school diploma`, `High school graduates, no college`, `Some college or associate degree`, `Bachelor's degree and higher` |

---

## 3. Employment & Structural Dimension Attributes

These attributes categorize the operational sector and type of work environment where the tenure was accumulated.

### Industry Classifications (`industry`)

Industries track the business activities of the employer. They map closely to the North American Industry Classification System (NAICS).

* **Public Sector:** Split into `Federal`, `State`, and `Local` government divisions (traditionally displaying the highest median tenure levels).
* **Private Sector:** Evaluates distinct economic segments:
* `Mining, quarrying, and oil and gas extraction`
* `Manufacturing`
* `Financial activities`
* `Leisure and hospitality` *(historically maps to the lowest median tenure levels)*



### Occupational Classifications (`occupation`)

Occupations follow the Standard Occupational Classification (SOC) matrix, highlighting the explicit tasks performed by the worker regardless of their industry setting.

* `Management, professional, and related occupations`
* `Service occupations`
* `Sales and office occupations`
* `Natural resources, construction, and maintenance occupations`
* `Production, transportation, and material moving occupations`

---

## 4. Measure Lookups & Data Element Definitions

The **`measure_code`** field dictates how database users must interpret the physical numeric `value` column.

| Measure Code | Metric Name | Unit | Definition |
| --- | --- | --- | --- |
| **`MT`** | Median Tenure | Years | The exact point at which half of the surveyed subset workers have more tenure and half have less. |
| **`D1`** | Short-Term Share | Percent (%) | The percentage of workers with **12 months or less** of tenure with their current employer (e.g., recent hires, recent job switchers). |
| **`D2`** | 1 to 2 Years Share | Percent (%) | The percentage of workers with more than 1 year up to 2 years of tenure. |
| **`D3`** | 3 to 4 Years Share | Percent (%) | The percentage of workers with 3 to 4 years of tenure. |
| **`D4`** | 5 to 9 Years Share | Percent (%) | The percentage of workers with 5 to 9 years of tenure. |
| **`D5`** | Long-Term Share | Percent (%) | The percentage of workers boasting **10 years or more** of tenure with their current employer. |
