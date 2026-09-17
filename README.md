# ECMWF Weather Forecast Verification and Renewable Energy Impact Analysis

## 📌 Project Overview

This project explores how real numerical weather forecasts can be processed, compared with observed weather conditions and connected to renewable-energy applications.

The project uses **ECMWF weather forecast data in GRIB format** and processes it with `ecCodes`, `cfgrib` and `xarray`. Forecasts for temperature, wind speed, cloud cover and solar radiation are extracted for selected German locations and multiple forecast horizons.

A small historical verification example is then carried out for Berlin by comparing an archived ECMWF forecast with observed German weather data.

In simple terms, this project tries to answer the question:

**How can weather forecasts be verified against observations, and how can forecast errors affect simplified wind and solar generation estimates?**

---

## 🎯 Project Objectives

The main objectives of this project are:

- To work with real ECMWF weather forecast data
- To read and process GRIB weather files
- To extract weather forecasts for selected German locations
- To analyse +6, +12, +24 and +48 hour forecast horizons
- To calculate wind speed from ECMWF wind components
- To convert accumulated solar radiation into an interpretable radiation value
- To match forecasts with observations using forecast valid time
- To evaluate forecast errors using MAE, RMSE, Bias and Correlation
- To demonstrate how wind and solar forecast errors can affect simplified renewable-energy estimates

---

## 🌍 Forecast Locations

Three German locations are used in the live forecast section:

- Berlin
- Hamburg
- Munich

The nearest ECMWF grid point is selected for each city because the exact city coordinates do not normally fall directly on a model grid point.

---

## ⏱️ Forecast Horizons

The project analyses four forecast lead times:

```text
+6 hours
+12 hours
+24 hours
+48 hours
```

A forecast lead time describes how far into the future a prediction is made from the model initialization time.

For example:

```text
Forecast initialization: 2025-09-01 00:00 UTC
Lead time: +12 hours
Valid time: 2025-09-01 12:00 UTC
```

The **valid time** is important because the forecast must be compared with the observation from the same actual time.

---

## 🌦️ Weather Variables

| Variable | Description | Final Unit |
|---|---|---|
| 2 m Temperature | Air temperature approximately 2 m above the surface | °C |
| 10 m Wind Speed | Calculated from ECMWF `u10` and `v10` wind components | m/s |
| Total Cloud Cover | Percentage of the sky covered by cloud | % |
| Solar Radiation | Surface solar radiation converted to average radiation | W/m² |

---

## 🗂️ Data Sources

### ECMWF Open Data

[ECMWF Open Data](https://www.ecmwf.int/en/forecasts/datasets/open-data) is used to demonstrate direct retrieval and processing of real operational weather forecasts in GRIB format.

### Open-Meteo Single Runs API

The [Open-Meteo Single Runs API](https://open-meteo.com/en/docs/single-runs-api) is used to retrieve an archived ECMWF forecast run for the historical verification example.

### Bright Sky / DWD Observations

Observed weather data are obtained through the [Bright Sky API](https://brightsky.dev/), which provides weather observations based on Deutscher Wetterdienst (DWD) station data.

---

## 🧩 GRIB Processing

GRIB is a binary meteorological format commonly used for numerical weather prediction data.

The ECMWF files are opened using:

```python
xr.open_dataset(
    "forecast.grib2",
    engine="cfgrib"
)
```

The resulting `xarray` dataset contains labelled weather dimensions and coordinates such as:

```text
latitude
longitude
forecast time
forecast step
valid time
```

This is useful because weather forecast data are multidimensional rather than ordinary tabular data.

---

## 🌡️ Global Temperature Forecast

The first step is to open a real ECMWF +24 hour temperature forecast and inspect the global model grid.

![Global Temperature Forecast](Outputs/ECMWF%20%2B24%20Hour%202%20m%20Temperature%20Forecast.png)

Temperature values are originally provided in Kelvin and are converted to Celsius using:

```text
Temperature (°C) = Temperature (K) - 273.15
```

---

## 🇩🇪 Germany Temperature Forecast

The global forecast grid is restricted to an approximate Germany bounding box before city-level values are extracted.

![Germany Temperature Forecast](Outputs/ECMWF%20%2B24%20Hour%20Temperature%20Forecast%20-%20Germany.png)

This step demonstrates spatial subsetting of gridded meteorological data.

---

## 💨 Wind Speed Calculation

ECMWF provides 10 metre wind using two horizontal components:

```text
u10 = east-west wind component
v10 = north-south wind component
```

Wind speed is calculated using:

```text
Wind Speed = √(u10² + v10²)
```

This converts the two wind components into one wind-speed value in metres per second.

---

## ☁️ Cloud Cover Processing

Total cloud cover is extracted from ECMWF forecast data.

The values are inspected before conversion because cloud cover may be represented as a fraction between 0 and 1. When required, the values are converted into percentages:

```text
0%   = clear sky
100% = completely overcast
```

---

## ☀️ Solar Radiation Processing

ECMWF surface solar radiation is stored as **accumulated energy**.

Therefore, the accumulated values cannot be treated directly as an instantaneous radiation measurement.

The energy received during consecutive 6-hour intervals is calculated as:

```text
Interval Energy =
Current Accumulation - Previous Accumulation
```

The interval energy is then divided by the number of seconds in six hours:

```text
Average Solar Radiation =
Interval Energy / 21,600 seconds
```

The final value is expressed in:

```text
W/m²
```

---

## 🔗 Combined Forecast Dataset

Temperature, wind speed, cloud cover and solar radiation are merged into one ECMWF forecast dataset.

The live forecast dataset contains:

```text
12 rows
10 columns
0 missing values
```

The 12 rows represent:

```text
3 locations × 4 forecast horizons
```

---

## 📊 Historical Forecast Verification

A historical ECMWF forecast initialized at:

```text
2025-09-01 00:00 UTC
```

is used for a small Berlin verification example.

The four forecast horizons are:

```text
+6h
+12h
+24h
+48h
```

Forecasts are matched with observed weather using `valid_time`.

The forecast error is calculated as:

```text
Error = Forecast - Observation
```

Therefore:

```text
Positive error → forecast was too high
Negative error → forecast was too low
Zero error     → forecast matched the observation
```

---

## 📐 Forecast Verification Metrics

Four basic metrics are used:

- **MAE** — average absolute size of the forecast error
- **RMSE** — similar to MAE but gives more weight to larger errors
- **Bias** — shows whether forecasts are generally too high or too low
- **Correlation** — shows how similarly forecast and observed values vary

Because the verification uses only four forecast cases, the correlation values should be interpreted carefully.

### Verification Results

| Variable | Unit | MAE | RMSE | Bias | Correlation |
|---|---|---:|---:|---:|---:|
| Temperature | °C | 0.67 | 0.77 | -0.23 | 0.99 |
| Wind Speed | m/s | 0.34 | 0.35 | -0.17 | 1.00 |
| Cloud Cover | percentage points | 22.00 | 38.06 | -15.50 | 0.72 |
| Solar Radiation | W/m² | 28.00 | 51.70 | 28.00 | 1.00 |

These results describe only the small Berlin demonstration sample and should **not** be interpreted as a general assessment of ECMWF forecast performance.

---

## 🌡️ Forecast vs Observed Temperature

![Temperature Forecast vs Observed](Outputs/ECMWF%20Forecast%20vs%20Observed%20Temperature%20-%20Berlin.png)

The temperature forecasts remain relatively close to the observed values across the four selected forecast cases.

---

## 💨 Forecast vs Observed Wind Speed

![Wind Forecast vs Observed](Outputs/ECMWF%20Forecast%20vs%20Observed%20Wind%20Speed%20-%20Berlin.png)

Wind-speed forecasts are also close to the observed values in this small sample.

---

## ☁️ Forecast vs Observed Cloud Cover

![Cloud Cover Forecast vs Observed](Outputs/ECMWF%20Forecast%20vs%20Observed%20Cloud%20Cover%20-%20Berlin.png)

Cloud cover shows larger differences, particularly in the +48 hour case.

---

## ☀️ Forecast vs Observed Solar Radiation

![Solar Radiation Forecast vs Observed](Outputs/ECMWF%20Forecast%20vs%20Observed%20Solar%20Radiation%20-%20Berlin.png)

The daytime solar-radiation forecast differs from the observed value, while the nighttime cases contain zero solar radiation.

---

## ⚡ Renewable Energy Impact Analysis

Weather forecast errors can affect renewable-energy generation estimates.

This project therefore uses two simple normalized generation proxies:

- Wind-power proxy
- Solar-power proxy

These proxies are used only to demonstrate the relationship between weather forecast errors and possible renewable-energy estimation errors.

They do **not** represent a real wind turbine, wind farm or solar installation.

---

## 🌬️ Wind Power Proxy

A simplified wind-turbine power curve is used:

```text
Wind speed below 3 m/s    → 0
Wind speed from 3–12 m/s  → increasing normalized generation
Wind speed from 12–25 m/s → 1
Wind speed above 25 m/s   → 0
```

The output is normalized between:

```text
0 = no generation
1 = approximate rated generation
```

![Wind Power Proxy](Outputs/Wind%20Power%20Proxy%20-%20Berlin.png)

The purpose is to demonstrate how a wind-speed forecast error can produce a difference in a simplified generation estimate.

---

## ☀️ Solar Power Proxy

The solar proxy is calculated using:

```text
Solar Power Proxy = Solar Radiation / 1000
```

The result is restricted to a range from 0 to 1.

For example:

```text
0 W/m²    → 0.0
500 W/m²  → 0.5
1000 W/m² → 1.0
```

![Solar Power Proxy](Outputs/Solar%20Power%20Proxy%20-%20Berlin.png)

This is **not an actual PV power model**. It does not include panel efficiency, temperature, orientation, shading, inverter losses or installed capacity.

Its purpose is only to show how a solar-radiation forecast error can produce a difference in a simplified solar-generation estimate.

---

## 🔎 Key Findings

For the small Berlin verification sample:

- Temperature forecast errors were relatively small.
- Wind-speed forecast errors were also small.
- Cloud-cover errors were larger, especially in the +48 hour case.
- The daytime solar-radiation forecast differed from the observed value.
- Wind and solar weather errors produced corresponding differences in the simplified renewable-energy proxies.

These findings apply only to the demonstration sample used in this project.

---

## ⚠️ Limitations

The project has several important limitations:

- The detailed historical verification uses one Berlin forecast run.
- Only four forecast lead times are compared.
- One case at each lead time is not enough to establish a general relationship between forecast accuracy and lead time.
- Correlation values are calculated from only four observations and should therefore be interpreted cautiously.
- ECMWF forecasts represent model grid points, while observations come from nearby DWD weather stations.
- The wind-power proxy does not represent a specific turbine or wind farm.
- The solar-power proxy does not include real PV-system characteristics.
- The results should not be interpreted as a comprehensive assessment of ECMWF forecast skill.

---

## 🔮 Future Improvements

Future work could extend the project by:

- Using a larger historical verification period
- Including more German locations
- Comparing forecast accuracy across seasons
- Analysing more forecast initialization times
- Using real wind-turbine power curves
- Using a more realistic PV-generation model
- Adding precipitation verification
- Creating an interactive weather dashboard

---

## ✅ Conclusion

This project demonstrates a complete student-level weather forecast verification workflow using real meteorological data.

The main outcomes are:

- ECMWF GRIB forecast files were successfully processed with `ecCodes`, `cfgrib` and `xarray`.
- Temperature, wind speed, cloud cover and solar radiation were extracted for selected German locations.
- Multiple forecast horizons were analysed.
- Historical ECMWF forecasts were matched with observed weather using valid time.
- Forecast errors were evaluated using MAE, RMSE, Bias and Correlation.
- Simplified wind and solar proxies demonstrated how weather forecast errors can influence renewable-energy estimates.

The project focuses on demonstrating the **weather-data processing and forecast-verification workflow** rather than claiming to provide a complete evaluation of ECMWF performance or a real renewable-energy generation model.

---

## 🛠️ Technologies Used

```text
Python
Pandas
NumPy
xarray
ecCodes
cfgrib
ECMWF Open Data
Open-Meteo API
Bright Sky / DWD
scikit-learn
Matplotlib
Jupyter Notebook
```

---

## 🛠️ Installation

Install the required Python packages:

```bash
pip install ecmwf-opendata xarray cfgrib eccodes pandas numpy matplotlib scikit-learn requests
```

---

## ▶️ How to Run

Open the Jupyter Notebook:

```bash
jupyter notebook weather_forecast_verification.ipynb
```

Run the notebook cells in order.

The notebook downloads live ECMWF forecast files, so an internet connection is required.

---

## 📈 Final Results Summary

| Task | Result |
|---|---|
| Live forecast locations | Berlin, Hamburg, Munich |
| Forecast horizons | +6h, +12h, +24h, +48h |
| Weather variables | Temperature, Wind Speed, Cloud Cover, Solar Radiation |
| Historical verification | Berlin |
| Temperature MAE | 0.67 °C |
| Wind Speed MAE | 0.34 m/s |
| Cloud Cover MAE | 22 percentage points |
| Solar Radiation MAE | 28 W/m² |
| Renewable analysis | Simplified Wind and Solar Power Proxies |

---

## 📂 Project Structure

```text
weather-forecast-verification/
│
├── README.md
├── weather_forecast_verification.ipynb
├── Outputs/
│   ├── global_temperature_forecast.png
│   ├── germany_temperature_forecast.png
│   ├── temperature_forecast_vs_observed.png
│   ├── wind_forecast_vs_observed.png
│   ├── cloud_cover_forecast_vs_observed.png
│   ├── solar_radiation_forecast_vs_observed.png
│   ├── wind_power_proxy.png
│   └── solar_power_proxy.png
└── ecmwf_forecasts.csv
```

The GRIB files used during processing can be kept locally and do not need to be committed to GitHub.

---

## 👨‍🎓 Author

Pratik Prakash Gawde  
MSc Data Analytics Student  
BSBI – Berlin School of Business and Innovation
