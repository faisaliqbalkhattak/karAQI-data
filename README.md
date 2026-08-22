# karAQI-data

Data repository for the [karAQI](https://github.com/faisaliqbalkhattak/karAQI) air quality forecasting system. This repo stores the output of automated CI/CD pipelines — model predictions, trained model files, and evaluation metrics — so the main karAQI repo stays focused on source code.

---

## What lives here

| File | Updated by | Frequency | What it contains |
|---|---|---|---|
| `data/static_forecast.json` | forecast_pipeline | Every hour | 72-hour AQI predictions (30 outputs) + Open-Meteo reference forecast |
| `data/model_eval.json` | training_pipeline | Daily | Model comparison metrics, SHAP values, rolling-origin results, EDA data |
| `models/*.joblib` | training_pipeline | Daily | Trained Ridge, Random Forest, XGBoost models |
| `models/*.keras` | training_pipeline | Daily | Trained LSTM model |
| `models/*_models.json` | training_pipeline | Daily | Model manifests with metrics and feature schemas |

---

## How the data flows

```
karAQI (code repo)                      This repo (karAQI-data)
───────────────────                     ────────────────────────

training_pipeline (daily 00:00 UTC)
  → Trains Ridge, RF, XGBoost, LSTM
  → Champion comparison
  → Pushes model files ──────────────►  models/*.joblib, *.keras
  → Pushes eval JSON ────────────────►  data/model_eval.json

forecast_pipeline (hourly :04)
  → Loads model from this repo
  → Runs inference
  → Pushes forecast JSON ────────────►  data/static_forecast.json

Dashboard (Streamlit Cloud)
  ← Reads via raw GitHub URLs ◄──────  data/static_forecast.json
  (near-instant, zero runtime inference)
```

---

## Why a separate repo?

The training pipeline commits model files and JSON data every hour/day. If these were in the main karAQI repo, the commit history would be dominated by automated data commits, making it hard to see human code changes. This repo isolates automated data traffic so the main repo's commit history stays clean and readable.

---

## File formats

### `data/static_forecast.json`

Generated every hour by the forecast pipeline. Contains:

```json
{
  "generated_at": "2026-08-22T10:34:28+00:00",
  "origin": "2026-08-22T15:00:00",
  "model": "aqi-hourly",
  "current_aqi": { "aqi": 113.0, "category": "Unhealthy for Sensitive Groups", ... },
  "outputs": [
    { "output": "aqi_plus_01h", "value": 113.0, "category": "..." },
    ...30 outputs total (24 hourly + 4 six-hour + 2 twelve-hour means)
  ],
  "ref_forecast": [
    { "time": "2026-08-22T15:00", "aqi": 104.0 },
    ...up to 72 hours of Open-Meteo AQ reference
  ]
}
```

### `data/model_eval.json`

Generated daily by the training pipeline. Contains:

- `registry` — registered models and their champion status
- `hourly_holdout` — chronological holdout metrics for all hourly models
- `daily_holdout` — chronological holdout metrics for all daily models
- `rolling_origin` — rolling-origin evaluation metrics
- `eda_hourly_dist` — hourly AQI category distribution
- `eda_hourly_ts` — hourly time series for the last 90 days

### `models/`

Trained model artifacts. The forecast pipeline downloads the Ridge model from here before running inference. All models are trained daily by the training pipeline in the karAQI repo.

---

## How to use this data

### For the dashboard (Streamlit)

```python
import json, requests

url = "https://raw.githubusercontent.com/faisaliqbalkhattak/karAQI-data/main/data/static_forecast.json"
response = requests.get(url, timeout=10)
forecast = response.json()
print(f"Current AQI: {forecast['current_aqi']['aqi']}")
```

### For model evaluation

```python
url = "https://raw.githubusercontent.com/faisaliqbalkhattak/karAQI-data/main/data/model_eval.json"
response = requests.get(url, timeout=10)
eval_data = response.json()
for model in eval_data['hourly_holdout']:
    print(f"{model['model']:20s} {model['group']:20s} RMSE={model['rmse']:.2f}")
```

---

## Data sources

All data originates from Open-Meteo (free, keyless API wrapping Copernicus CAMS reanalysis). No API keys are required. The data represents modeled/reanalysis atmospheric conditions, not ground-station measurements.

---

## License

This repository contains automated outputs from the karAQI project. See the [main repo](https://github.com/faisaliqbalkhattak/karAQI) for project details and licensing.
