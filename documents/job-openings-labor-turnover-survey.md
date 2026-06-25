The Job Openings and Labor Turnover Survey (JOLTS) dataset produced by the Bureau of Labor Statistics (BLS) measures labor demand and turnover at the national, regional, and state levels.

Like other BLS programs, JOLTS organizes data in a relational format. It separates time-series observations (the measurements) from the structural categories describing those observations (the dimensions). Every specific cross-section of data maps back to a unique composite smart key called the **`series_id`**.

---

## 1. Fact Table: Measurements & Observations

This table stores the historical metrics captured by the survey over time. Each row represents a specific data point for an industry, region, size class, and time interval.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(21) | Foreign Key | The 21-character unique code identifying a specific labor market intersection. | `JTU000000000000000HIL` |
| **`year`** | INT | Composite PK | The 4-digit calendar year of the observation. | `2026` |
| **`period`** | CHAR(3) | Composite PK | The exact month or summary window of the data point. | `M01`–`M12` (Months), `M13` (Annual Average) |
| **`value`** | NUMERIC(12,2) | **Measurement** | The actual quantity. This represents **either an absolute count in thousands** (for levels) or a **percentage** (for rates). | `5462.00` (5,462,000 hires) or `3.4` (3.4% hires rate) |
| **`footnote_codes`** | VARCHAR(10) | Lookup Link | Flags data revisions, anomalies, or collection caveats. Left blank for standard data. | `P` (Preliminary), `R` (Revised) |

---

## 2. Main Dimension Table: Series Metadata

This table defines what a given `series_id` actually represents by breaking out its distinct contextual properties.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(21) | Primary Key | Master unique identifier for database joins. | `JTU110099000000000HIL` |
| **`seasonal`** | CHAR(1) | Dimension | Indicates if seasonal, holiday, or calendar noise has been statistically smoothed out. | `S` (Seasonally Adjusted), `U` (Not Seasonally Adjusted) |
| **`industry_code`** | VARCHAR(6) | Foreign Key | Maps to the NAICS or custom sector groupings table. | `000000` (Total Nonfarm), `110099` (Logging) |
| **`state_code`** | CHAR(2) | Foreign Key | Identifies the specific U.S. state or region. | `00` (Total US), `06` (California) |
| **`area_code`** | VARCHAR(5) | Foreign Key | Identifies sub-state geographical or metropolitan zones. | `00000` (All Areas) |
| **`size_class_code`** | CHAR(2) | Foreign Key | Represents the establishment size bracket by employee count. | `00` (All Sizes), `01` (1-9 employees) |
| **`data_element_code`** | CHAR(2) | Foreign Key | The specific turnover event being tracked. | `JO` (Job Openings), `HI` (Hires) |
| **`rate_level_code`** | CHAR(1) | Foreign Key | Denotes whether the figure is expressed as a count or a rate. | `L` (Level), `R` (Rate) |
| **`series_title`** | VARCHAR(255) | Attribute | Clear description of the tracking criteria. | `"Hires level, Total nonfarm, Seasonally Adjusted"` |
| **`begin_year`** | INT | Attribute | First available calendar year of tracking. | `2000` |
| **`end_year`** | INT | Attribute | The most recent or current active year. | `2026` |

---

## 3. Sub-Dimension Lookups

To save space and avoid redundancy, the metric classifications and parameters link out to descriptive secondary tables.

### Data Element Code Lookup (`data_element`)

| Code | Definition | Description |
| --- | --- | --- |
| **`JO`** | Job Openings | Active, unfilled positions on the last business day of the month. |
| **`HI`** | Hires | All additions to the payroll occurring throughout the entire month. |
| **`TS`** | Total Separations | Total number of employees leaving payroll (Quits + Layoffs + Others). |
| **`QU`** | Quits | Voluntary separations initiated by the employee. |
| **`LD`** | Layoffs and Discharges | Involuntary separations initiated by the employer. |
| **`OS`** | Other Separations | Executive retirements, deaths, transfers, or permanent disability. |

### Rate / Level Code Lookup (`rate_level`)

| Code | Label | Measurement Context |
| --- | --- | --- |
| **`L`** | Level | Expressed as an absolute count **in thousands** (e.g., `5,000` = 5,000,000 people). |
| **`R`** | Rate | Expressed as a **percentage**. For openings, it is relative to total employment + openings. For turnover, it is relative to total employment. |

### Size Class Code Lookup (`size_class`)

| Code | Employee Count Range |
| --- | --- |
| **`00`** | All Size Classes Combined |
| **`01`** | 1 to 9 employees |
| **`02`** | 10 to 49 employees |
| **`03`** | 50 to 249 employees |
| **`04`** | 250 to 999 employees |
| **`05`** | 1,000 to 4,999 employees |
| **`06`** | 5,000 or more employees |

---

## 4. The Anatomy of a JOLTS `series_id`

You can parse the 21-character structure of a JOLTS `series_id` programmatically to read the dimensions directly without executing a relational database join.

> **Example Series ID:** `JTU110099000000000HIL`

* **Positions 1–2 (Survey Prefix):**
* `JT`: Job Openings and Labor Turnover Survey


* **Position 3 (Seasonal Adjustment Code):**
* `U`: Not Seasonally Adjusted
* `S`: Seasonally Adjusted


* **Positions 4–9 (Industry Code):**
* `110099`: Industry sub-sector or aggregate sector (e.g., `000000` is Total Nonfarm)


* **Positions 10–11 (State Code):**
* `00`: Represents the collective United States national data


* **Positions 12–16 (Area Code):**
* `00000`: Placeholder padding for future regional/MSA expansions


* **Positions 17–18 (Size Class Code):**
* `00`: All establishment size tiers aggregate


* **Positions 19–20 (Data Element Code):**
* `HI`: Tracks hiring events


* **Position 21 (Rate/Level Code):**
* `L`: Values represent standard level metrics (counts in thousands)
