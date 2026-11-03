# Uber NYC Pickups: Demand Pattern Analysis

Exploratory data analysis of hourly Uber pickups across New York City boroughs (January–June 2015), combined with weather and holiday data. The goal is to find demand patterns the business can act on.

## Business Context

Ridesharing demand shifts with time, location, weather and local events. Matching driver supply to those shifts reduces rider wait times and lost trips. This analysis measures which factors drive pickup volume most and where operations could respond.

**Key questions**
1. Which boroughs and hours of the day carry the most demand?
2. Does weather (temperature, wind, visibility, precipitation, snow) explain meaningful variation in pickups?
3. Do holidays change demand, and is the effect the same in every borough?

## Dataset

`Uber.csv`: 29,101 rows × 13 columns. Each row is one borough in one hour.

| Column | Description |
|---|---|
| `pickup_dt` | Date and time of the pickup period (hourly, 2015-01-01 01:00 to 2015-06-30 23:00) |
| `borough` | NYC borough (Bronx, Brooklyn, EWR, Manhattan, Queens, Staten Island) |
| `pickups` | Number of pickups in the period |
| `spd` | Wind speed (mph) |
| `vsb` | Visibility (miles) |
| `temp` | Temperature (°F) |
| `dewp` | Dew point (°F) |
| `slp` | Sea-level pressure |
| `pcp01`, `pcp06`, `pcp24` | Precipitation over the past 1, 6 and 24 hours (inches) |
| `sd` | Snow depth (inches) |
| `hday` | Holiday indicator (`Y` / `N`) |

## Methodology

| Stage | What was done |
|---|---|
| Data profiling | Shape, dtypes, null counts, `describe()` for numeric and categorical columns, category frequencies |
| Data cleaning | 3,043 missing `borough` values (10.5% of rows) labeled `"Unknown"` rather than dropped, so the weather and pickup data on those rows is kept |
| Validation | Pandas statistics cross-checked against NumPy (`mean`, `std`, 90th percentile) |
| Feature engineering | Month, day, weekday and hour extracted from `pickup_dt` |
| Selection and filtering | Conditional indexing with `loc` / `iloc` (for example, Manhattan hours above 5,000 pickups: 218 rows) |
| Aggregation | `groupby` on borough, holiday status, and borough × holiday |
| Univariate analysis | Histogram with box plot for `pickups`; share of records by borough |
| Bivariate and multivariate analysis | Hourly pickup profile by borough; temperature vs. pickups scatter; Manhattan hour × weekday heat map |

**Stack:** Python, pandas, NumPy, seaborn, matplotlib

## Key Findings

### 1. Demand is highly concentrated in Manhattan
| Borough | Average pickups per hour |
|---|---|
| Manhattan | 2,387 |
| Brooklyn | 534 |
| Queens | 309 |
| Bronx | 51 |
| Staten Island | 1.6 |
| EWR | 0.02 |

Manhattan accounts for roughly 73% of all pickups, Brooklyn 16% and Queens 9%. EWR and Staten Island are near zero and contribute almost nothing to volume.

### 2. Pickups are heavily right-skewed
Mean pickups per row are 490 against a median of 54, with a maximum of 7,883. About 19% of rows have zero pickups (mostly EWR, Staten Island and low-activity hours). The mean alone misleads here. Any modeling work should consider a log transform or a model fitted per borough.

### 3. Time of day is the strongest demand driver
In Manhattan, demand peaks in the evening at 18:00–22:00, with 19:00 the busiest hour (about 4,300 average pickups). It is lowest at 04:00–05:00 (about 600). The busiest hour-and-weekday cells are Thursday and Friday around 19:00–20:00 (about 5,000 average pickups), with a second peak late on Saturday night. Friday and Saturday are the busiest days overall, Monday the quietest.

### 4. Weather has a weak relationship with pickups
The temperature scatter shows no strong pattern. In a supplementary check run for this write-up, the correlation between pickups and temperature was about 0.06 across all rows, and about 0.16 within Manhattan. Wind speed, visibility, precipitation and snow depth were all near zero. Weather alone does not explain demand in this data. This is a linear, univariate check and does not rule out interaction effects (for example, rain during rush hour).

### 5. Holidays lower demand, unevenly
| | Non-holiday | Holiday | Change |
|---|---|---|---|
| All boroughs | 492 | 437 | about -11% |
| Manhattan | 2,401 | 2,036 | about -15% |
| Brooklyn | 535 | 527 | about -1% |
| Queens | 309 | 321 | about +4% |

The holiday drop comes mainly from Manhattan. Queens is slightly higher on holidays. Holidays are only 1,121 of 29,101 rows (3.9%), so these differences should be confirmed with significance testing before acting on them.

## Recommendations

1. **Position supply in Manhattan for the evening peak.** Concentrate driver availability and incentives at 18:00–22:00, especially Thursday and Friday.
2. **Use the hour × weekday heat map for scheduling.** Target incentives at the windows where demand is high and supply is likely short, and avoid paying for supply in the 03:00–06:00 trough.
3. **Treat holidays as borough-specific.** A blanket holiday policy would over-correct for Brooklyn and Queens. Investigate local events behind the Queens increase.
4. **Do not prioritize weather in a first-pass staffing model.** Time and location explain far more variation. Revisit weather in combination with time of day.
5. **Review the EWR and Staten Island data.** Near-zero volume suggests either minimal service in 2015 or incomplete coverage in the source data.

## Limitations and Next Steps

- **Time window:** The data covers six months (January–June 2015), so seasonality across a full year cannot be assessed. Holiday effects rest on a small sample.
- **Unknown borough:** 3,043 rows have no borough. Each is a single row per timestamp averaging about 2 pickups, so the effect on borough totals is minimal, but the cause of the missing labels is unconfirmed.
- **Correlation, not causation:** The analysis is descriptive. No hypothesis tests or models were run.
- **Next steps:**
  - Test holiday and weekday differences for significance.
  - Fit a demand forecasting model (for example, gradient boosting) with hour, weekday, borough, holiday and weather features.
  - Add event data and a full year of history.

## Repository Structure

```
.
├── Second_case_study_AI_assisted_Coding_and_Data_Analysis.ipynb   # Analysis notebook
├── Uber.csv                                                       # Dataset
└── README.md
```

## How to Run

```bash
git clone https://github.com/Bilal-Butt/Uber-Data-Analysis.git
cd Uber-Data-Analysis
pip install pandas numpy seaborn matplotlib jupyter
jupyter notebook Second_case_study_AI_assisted_Coding_and_Data_Analysis.ipynb
```

`Uber.csv` must stay in the same folder as the notebook, since it is loaded by filename.

## Author

Bilal Butt
