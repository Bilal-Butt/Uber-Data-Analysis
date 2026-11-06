# Full Notebook Narrative

All markdown text from [`uber_nyc_pickups_analysis.ipynb`](../notebooks/uber_nyc_pickups_analysis.ipynb) in order: problem statement, data dictionary, explanation of each step, observations after each chart and table, summary of findings, recommendations and limitations. Code is omitted; see the notebook.

---

# Problem Statement :    Uber Data Analysis

## Context
 Uber Technologies, Inc. is an American multinational transportation network company based in San Francisco and has operations in approximately 72 countries and 10,500 cities. In the fourth quarter of 2021, Uber had 118 million monthly active users worldwide and generated an average of 19 million trips per day.

Ridesharing is a very volatile market and demand fluctuates wildly with time, place, weather, local events, etc. The key to being successful in this business is to be able to detect patterns in these fluctuations and cater to demand at any given time.

As a newly hired Data Scientist in Uber's New York Office, you have been given the task of extracting insights from data that will help the business better understand the demand profile and take appropriate actions to drive better outcomes for the business. Your goal is to identify good insights that are potentially actionable, i.e., the business can do something with it.
## Objective
To extract actionable insights around demand patterns across various factors.

## Data Dictionary

**The data contains information about the weather, location, and pickups.**

| Column | Description |
| --- | --- |
| `pickup_dt` | Date and time of the pick-up |
| `borough` | NYC's borough |
| `pickups` | Number of pickups for the period |
| `spd` | Wind speed in miles/hour |
| `vsb` | Visibility in miles to the nearest tenth |
| `temp` | Temperature in Fahrenheit |
| `dewp` | Dew point in Fahrenheit |
| `slp` | Sea level pressure |
| `pcp01` | 1-hour liquid precipitation |
| `pcp06` | 6-hour liquid precipitation |
| `pcp24` | 24-hour liquid precipitation |
| `sd` | Snow depth in inches |
| `hday` | Holiday (Y) or not (N) |

## Questions to explore
1. What are the different variables that influence pickups?

2. Which factor affects the pickups the most? What could be plausible reasons for that?

3. What are your recommendations to Uber management to capitalize on fluctuating demand?


## Guidelines
1. Perform univariate analysis on the data to better understand the variables at your disposal.

2. Perform bivariate analysis to better understand the relationships between different variables.

3. Create visualizations to explore data and extract the insights.

### Explanation

- `describe(exclude="number")` switches the summary to text columns, showing the count, number of unique values, the most frequent value, and its frequency.
- `value_counts()` breaks that down further, listing exactly how many records fall into each category.
- We can already see `borough` has six categories and `hday` has two (`Y` and `N`), with far fewer holidays than regular days, which makes sense in six months of data (January-June 2015).

This kind of quick categorical check tells us whether a variable is balanced or skewed, which matters later when comparing pickups across categories.

# **Topic 2: Data Cleaning and a NumPy Check**

> Check if there are any missing values in the data. If `borough` has missing values, replace them with the label `"Unknown"` instead of dropping the rows.

**What the data shows.** 3,043 rows (10.5%) had no borough. They are extra rows with their own timestamps (not duplicates of labelled rows) and average only 2.1 pickups, so together they make up well under 0.1% of all pickups. Labelling them `"Unknown"` keeps their weather information without distorting the borough totals.

### Explanation

- `df.isna().sum()` adds up the missing values in every column. Only `borough` has gaps here.
- `fillna("Unknown")` replaces every missing value in that column with the label `"Unknown"` rather than removing the row. Dropping rows would also throw away the weather data attached to them, which we still need.
- Running `isna().sum()` again confirms the fix worked.

**Key takeaway:** Missing data should be handled deliberately. Dropping rows is not always the safest option, especially when the rest of the row still carries useful information.

> The `pickups` column is a whole number for every row. Can you show me how NumPy computes basic statistics on it directly, and how that compares with pandas' built-in functions?

### Explanation

- `to_numpy()` converts a pandas Series into a plain NumPy array. A pandas Series is, underneath, a NumPy array with a label attached to each value.
- `np.mean()`, `np.std()`, and `np.percentile()` are NumPy functions that operate directly on the array. They return the same mean as pandas' `.mean()` because pandas uses NumPy internally to do the math.
- This matters because it explains why pandas feels fast: every column-wide calculation is really a NumPy operation running under the hood.

Knowing both tools helps when you need a quick statistic (`NumPy`) versus when you need it tied back to a labeled DataFrame for further work (`pandas`).

# **Topic 3: Selecting and Filtering Data**

### Explanation

- `pd.to_datetime()` converts the text column into an actual datetime type, unlocking the `.dt` accessor used to pull out month, day, weekday, and hour.
- The condition `(df["borough"] == "Manhattan") & (df["pickups"] > 5000)` is **conditional indexing**: pandas checks each row against both conditions and keeps only the rows where both are true. The `&` combines conditions, and each condition needs its own parentheses.
- `.loc[:, [...]]` selects rows and columns by **label**. The `:` keeps every row from `high_demand`, and the list picks exactly the columns we want to see.

**Key takeaway:** Conditional indexing lets you slice a DataFrame down to only the records relevant to a question, without writing a loop.

### Explanation

- `iloc` selects by **integer position**, not by column name. `0:5` takes the first five rows and `0:3` takes the first three columns, whatever they happen to be named.
- `loc` is safer when you know the column names and want to be explicit. `iloc` is useful when you are working positionally, for example, always wanting "the first few columns" regardless of what they are called.

Both return the same kind of result here because the first three columns in `high_demand` happen to be `borough`, `pickups`, and `temp`.

# **Topic 4: GroupBy and Statistical Functions**

### Explanation

- `groupby("borough")` splits the data into one group per borough. `["pickups"].mean()` then computes the average pickups within each group.
- `sort_values(ascending=False)` orders the result from highest to lowest average, so the busiest borough appears first.
- The same pattern applied to `hday` compares holidays against regular days.

**Key takeaway:** `groupby` is how you answer "average of X, broken down by Y" without writing a loop. It is one of the most common operations in exploratory analysis.

### Explanation

- Passing a list, `["borough", "hday"]`, to `groupby()` creates one group for every combination of borough and holiday status.
- `round(1)` cleans up the output to one decimal place, which is easier to compare at a glance.
- This confirms whether the overall holiday effect is consistent everywhere or whether one borough behaves differently, which is exactly the kind of detail that turns into a business recommendation.

**What the data shows.** Average pickups per hour are heavily concentrated: Manhattan 2,387, Brooklyn 534, Queens 309, Bronx 51, Staten Island 1.6 and EWR 0.02. Manhattan alone produces about 73% of all pickups, Brooklyn 16% and Queens 9%. Across all boroughs, holiday hours average 437 pickups vs 492 on other hours (about -11%); the drop is mostly Manhattan (2,401 to 2,036, about -15%), while Brooklyn is nearly flat and Queens is slightly higher. Section 7 below tests whether that holiday pattern holds up.

# **Topic 5: Univariate Visualization**

### Explanation

- The histogram shows how often different pickup counts occur, while the box plot above it marks the median, the quartiles, and any outliers as individual points. Placing them together makes it easy to connect a spike in the histogram with its position in the box plot.
- The dashed green line marks the mean and the solid black line marks the median. When they are far apart, the distribution is skewed, which is exactly what we see here: a long right tail of very high-pickup hours.
- The bar graph converts `value_counts()` into percentages, showing whether boroughs are represented evenly in the dataset.

**Key takeaway:** Univariate plots describe one variable at a time. A skewed distribution like `pickups` often calls for a log transform later if you plan to model it, and it also tells you the mean alone would be a misleading summary.

**What the data shows.** `pickups` is strongly right-skewed (skewness 3.0): the mean is 490 against a median of 54, with a maximum of 7,883. About 19% of rows have zero pickups, almost all in EWR (4,243 of its 4,343 hours) and Staten Island (1,291 hours); each of the other four boroughs has zero pickups in only 8-9 hours. Each named borough has exactly 4,343 rows (14.9% of records), so the bar chart shows balanced *coverage*, not balanced demand. Because of the skew, any model should be fitted per borough or on a log scale.

# **Topic 6: Bivariate and Multivariate Visualization**

### Explanation

- `sns.lineplot()` with `hue="borough"` draws one line per borough, all on the same axes, so hourly patterns can be compared directly. `errorbar=None` keeps the chart clean by removing the confidence band.
- The scatter plot places every ride as a point, with `temp` on one axis and `pickups` on the other. `alpha=0.3` makes each point partly transparent, so overlapping points show up as darker patches, revealing where most of the data sits.
- Together these are **bivariate** plots: each one shows the relationship between two variables, rather than describing one variable alone.

**Key takeaway:** Line plots are best for a trend over an ordered variable like time. Scatter plots are best for spotting whether two continuous variables move together.

**What the data shows.** Manhattan has a pronounced daily cycle: a smaller morning peak around 09:00 (about 2,434 pickups per hour on average), a dip around midday, then a rise through the afternoon to the main peak at 19:00 (about 4,344), and the lowest demand at 04:00-05:00 (about 600). Brooklyn and Queens show the same overnight dip and a gentle evening rise but a much flatter pattern during the day, and the Bronx is nearly flat. The temperature scatter shows no usable pattern: the correlation between pickups and temperature is 0.05 across all rows and 0.16 within Manhattan, and all other weather variables are even closer to zero.

### Explanation

- `pivot_table()` reshapes the data into a grid: one row per hour, one column per weekday, and each cell holds the average pickups for that combination.
- Reordering the columns with `weekday_order` keeps the days in their natural sequence instead of alphabetical order.
- `sns.heatmap()` colors each cell by its value. Darker or more intense cells mark the busiest hour-and-day combinations, making the peak windows visible at a glance instead of having to scan a table of numbers.

**Key takeaway:** A heat map is the fastest way to spot patterns across two categorical or ordered variables at once. It combines what a line plot and a table would each show separately into a single view.

**What the data shows.** The busiest Manhattan hour-and-weekday cells are Thursday 19:00 (about 5,090 pickups), Friday 19:00 (5,030), Friday 20:00 (4,980), Saturday 23:00 (4,820) and Saturday 20:00 (4,790). By weekday, Friday (2,693), Saturday (2,688) and Thursday (2,611) are busiest and Monday (1,944) is quietest. Weekday evenings and late Friday/Saturday nights are the windows where supply matters most.

# **Topic 7: Which Factors Matter Most? (Quantified)**

The charts suggest that location and time of day dominate. This section measures it: how much of the variation in pickups does each factor explain (R-squared from a regression on that factor), and does weather or the holiday flag add anything once time is accounted for?

**Answer to "which factor affects pickups the most?"** Location first, then time of day. Borough alone explains **67%** of the variation in pickups, and borough combined with hour of day explains **87%**. Weekday, month and all nine weather variables together add less than one percentage point (88.1%). Within Manhattan, hour of day alone explains **62%**; adding weekday and month raises that to 70%, and weather adds only about 0.5 points (69.6% to 70.1%). A "weather only" model looks better in Manhattan (9%), but most of that is weather proxying for time of day and season.

*Plausible reasons for the dominance of location and hour (not tested in this data):* Manhattan concentrates residents, workers, nightlife and visitors in a small area, and its demand follows commuting and going-out routines.

**Weather.** No weather variable has a correlation above 0.16 with pickups. After controlling for hour, weekday and month, hours with measurable rain in Manhattan show about **214 more pickups per hour** (roughly 9% above the Manhattan average). This is suggestive of a rain uplift, but hourly observations are not independent, so it is not a formal test and should be validated on more data.

**Holidays.** Only **7 distinct days** carry the holiday flag (Jan 1, Jan 19, Feb 12, Feb 16, May 10, May 25 and Jun 3), and three of them are Mondays, the quietest weekday, so a raw holiday-vs-non-holiday average mixes the holiday effect with a weekday effect. Compared with the average non-holiday day of the same weekday, Manhattan demand fell sharply on **Jan 1 (-44%), Jan 19 (-32%) and May 25 (-27%)**, but was flat or higher on the other four days (+1% to +16%). On May 25, Manhattan fell 27% while Brooklyn rose 41% and Queens 33%. So the overall Manhattan holiday drop of about 15% is driven by three days, and the holiday effect is inconsistent across holidays and boroughs. Seven days are too few for a reliable statistical test.

**Trend over the six months.** Average Manhattan pickups per hour rose from 1,959 in January to 2,771 in June (+41%), though not steadily (March is below February). With only six months of data, a growth trend cannot be separated from seasonality.

# **Summary of Findings, Recommendations and Limitations**

## Answers to the guiding questions

1. **What variables influence pickups?** Borough (location) and hour of day are by far the strongest. Weekday and month matter a little. Weather variables and the holiday flag add very little once time is accounted for.
2. **Which factor affects pickups the most, and why?** Location: borough alone explains 67% of the variation, and borough with hour of day 87%. Manhattan accounts for about 73% of all pickups. Plausible reasons (not testable with this data): density of residents, workers and nightlife, and routine commuting and going-out patterns.
3. **Recommendations:** see below.

## Key findings

- **Manhattan dominates demand** (2,387 pickups per hour on average, about 73% of all pickups); EWR and Staten Island are near zero.
- **Time of day is the main driver within a borough:** Manhattan peaks at 19:00 (about 4,300) and bottoms out at 04:00-05:00 (about 600). The busiest cells are Thursday and Friday around 19:00-20:00 and Saturday evening and late night.
- **Weather has a weak effect:** correlations are 0.16 or less, and weather adds about 0.5 points of explained variance in Manhattan once time is controlled. Rain hours show a suggestive uplift of about 9%.
- **Holidays are inconsistent:** the overall Manhattan holiday drop (about -15%) is driven by three of seven flagged days; the other four were flat or higher, and boroughs behave differently (on May 25, Manhattan -27%, Brooklyn +41%, Queens +33%).
- **Demand grew from January to June** (Manhattan +41%), but six months cannot separate trend from seasonality.

## Recommendations for Uber management

1. **Position supply in Manhattan for the evening peak:** concentrate driver availability and incentives at 18:00-22:00, especially Thursday and Friday, plus Saturday night.
2. **Use the hour x weekday view for incentive planning:** target windows where demand is high and supply is likely short, and avoid paying for supply in the 03:00-06:00 trough.
3. **Do not apply a blanket holiday policy:** effects differ by holiday and by borough. Build holiday-specific forecasts once there is more holiday history.
4. **Treat weather as a secondary signal:** test rain as an uplift flag in a forecasting model rather than building weather-driven staffing rules.
5. **Review EWR and Staten Island coverage:** EWR has zero pickups in 4,243 of 4,343 hours and Staten Island in 1,291, which suggests minimal service or incomplete data in 2015.
6. **Next step:** fit a demand forecasting model (for example gradient boosting) with borough, hour, weekday, holiday, weather and lagged pickups, using at least a full year of data.

## Limitations

- **Six months of data (January-June 2015):** seasonality across a full year cannot be assessed, and only 7 holiday days are flagged.
- **Hourly data are autocorrelated:** significance tests treating each hour as independent would be misleading, so effects are described rather than tested.
- **Unknown borough:** 3,043 rows have no borough label; they carry under 0.1% of pickups, but the cause of the missing labels is unconfirmed.
- **Demand only:** there is no driver-supply, wait-time or price data, so the recommendations assume supply can follow demand.
- **Descriptive analysis:** associations, not causal effects.
