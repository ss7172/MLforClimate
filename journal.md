# Project Journal: ML On Climate - Energy Consumption and Cloud Coverage Imputation

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

## 3. Investigation of Missing Cloud Coverage

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
