# 🌦️ Weather Trend Forecasting

> **Tech Assessment — Data Science & Forecasting**
> Submitted as part of the PM Accelerator application process.

---

## 📌 About PM Accelerator

**Product Manager Accelerator Mission** 

By making industry-leading tools and education available to individuals from all backgrounds, we level the playing field for future PM leaders. This is the PM Accelerator motto, as we grant aspiring and experienced PMs what they need most – Access. We introduce you to industry leaders, surround you with the right PM ecosystem, and discover the new world of AI product management skills.

🔗 [www.pmaccelerator.io](https://www.pmaccelerator.io)

---

## 📊 Project Overview

This project analyzes the **Global Weather Repository** dataset (145,212 rows,
41 columns, spanning May 2024 – June 2026) to forecast future weather trends
and uncover patterns across 257 cities worldwide.

The project targets the **Advanced Assessment**, covering:
- Full data cleaning and feature engineering pipeline
- Exploratory data analysis with 9 visualizations
- Anomaly detection using Isolation Forest
- Three forecasting models + an ensemble
- Advanced analyses: feature importance (SHAP), spatial mapping, geographical
  patterns, and air quality correlation

---

## 📁 Repository Structure

weather-trend-forecasting/
├── notebooks/
│   ├── 01_eda.ipynb                
│   ├── 02_preprocessing.ipynb     
│   ├── 03_models.ipynb            
│   └── 04_advanced_analysis.ipynb 
├── outputs/                       
│   ├── 01_global_temp_trend.png
│   ├── 02_temp_by_continent.png
│   ├── 03_choropleth_temp.png
│   ├── 04_precip_histogram.png
│   ├── 05_precip_vs_humidity.png
│   ├── 06_wettest_driest.png
│   ├── 07_correlation_heatmap.png
│   ├── 08_city_temp_trends.png
│   ├── 09_seasonal_decomp_tokyo.png
│   ├── 10_anomaly_scatter.png
│   ├── 11_anomaly_timeseries.png
│   ├── 12_sarima_forecast.png
│   ├── 13_prophet_forecast.png
│   ├── 14_xgboost_forecast.png
│   ├── 15_all_models_comparison.png
│   ├── 16_model_comparison.png
│   ├── 17_feature_importance.png
│   ├── 18_shap_summary.png
│   ├── 19_choropleth_anomaly_rate.html
│   ├── 20_temp_by_continent.png
│   ├── 21_air_quality_correlation.png
│   ├── minmax_scaler.pkl
│   ├── xgb_model.pkl
│   └── model_metrics.csv
├── .gitignore
├── requirements.txt
└── README.md

> **Note:** The `data/` folder is gitignored. Download the dataset manually
> (instructions below) before running the notebooks.

---

## ⚙️ Setup

### 1. Clone the repository
```bash
git clone https://github.com/Rehaan-2006/weather-trend-forecasting.git
cd weather-trend-forecasting
```

### 2. Create and activate a virtual environment
```bash
python3 -m venv venv
source venv/bin/activate        # macOS / Linux
# venv\Scripts\activate         # Windows
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Download the dataset
The raw dataset is not committed to this repository.
Download it from Kaggle and place it in the `data/` folder:

🔗 [Global Weather Repository — Kaggle](https://www.kaggle.com/datasets/nelgiriyewithana/global-weather-repository)

```bash
mkdir data
# Place GlobalWeatherRepository.csv inside data/
```

Or use the Kaggle CLI:
```bash
pip install kaggle
kaggle datasets download -d nelgiriyewithana/global-weather-repository -p data/ --unzip
```

---

## ▶️ How to Run

Run the notebooks **in order**. Each notebook depends on the output of the one before it.

```bash
jupyter lab
```

| Step | Notebook | Produces |
|------|----------|----------|
| 1 | `01_eda.ipynb` | 9 EDA plots in `outputs/` |
| 2 | `02_preprocessing.ipynb` | `data/df_clean.csv` |
| 3 | `03_models.ipynb` | Forecast plots + `model_metrics.csv` |
| 4 | `04_advanced_analysis.ipynb` | Anomaly plots + SHAP + spatial maps |

---

## 🔬 Methodology

### Data Cleaning 
- Replaced hidden sentinel values (`-9999`) in air quality columns with `NaN`
- Clipped physically impossible values (wind > 500 kph, pressure > 1100 mb)
- Used time-aware **linear interpolation per city** (limit = 6 periods) for
  missing values — median imputation was intentionally avoided as it destroys
  time-series autocorrelation
- Flagged outliers using the IQR method without deleting rows, preserving the
  integrity of the time series

### Feature Engineering 
- Parsed `last_updated` into `month`, `hour`, `day_of_year`
- Created `season` and `continent` columns
- Built lag features grouped by city: `lag_1`, `lag_7`, `rolling_7`
  (the 257 NaNs in `lag_1` are expected — one per city's first record)

### Anomaly Detection 
- Trained **Isolation Forest** (`contamination=0.01`) on 4 core features:
  temperature, humidity, wind speed, pressure
- Detected **1,437 anomalies** (1% of records)
- Top anomaly countries: Kuwait (172), Iceland (142), Iraq (119) —
  consistent with extreme climate profiles

### Forecasting Models 
- **SARIMA(1,1,1)(1,1,0,7)** — classical statistical baseline
- **Prophet** — with yearly + weekly seasonality (2 full years confirmed in dataset)
- **XGBoost** — trained on lag features and calendar variables
- **Ensemble** — weighted average (30% SARIMA + 30% Prophet + 40% XGBoost)
- Dynamic 80/20 train/test split by time (no random splitting)
- Representative city: **Accra, Ghana** (743 daily observations)

---

## 📈 Results

| Model | RMSE | MAE | MAPE % | R² |
|-------|------|-----|--------|----|
| SARIMA | 1.523 | 1.270 | 4.73% | -0.630 |
| Prophet | 2.234 | 1.877 | 6.97% | -2.507 |
| XGBoost | 1.318 | 0.984 | 3.79% | -0.221 |
| **Ensemble** | **1.317** | **1.091** | **4.08%** | **-0.220** |

> **Note on R²:** Accra is a tropical equatorial city with very low temperature
> variance (~3°C annual range). For such low-variance series, R² is an
> unreliable metric — RMSE and MAPE are the appropriate measures of accuracy.
> An RMSE of 1.3°C and MAPE of 3.79% represent strong performance.

**Key finding — Feature Importance:** The 7-day rolling average (`rolling_7`)
was the dominant predictor at 0.70 importance, confirming that recent
temperature history is far more predictive than calendar features for a
tropical city with no strong seasonality.

---

## 🗺️ Advanced Analyses

- **Anomaly Detection:** Isolation Forest flagged extreme combined weather
  states; Summer had 2× more anomalies than Winter (656 vs 459)
- **Spatial Analysis:** Interactive choropleth map of anomaly rates by country
  (`outputs/19_choropleth_anomaly_rate.html`)
- **Geographical Patterns:** Monthly average temperature trends broken down
  by continent over 2 years
- **Feature Importance:** XGBoost built-in importance + SHAP summary plots
- **Environmental Impact:** Correlation heatmap of air quality indices against
  temperature, humidity, wind speed, and pressure

---

## 📦 Dependencies

See `requirements.txt`. Key libraries:
pandas, numpy, matplotlib, seaborn, plotly
scikit-learn, statsmodels, prophet, xgboost, shap
kaggle, jupyterlab

---

## 🎥 Demo Video

🔗 [YouTube link here]

---

*Submitted by Rehaan | PM Accelerator Tech Assessment*