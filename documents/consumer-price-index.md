The Bureau of Labor Statistics (BLS) Consumer Price Index (CPI) dataset follows a relational database structure (similar to a star schema). It separates economic measurements (the fact table) from descriptive context (dimension tables).

The primary link between measurements and dimensions is the **`series_id`**, which functions as a unique time-series identifier and a composite smart key.

---

## 1. Fact Table: Measurements & Observations

This table stores the physical data observations collected over time. Each row represents a specific metric recorded at a given point in time for a distinct index.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(17) | Foreign Key | The unique alpha-numeric code tracking a specific items-and-geography time series. | `CUUR0000SA0` |
| **`year`** | INT | Composite PK | The 4-digit calendar year of the price observation. | `2026` |
| **`period`** | CHAR(3) | Composite PK | The code indicating the exact time interval within the year. | `M01`–`M12` (Months), `M13` (Annual Average), `S01`–`S02` (Semi-Annual) |
| **`value`** | NUMERIC(7,3) | **Measurement** | The core metric value. Typically represents the index level normalized to a base period (usually 1982-1984 = 100), or an average price amount. | `335.123` |
| **`footnote_codes`** | VARCHAR(10) | Lookup Link | Identifies specific data annotations, anomalies, or collection constraints. Blank indicates a standard, unflagged entry. | `P` (Preliminary), `R` (Revised) |

---

## 2. Main Dimension Table: Series Metadata

This table defines the structural traits of a given time series. It maps the `series_id` to its localized, specific dimensions.

| Field Name | Data Type | Keys / Relations | Description | Example / Allowed Values |
| --- | --- | --- | --- | --- |
| **`series_id`** | VARCHAR(17) | Primary Key | Master identifier for the overarching dataset join. | `CUUR0000SA0` |
| **`area_code`** | CHAR(4) | Foreign Key | Links to the geographical area dimension lookup table. | `0000` (U.S. City Average), `S49E` (San Diego) |
| **`item_code`** | VARCHAR(10) | Foreign Key | Links to the distinct item category/basket breakdown. | `SA0` (All Items), `SAF1` (Food) |
| **`seasonal`** | CHAR(1) | Dimension | Designates whether the observations filter out predictable calendar-based patterns. | `S` (Seasonally Adjusted)<br>

<br>
`U` (Not Seasonally Adjusted) |
| **`periodicity_code`** | CHAR(1) | Dimension | Tracks how frequently the index updates. | `R` (Regularly/Monthly)<br>

<br>`S` (Semi-Annually) |
| **`base_period`** | VARCHAR(20) | Attribute | The reference period assigned a value index benchmark of 100. | `1982-84=100` |
| **`series_title`** | VARCHAR(255) | Attribute | Full, human-readable description of the dimension intersection. | `"All items in U.S. city average, all urban consumers, not seasonally adjusted"` |
| **`begin_year`** | INT | Attribute | The historical start date of the time series. | `1913` |
| **`end_year`** | INT | Attribute | The final or active year of recording. | `2026` |

---

## 3. Sub-Dimension Lookups (Area and Item Tables)

To avoid data redundancy, localized classifications are broken out into secondary lookup definitions.

### Area Dimension (`area`)

| Field Name | Data Type | Description |
| --- | --- | --- |
| **`area_code`** | CHAR(4) [PK] | Unique token tracking a geography. |
| **`area_name`** | VARCHAR(100) | Plain-text name of the city, region, or aggregate metropolitan boundary. |

### Item Dimension (`item`)

| Field Name | Data Type | Description |
| --- | --- | --- |
| **`item_code`** | VARCHAR(10) [PK] | Unique identifier mapping to the basket classification index. |
| **`item_name`** | VARCHAR(150) | The specific type of good, service, or overarching major category (e.g., "Medical care", "Apparel"). |

---

## 4. The Anatomy of a `series_id` (Composite Smart Key)

If you are querying data via flat text files or the BLS Public API, you can programmatically extract major dimensions straight out of the `series_id` character placements without executing a database join:

> **Example Series ID:** `CUUR0000SA0`

* **Characters 1–2 (Index Type / Prefix):** Targets the surveyed population demographic group.
* `CU`: All Urban Consumers (CPI-U) — *Covers ~93% of the U.S. population.*
* `CW`: Urban Wage Earners and Clerical Workers (CPI-W) — *Used primarily for Social Security COLA adjustments.*
* `SU`: Chained CPI for All Urban Consumers (C-CPI-U).


* **Character 3 (Seasonal Adjustment Status):**
* `U`: Unadjusted (Not Seasonally Adjusted).
* `S`: Seasonally Adjusted.


* **Character 4 (Periodicity):**
* `R`: Regular publication schedule (Monthly).
* `S`: Semi-Annual compilation schedule.


* **Characters 5–8 (Area Code):** Mapped string specifying region (e.g., `0000` is the absolute national average).
* **Characters 9+ (Item Code):** Variable-length code mapping straight to the item taxonomy (e.g., `SA0` represents "All Items").
