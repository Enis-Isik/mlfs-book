Air Quality Forecasting Service (MLOps)

This project implements an end-to-end Machine Learning Operations (MLOps) pipeline to forecast air quality levels (specifically $PM_{2.5}$) for Bologna, Italy. The system leverages Hopsworks as a Feature Store to manage historical and daily data, utilizing an XGBoost regressor for predictions. The pipeline is designed to be fully automated, handling backfilling, daily feature updates, model training, and batch inference.

Project Overview

The core objective of this repository is to establish a robust infrastructure for air quality forecasting. The project has evolved from a baseline configuration to an advanced implementation using time-series feature engineering to improve predictive accuracy.

Key Capabilities:
Data Ingestion: Fetches air quality data from the World Air Quality Index (WAQI) API.
Feature Store: Manages feature groups and training datasets using Hopsworks.
Feature Engineering: Utilizes lagged features (autocorrelation) to capture temporal dependencies in pollution data.
Inference: Generates a 7-day air quality forecast and monitors model performance via hindcasting.

Configuration & Setup
1. PrerequisitesHopsworks Account: You need a project set up on Hopsworks.ai.WAQI API Key: A free API key from the AQICN service.
2. Environment Variables: Security is handled via a local .env file. You must create this file in the root directory to store your credentials securely. Required Variables:
HOPSWORKS_API_KEY=your_hopsworks_api_key
HOPSWORKS_PROJECT=your_project_name
HOPSWORKS_HOST=your_host_url
AQICN_API_KEY=your_aqicn_api_key

Pipeline ArchitectureThe project is organized into four distinct stages implemented via Jupyter Notebooks:

1. Backfill & Feature Engineering (1_air_quality_feature_backfill.ipynb)

This notebook initializes the system. It downloads historical data for the Via John Cage sensor in Bologna. Feature Engineering: Calculates lagged values for $PM_{2.5}$ (previous 1, 2, and 3 days) to improve model accuracy. Creates Feature Groups (air_quality_hourly_features, weather_hourly_features) in Hopsworks. Ingests the historical data into the Feature Store.

2. Daily Feature Pipeline (2_daily_feature_pipeline.ipynb)

This operational notebook runs daily (e.g., via GitHub Actions). It fetches the latest air quality and weather data.Applies the same lag feature logic as the backfill process.Inserts the new data into the existing Feature Groups.

3. Model Training (3_model_training.ipynb)

Creates a Feature View combining weather data and the lagged pollution features (pm25_lag_1, pm25_lag_2, pm25_lag_3).Trains an XGBoost regression model.Registers the best-performing model to the Hopsworks Model Registry.

4. Batch Inference (4_daily_batch_inference.ipynb)

Downloads the registered model.Generates predictions for the next 7 days.Creates a Hindcast Graph to visualize actual vs. predicted values for validation. Updates the forecasting dashboard.

Model Improvements:

The initial baseline model relied solely on weather data. To improve performance (Grade C implementation), we introduced Time-Series Feature Engineering. The Logic: Air quality exhibits high autocorrelation—the pollution level today is heavily dependent on the levels from yesterday. By feeding the model the $PM_{2.5}$ values from $t-1$, $t-2$, and $t-3$ (days), the model can understand the "momentum" of pollution trends rather than relying purely on meteorological proxies.

Results: Improved Mean Squared Error (MSE) compared to the baseline. Validation: The hindcast graph demonstrates a tighter      correlation between predicted and observed values.

Troubleshooting and Implementation:

NotesGeolocation & User-Agent FixDuring the backfill process, you may encounter an HTTP 403 Forbidden error when using geopy to retrieve coordinates for Bologna. This occurs because the default user agent is often blocked by the Nominatim service.The Solution: We have overridden the default geolocation function to use a custom user_agent string:Python# Custom user_agent prevents 403 blocks
geolocator = Nominatim(user_agent="air_quality_project_balda_v1")
This ensures consistent retrieval of coordinates (approx. 44.48, 11.35) without hard-coding them, preserving code modularity for other cities.Connecting to HopsworksEnsure you restart your Jupyter kernel after updating the .env file to ensure the new environment variables are loaded into the runtime memory.

Data SourceSensor Location: Via Jhon Cage, Bologna, Italy. Source: World Air Quality Index (AQICN)
