# Dhaka AQI benchmark with a brick-kiln season feature

Companion code for:

> Chakraborty, A., Santo, I. A., Kabir, J., **Mahmood, A. A.**, Sajid, A., & Siow, C. L. (2026).
> *Air Quality Crisis Prediction in Dhaka: Benchmarking Machine Learning with Seasonal Brick Kiln Features.*
> 7th Multimedia University Engineering Conference (MECON2026), Malaysia.

An open, reproducible **demonstration of the paper's method** on public data. It is not the original experiment
code and its numbers are not the paper's results.

## What it does

1. Downloads daily PM2.5 for central Dhaka and converts it to the U.S. EPA AQI (2024 breakpoints).
2. Joins daily weather: mean temperature, relative humidity, maximum wind speed, rainfall.
3. Builds lagged AQI and PM2.5 features and a binary **brick-kiln season** flag (November to March).
4. Benchmarks **Random Forest, XGBoost and Support Vector Regression** under three protocols: 10-fold
   cross-validation stratified by season, an 80/20 season-stratified split (both as in the paper), and a
   **chronological hold-out on the final 12 months**, the stricter test for a forecasting task.
5. Runs an ablation with and without the kiln flag, with and without AQI history.

## Data

| Source | Use | Access |
|---|---|---|
| Open-Meteo Air Quality API (CAMS global reanalysis) | PM2.5, default | free, no key; from August 2022 |
| Open-Meteo Historical Weather API (ERA5) | weather | free, no key |
| U.S. Embassy Dhaka monitor (AirNow) | measured PM2.5, optional | hourly CSVs into `data/raw/` |

## Run it

```bash
pip install -r requirements.txt
python src/benchmark.py                  # reanalysis PM2.5, Aug 2022 to Dec 2025
python src/benchmark.py --source airnow  # measured PM2.5 from data/raw/*.csv
```

## Results on the default data

1,239 daily observations, August 2022 to December 2025. R² on the chronological hold-out (last 12 months):

| Model | Weather only | Weather + kiln flag | Lags + weather | Lags + weather + kiln flag |
|---|---|---|---|---|
| RandomForest | 0.618 | 0.611 | 0.849 | 0.849 |
| XGBoost | 0.578 | 0.578 | 0.820 | 0.824 |
| SVR | 0.679 | 0.667 | 0.826 | 0.830 |

Full metrics for all three protocols: [`results/metrics.csv`](results/metrics.csv).

![Monthly mean AQI](results/monthly_aqi.png)
![Chronological hold-out](results/holdout_xgboost.png)

## What the ablation shows

AQI history carries most of the predictive signal: adding lagged AQI lifts R² from about 0.6 to about 0.83 to 0.85.
On this reanalysis data the binary kiln flag adds almost nothing on top. The likely reason is that in Dhaka the
kiln season coincides with the dry, cool, low-wind months, so temperature, rainfall and wind already encode most
of the same seasonal information. Separating kiln emissions from meteorology needs measured PM2.5 (the
`--source airnow` option) or a direct measure of kiln activity, such as satellite fire or thermal-anomaly counts
over the kiln clusters around the city. That is the natural next step.

## Structure

```
src/data.py        data download, AirNow loader, PM2.5 to AQI
src/features.py    lags, rolling mean, kiln-season flag, feature sets
src/benchmark.py   models, evaluation protocols, figures
results/           metrics and figures written by benchmark.py
```

## Licence

MIT. Data remain under their providers' terms (Open-Meteo: CC BY 4.0; CAMS and ERA5: Copernicus licence).
