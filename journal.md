# Project Journal: ML On Climate - Energy Consumption and Cloud Coverage Imputation

## Overview

This project is a Kaggle in-class competition inspired by the ASHRAE – Great Energy Predictor III challenge. The goal is to build predictive models for electricity consumption (meter_reading) using building metadata, historic usage, and weather data collected from 100 buildings across various sites worldwide. A particular challenge has been handling missing values in the weather variables, with cloud_coverage having a notably high missing rate (~43%).

## Week 1 updates 
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
  - Applied `SimpleImputer` with strategy 'median' using a `ColumnTransformer`.

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

## 4. Cloud Coverage Imputation Strategies

### A. Time-Based Interpolation (Current Focus)
- **Method:**  
  - Group data by building and sort by timestamp.
  - Interpolate using the `method='time'` approach within each building’s time series.
  - Forward-fill and backward-fill are applied to handle edge cases.
- **Findings:**  
  - Post-interpolation, correlation with weather variables (e.g., air_temperature, dew_temperature) was lower than before, indicating that the interpolation smoothed out the natural variability.
- **Considerations:**  
  - Lower variance may reduce correlation because the real “ups and downs” of cloud_coverage have been flattened by the interpolation.
  - Options for improvement include using more sophisticated interpolation (polynomial/spline) or limiting interpolation over long gaps.

### B. Model-Based Imputation (Planned Next Step)
- **Method (Planned):**  
  - Use IterativeImputer with a RandomForestRegressor to impute cloud_coverage, leveraging other weather features and time features (hour, month).
- **Expected Benefits:**  
  - Capture non-linear relationships and seasonal effects more accurately.
  - Potentially preserve the natural variability in cloud_coverage better than time-based methods.

### C. Hybrid or Cluster-Based Approaches (Future Exploration)
- **Hybrid Approach:**  
  - Combine time-based interpolation for short gaps and model-based imputation for long gaps.
- **Cluster-Based Strategy:**  
  - Cluster buildings based on aggregate weather data and develop imputation models tailored to each cluster.

---

## 5. Production-Grade Code Implementation (For Time-Based Interpolation & Iterative Imputer)

- **Time-Based Interpolation:**  
  A production-grade function was developed that performs time-based interpolation on cloud_coverage on a per-building basis. The function handles timestamp conversion, groups data, interpolates using `method='time'`, fills edges with `ffill` and `bfill`, and logs the number of missing values before and after processing.

- **Model-Based Imputation:**  
  A separate production-grade function using IterativeImputer was implemented. It extracts time features, validates required columns, and uses RandomForestRegressor as the estimator for IterativeImputer to predict missing cloud_coverage.

---

## 6. Next Steps

- **Comparison:**  
  Evaluate downstream model performance (e.g., RMSE in predicting meter_reading) when using:
  - Cloud coverage imputed via
