# Uber NYC Pickups Analysis

AI-assisted coding and data analysis case study exploring Uber pickups across New York City boroughs.

## Dataset
`Uber.csv` has 29,101 rows and 13 columns:

| Column | Description |
|---|---|
| pickup_dt | Pickup date and time |
| borough | NYC borough |
| pickups | Number of pickups |
| spd, vsb, temp, dewp, slp | Weather: wind speed, visibility, temperature, dew point, sea-level pressure |
| pcp01, pcp06, pcp24 | Precipitation over 1, 6 and 24 hours |
| sd | Snow depth |
| hday | Holiday flag |

## Notebook
`Second_case_study_AI_assisted_Coding_and_Data_Analysis.ipynb` covers:
- Loading data and summary statistics
- Missing values and NumPy operations
- Feature extraction, `loc` / `iloc`
- Groupby aggregations
- Univariate and bivariate plots, heat map

## Run
```
pip install pandas numpy seaborn matplotlib jupyter
jupyter notebook Second_case_study_AI_assisted_Coding_and_Data_Analysis.ipynb
```
Keep `Uber.csv` in the same folder as the notebook.
