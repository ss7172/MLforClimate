# Abstract

This project focuses on developing predictive models for **hourly electricity consumption** using data gathered from 100 buildings across multiple global sites. 
Each record combines:

- **Building Metadata:**  
  - `primary_use` (EnergyStar category)  
  - `square_feet` (gross floor area)  
  - `year_built`  
  - `floor_count`  

- **Weather Conditions:**  
  - `air_temperature`  
  - `cloud_coverage`  
  - `dew_temperature`  
  - `precip_depth_1_hr`  
  - `sea_level_pressure`  
  - `wind_direction`  
  - `wind_speed`  

- **Target Variable:**  
  - `meter_reading` in kWh  

The goal is to accurately forecast electricity usage by leveraging building attributes and environmental factors. This involves:

1. **Data Cleaning & Exploration**  
   - Identifying and handling anomalies (e.g., outliers or missing data)  
   - Aligning time zones, if necessary  

2. **Feature Engineering**  
   - Time-based features (hour of day, day of week, seasonal indicators)  
   - Weather-derived features (temperature lags, smoothed variables)  
   - Building characteristics (floor area, year built, etc.)  

3. **Model Development**  
   - **Gradient Boosted Trees** (LightGBM, XGBoost, CatBoost)  
   - **Random Forests**  
   - **Neural Networks** (LSTMs)  
   - **Ensembles** (stacking or averaging multiple models)  

By evaluating these approaches with appropriate metrics (e.g., RMSE), I seek to provide actionable insights for energy management and cost optimization. Ultimately, the project aims to enable data-driven decisions, enhance operational efficiency, and contribute to sustainable building operations.

Source of dataset:
https://www.kaggle.com/competitions/predicting-electricity-consumption/data
