# Findings and Recommendations

Companion to the [notebook](../notebooks/uber_nyc_pickups_analysis.ipynb). Dataset: 29,101 hourly borough-level records of Uber pickups in New York City, January to June 2015, with weather and holiday information.

---

## 1. Where demand is

| Borough | Avg pickups per hour | Share of all pickups |
|---|---:|---:|
| Manhattan | 2,387 | 72.7% |
| Brooklyn | 534 | 16.3% |
| Queens | 309 | 9.4% |
| Bronx | 51 | 1.5% |
| Staten Island | 1.6 | under 0.1% |
| EWR | 0.02 | under 0.1% |
| Unknown (3,043 unlabelled rows) | 2.1 | under 0.1% |

- Pickups are strongly right-skewed (skewness 3.0): mean 490, median 54, maximum 7,883.
- 19% of rows have zero pickups: EWR has zero in 4,243 of its 4,343 hours and Staten Island in 1,291; each other named borough has zero in only 8-9 hours.
- Each named borough has exactly 4,343 rows (one per hour), so record counts show coverage, not demand.
- 218 Manhattan hours exceed 5,000 pickups.

## 2. When demand peaks (Manhattan)

- **Hour of day:** a smaller morning peak around 09:00 (about 2,434), a midday dip (1,984 at 11:00), then a rise to the main peak at **19:00 (4,344)**. The trough is 04:00-05:00 (about 600).
- **Busiest hour-and-weekday cells:** Thursday 19:00 (5,086), Friday 19:00 (5,032), Friday 20:00 (4,978), Saturday 23:00 (4,821), Saturday 20:00 (4,789).

| Weekday | Avg Manhattan pickups per hour |
|---|---:|
| Friday | 2,693 |
| Saturday | 2,688 |
| Thursday | 2,611 |
| Wednesday | 2,389 |
| Tuesday | 2,255 |
| Sunday | 2,131 |
| Monday | 1,944 |

- **Trend:** Manhattan averages rose from 1,959 pickups per hour in January to 2,771 in June (+41%), but not steadily (February 2,557, March 2,233, April 2,289, May 2,539). Six months cannot separate trend from seasonality.

## 3. Which factor matters most?

R-squared of a regression of pickups on each factor (how much of the variation it explains):

| Scope | Factors | R-squared |
|---|---|---:|
| All boroughs | Borough only | **0.673** |
| All boroughs | Hour of day only | 0.043 |
| All boroughs | Weekday only | 0.003 |
| All boroughs | Month only | 0.003 |
| All boroughs | Borough x hour of day | **0.874** |
| All boroughs | Borough x hour + weekday + month | 0.881 |
| All boroughs | ... plus all weather variables | 0.881 |
| Manhattan | Hour of day only | **0.623** |
| Manhattan | Weekday only | 0.036 |
| Manhattan | Month only | 0.034 |
| Manhattan | Weather variables only | 0.091 |
| Manhattan | Hour + weekday + month | 0.696 |
| Manhattan | ... plus all weather variables | 0.701 |

**Location comes first, then time of day.** Weekday, month and weather add very little once those two are known. The Manhattan "weather only" figure (9%) mostly reflects weather proxying for time of day and season: adding weather to a time model lifts R-squared by only 0.5 points.

*Plausible reasons (not testable with this data):* Manhattan concentrates residents, workers, nightlife and visitors in a small area, and its demand follows commuting and going-out routines.

## 4. Weather

| Variable | Correlation with pickups, all rows | Manhattan only |
|---|---:|---:|
| Temperature (`temp`) | 0.05 | 0.16 |
| Dew point (`dewp`) | 0.03 | 0.08 |
| Wind speed (`spd`) | 0.01 | 0.06 |
| Visibility (`vsb`) | -0.01 | -0.04 |
| Sea-level pressure (`slp`) | -0.02 | -0.05 |
| Precipitation 1h / 6h / 24h | 0.00 / -0.00 / -0.02 | 0.02 / -0.02 / -0.07 |
| Snow depth (`sd`) | -0.01 | 0.00 |

After controlling for hour, weekday and month, the 394 Manhattan hours with measurable rain show about **214 more pickups per hour** (roughly 9% above the Manhattan average of 2,387). This is suggestive of a rain uplift, but hourly data are autocorrelated, so it is not a formal test and should be validated on more data.

## 5. Holidays

Only **7 distinct days** carry the holiday flag, and three of them are Mondays (the quietest weekday), so a raw holiday vs non-holiday average mixes a holiday effect with a weekday effect.

Raw averages: all boroughs 437 vs 492 pickups per hour (about -11%); Manhattan 2,036 vs 2,401 (about -15%); Brooklyn 527 vs 535 (about -1%); Queens 321 vs 309 (about +4%).

Day-level comparison against the average non-holiday day with the same weekday (% difference in daily pickups):

| Date | Weekday | Manhattan | Brooklyn | Queens |
|---|---|---:|---:|---:|
| Jan 1 | Thursday | -44% | +10% | -29% |
| Jan 19 | Monday | -32% | -15% | -20% |
| Feb 12 | Thursday | +3% | -7% | -10% |
| Feb 16 | Monday | +16% | +10% | +6% |
| May 10 | Sunday | +1% | +19% | +17% |
| May 25 | Monday | -27% | +41% | +33% |
| Jun 3 | Wednesday | +13% | +24% | +24% |

The Manhattan drop is driven by three days; on the other four, Manhattan demand was flat or higher. Boroughs also diverge: on May 25 Manhattan fell 27% while Brooklyn rose 41% and Queens 33%. With seven days there is not enough data for a reliable statistical test.

## 6. Recommendations

1. **Position supply in Manhattan for the evening peak:** concentrate driver availability and incentives at 18:00-22:00, especially Thursday and Friday, plus Saturday night.
2. **Use the hour x weekday view for incentive planning:** target windows where demand is high and supply is likely short, and avoid paying for supply in the 03:00-06:00 trough.
3. **Do not apply a blanket holiday policy:** effects differ by holiday and by borough. Build holiday-specific forecasts once there is more holiday history.
4. **Treat weather as a secondary signal:** test rain as an uplift flag in a forecasting model rather than building weather-driven staffing rules.
5. **Review EWR and Staten Island coverage:** near-zero volume suggests minimal service or incomplete data in 2015.
6. **Next step:** fit a demand forecasting model (for example gradient boosting) with borough, hour, weekday, holiday, weather and lagged pickups, using at least a full year of data.

## 7. Limitations

- **Six months of data (January-June 2015):** seasonality across a full year cannot be assessed, and only 7 holiday days are flagged.
- **Hourly data are autocorrelated:** significance tests treating each hour as independent would be misleading, so effects are described rather than tested.
- **Unknown borough:** 3,043 rows have no borough label; they carry under 0.1% of pickups, but the cause is unconfirmed.
- **Demand only:** no driver-supply, wait-time or price data, so the recommendations assume supply can follow demand.
- **Descriptive analysis:** associations, not causal effects.
