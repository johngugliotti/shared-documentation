The Bureau of Labor Statistics (BLS) tracks unemployment and labor market information through two primary survey datasets: the **Current Population Survey (CPS)** (national-level and demographic data) and the **Local Area Unemployment Statistics (LAUS)** (state, regional, county, and metro data).

Both utilize a relational data warehouse model mapping physical economic measurements to descriptive dimension attributes via a composite smart key called the **`series_id`**.

---

## 1. Fact Table: Measurements & Observations

This table holds the actual time-series logs. Each row represents a specific metric captured at a given point in time.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(20) | Foreign Key | The exact alphanumeric key mapping to specific survey segments and geographies. | `LNS14000000`, `LAUCN040130000000003` |
| **`year`** | INT | Composite PK | The 4-digit calendar year of the recorded labor data. | `2026` |
| **`period`** | CHAR(3) | Composite PK | The standardized interval within the year. | `M01`–`M12` (Months), `M13` (Annual Average) |
| **`value`** | NUMERIC(12,2) | **Measurement** | The actual count or rate. Represents **either a percentage** (for rates) or **counts in thousands/absolute numbers** (for volumes). | `4.1` (4.1% Unemployment Rate) or `6200` (6,200,000 individuals) |
| **`footnote_codes`** | VARCHAR(10) | Lookup Link | Flags data caveats, updates, or collection anomalies. Blank means standard production data. | `P` (Preliminary), `R` (Revised) |

---

## 2. Main Dimension Table: Series Metadata

This table serves as the primary master catalog defining what a particular `series_id` represents structurally.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(20) | Primary Key | Master unique identifier for the time series. | `LNS14000000` |
| **`survey_id`** | CHAR(2) | Dimension | Identifies the originating operational program. | `LN` (Current Population Survey)<br>

<br>`LA` (Local Area Unemployment Stats) |
| **`seasonal`** | CHAR(1) | Dimension | Shows if raw calendar noise or seasonal hiring patterns have been smoothed out. | `S` (Seasonally Adjusted)<br>

<br>`U` (Not Seasonally Adjusted) |
| **`area_code`** | VARCHAR(15) | Foreign Key | Maps directly to geographic classification tables (crucial for LAUS). | `` (Blank for National), `CN0401300000000` (Maricopa County, AZ) |
| **`measure_code`** | CHAR(2) | Foreign Key | The type of labor market metric being explicitly quantified. | `03` (Unemployment Rate), `06` (Labor Force) |
| **`series_title`** | VARCHAR(255) | Attribute | Descriptive, human-readable title specifying demographics, metrics, and location boundaries. | `"Unemployment Rate - 16 Years & Over, Seasonally Adjusted"` |
| **`begin_year`** | INT | Attribute | The historical beginning point of the data series. | `1948` |
| **`end_year`** | INT | Attribute | The most recent or current year of active tracking. | `2026` |

---

## 3. Sub-Dimension Lookups (Measure and Area)

### Measure Code Lookup (`measure`)

While the measurement column in the fact table holds the numeric value, this dimension defines what that value represents:

| Measure Code | Measure Name | Unit of Measurement | Description |
| --- | --- | --- | --- |
| **`03`** | Unemployment Rate | Percent (%) | The percentage of the labor force that is unemployed. |
| **`04`** | Unemployment Level | Count (Absolute or Thousands) | The total number of people actively seeking work but unemployed. |
| **`05`** | Employment Level | Count (Absolute or Thousands) | The total number of individuals currently holding jobs. |
| **`06`** | Civilian Labor Force | Count (Absolute or Thousands) | The sum of all employed and unemployed individuals. |

### LAUS Area Lookup (`area`)

For sub-national data, areas are broken into detailed regional segments using an established hierarchy layout.

| Field Name | Data Type | Description | Example |
| --- | --- | --- | --- |
| **`area_code`** | VARCHAR(15) [PK] | Code utilizing Federal Information Processing Standards (FIPS). | `CN0401300000000` |
| **`area_type_code`** | CHAR(1) | Classifies the spatial scale of the target boundary. | `F` (State), `G` (County), `B` (Metropolitan Area) |
| **`area_name`** | VARCHAR(100) | Plain-text name of the geographical boundary. | `Maricopa County, Arizona` |

---

## 4. The Anatomy of an Unemployment `series_id`

When importing BLS flat files or hitting their public API directly, you can parse the string programmatically to isolate dimensions instantly without running multi-table joins.

### A. National / Demographic Data (CPS / `LN` Series)

> **Example Series ID:** `LNS14000000`

* **Characters 1–2 (`LN`):** Core Program Prefix — Labor Force Statistics from the Current Population Survey.
* **Character 3 (`S`):** Seasonal Adjustment Status (`S` = Seasonally Adjusted, `U` = Unadjusted).
* **Characters 4–11 (`14000000`):** Character / Demographic Group Code.
* `14000000` represents the general top-line national **Unemployment Rate**.
* Shifted variations track subsets like `14000003` (Unemployment Rate for Men) or `14000006` (Unemployment Rate for Black or African American workers).



### B. State / Local Data (LAUS / `LA` Series)

> **Example Series ID:** `LAUCN040130000000003`

* **Characters 1–2 (`LA`):** Core Program Prefix — Local Area Unemployment Statistics.
* **Character 3 (`U`):** Seasonal Adjustment Status (`U` = Not Seasonally Adjusted, `S` = Seasonally Adjusted).
* **Characters 4–18 (`CN040130000000`):** 15-character Geographic Area Code.
* `CN` acts as an area type flag (County).
* `04013` matches the FIPS state and county schema (`04` = Arizona, `013` = Maricopa County).
* Trailing zeros pad the remaining positions.


* **Characters 19–20 (`03`):** Measure Code (Maps directly to the **Measure Lookup Table** to denote that this tracks the *Unemployment Rate* rather than raw employment totals).
