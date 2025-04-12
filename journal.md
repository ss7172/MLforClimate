# Project Journal: ML On Climate - Energy Consumption 

## Overview

The goal of this project inspired by the ASHRAE – Great Energy Predictor III challenge is to build predictive models for electricity consumption (meter_reading) using building metadata, historic usage, and weather data collected from 100 buildings across various sites worldwide. 
## Week 1 updates 

## challenges this week
    - A particular challenge has been handling missing values in the weather variables, with cloud_coverage having a notably high missing rate (~43%).

---

## 1. Data Exploration & EDA

### Data Loading and Summary
- **File:** `data/train.csv`
- **Parsing:** Timestamp column parsed with `parse_dates=['timestamp']`.
- **Information:** 
  - ~700K entries, 13 columns.
  - Columns include: building_id, timestamp, meter_reading, primary_use, square_feet, year_built, air_temperature, cloud_coverage, dew_temperature, precip_depth_1_hr, sea_level_pressure, wind_direction, wind_speed.
- **Summary Statistics:**  
  - Meter reading shows a skewed distribution with many zero readings and a long tail.
  - Weather variables have varying ranges and missing data.

### Visualizations
- **Missing Values:**  
  - Bar plot showing missing counts per column.
  - Cloud_coverage is missing about 43% of the time.
- **Meter Reading:**  
  - Histogram and boxplot (both raw and log-transformed) to assess distribution and identify outliers.
- **Categorical Analysis:**  
  - Bar chart of `primary_use` reveals its distribution.
- **Time Series Analysis:**  
  - Plot of meter_reading over time, indicating trends and seasonality.
- **Correlation Analysis:**  
  - Heatmap of correlations among numeric features (meter_reading, square_feet, year_built, weather variables, etc.).
  - Log-transformed meter_reading plotted to normalize target distribution.

---

## 2. Preprocessing and Missing Value Imputation for Minimal and Moderate Variables

### Strategy for Minimal Missingness
- **Minimal Columns:** `air_temperature`, `dew_temperature`, `precip_depth_1_hr`
- **Approach:**  
  - Applied `SimpleImputer` with strategy 'mean', using a `ColumnTransformer`.

### Strategy for Moderate Missingness
- **Moderate Columns:** `sea_level_pressure`, `wind_direction`
- **Approach:**  
  - Applied `SimpleImputer` with strategy 'median' via the same pipeline.

### Implementation
- A pipeline (`pipeline1`) was built using `make_pipeline` and `ColumnTransformer` to transform the dataset with imputation applied to the designated columns.
- After transformation, the data was converted back into a DataFrame and verified via `df_transformed.info()` and `df_transformed.isnull().sum()`.

---

## 3. Strategies to tackle missing Cloud Coverage

### Analysis by Building and Month
- **Building-Level Analysis:**  
  - Random sampling of building IDs and group-wise correlation analysis to understand the relationship between cloud_coverage and other weather features (air_temperature, dew_temperature, sea_level_pressure).
  - Uniform missing fraction across buildings (~74% to 79%), indicating that missingness is largely systematic (likely due to centralized weather data capture failures rather than building-specific issues).

- **Monthly Analysis:**  
  - Grouped data by month and calculated correlations.
  - Found that the correlation between cloud_coverage and other variables varied by month, indicating seasonal variation in cloud formation patterns.

### Temporal Patterns of Missingness
- **Daily Missingness:**  
  - Calculated the fraction of missing cloud_coverage per date and plotted as a line chart.
  - Noted variability over days, with some days experiencing up to ~80% missing and others near 0%.
- **Hourly Missingness:**  
  - Aggregated by hour to compute missing fractions.
  - Observed lower missingness during midday and higher rates during nighttime/early morning hours.

---
## Time‐Based Interpolation

**Rationale**  
Since each building’s records follow a temporal order, a straightforward approach is to interpolate `cloud_coverage` values over time for each building’s time series. By treating the data chronologically, we aim to preserve local patterns (e.g., gradual changes from hour to hour).

**Method Summary**  
1. **Sort & Group by Building**: The data is sorted by `building_id` and timestamp, ensuring each building’s weather observations form a continuous time series.  
2. **Time‐Dependent Filling**: Missing `cloud_coverage` entries are estimated by looking at neighboring timestamps, using a time‐based method that tries to remain faithful to each building’s local trends.  
3. **Edge Cases**: Any leading or trailing gaps are resolved with forward/backward fills, preventing persistent NaNs at the edges.

**Findings**  
- **Advantages**: Easy to implement, works directly off the timestamp ordering, and helps fill many missing values without requiring advanced modeling.  
- **Drawbacks**: Tends to smooth out fluctuations in cloud coverage, which may reduce variance and weaken correlations with other weather variables. Polynomial or spline variants often did not yield significant improvements in correlation.

---

## Multivariate Imputation (Iterative)

**Rationale**  
Time‐based interpolation alone might miss relationships with other weather parameters (like air temperature or dew temperature). A more sophisticated approach leverages several weather features and time indicators to “learn” how `cloud_coverage` behaves under different conditions.

**Method Summary**  
1. **Feature Inclusion**: Additional columns (e.g., `air_temperature`, `dew_temperature`, `sea_level_pressure`, `wind_speed`) provide contextual data, alongside time‐derived features (month, hour).  
2. **Iterative Process**: A predictive model systematically estimates missing `cloud_coverage` by treating it as a regression target, looping through multiple refinement steps.  
3. **Outcome**: This “multivariate” method often captures seasonal patterns or non‐linear effects better than a simple time interpolation.

**Findings**  
- **Advantages**: Improves correlations between `cloud_coverage` and other weather features, potentially providing more accurate estimates of missing values.  
- **Drawbacks**: Computationally heavier than a simple time‐based approach and more complex to tune. The direct impact on `meter_reading` predictions remains to be validated.



## Planned next steps for Missing cloud values
- Cluster based startegies
- Season specific imputation
- Modelling location dependencies without explicit geographic data
    - Using building id as a proxy for location 
    - Aggregate weather patterns per building and unsupervised clustering
    - Incorporating time based features for seasonality

--
## 4. Implementing Models on the different datasets created after EDA

