# UpLift — Data Directory

This folder contains the dataset used to train and evaluate the UpLift predictive maintenance model.

-----

## Current File

### `uplift_equipment_data.csv`

A synthetic dataset of **2,000 equipment-window records** generated to mirror the structure, distributions, and class balance of real transit agency outage data. Each row represents one piece of accessibility equipment (elevator, escalator, or platform lift) observed over one 30-day rolling window.

This dataset was generated in `notebooks/01_data_generation_eda.ipynb` using statistically realistic parameters derived from publicly available MTA, SEPTA, and NTD records.

**It is not real agency data.** It is a demonstration pipeline. The model architecture, feature schema, evaluation framework, and equity design are production-ready. The data is the placeholder.

-----

## Schema

|Column                        |Type       |Description                                                                  |
|------------------------------|-----------|-----------------------------------------------------------------------------|
|`equipment_id`                |string     |Unique equipment identifier (e.g. EQ-0001)                                   |
|`equipment_type`              |categorical|elevator, escalator, or platform_lift                                        |
|`agency`                      |categorical|Transit agency (SEPTA, MTA_NYC, WMATA, CTA, MBTA)                            |
|`manufacturer`                |categorical|Equipment manufacturer                                                       |
|`equipment_age_years`         |float      |Years since installation                                                     |
|`days_since_last_maintenance` |float      |Days elapsed since last scheduled service                                    |
|`unplanned_outages_12mo`      |int        |Number of unplanned outages in prior 12 months                               |
|`total_usage_load_millions`   |float      |Estimated cumulative ridership load (millions)                               |
|`days_since_parts_replacement`|float      |Days since most recent component replacement                                 |
|`inspection_score`            |float      |Most recent agency inspection score (0–100)                                  |
|`avg_monthly_temp_f`          |float      |Average monthly temperature at station location (°F)                         |
|`avg_daily_ridership`         |float      |Station average daily ridership                                              |
|`floors_served`               |int        |Number of floors served by the unit                                          |
|`is_transfer_hub`             |binary     |1 = major transfer hub station, 0 = standard station                         |
|`outage_within_30_days`       |binary     |**Target label.** 1 = unplanned outage occurred within 30 days, 0 = no outage|

-----

## Real Data Sources

When real agency data is available, the following public datasets replace the synthetic file. All are openly accessible — no proprietary data required.

|Dataset                                                         |Replaces                     |Access                                                                                                             |
|----------------------------------------------------------------|-----------------------------|-------------------------------------------------------------------------------------------------------------------|
|MTA NYC Subway Elevator & Escalator Availability (2015–present) |Primary training data        |[data.ny.gov](https://data.ny.gov/Transportation/MTA-Subway-Elevator-and-Escalator-Availability-Beg/rc78-7x78)     |
|MTA LIRR Elevator & Escalator Availability (2019–present)       |Supplemental training        |[data.ny.gov](https://data.ny.gov/Transportation/MTA-LIRR-Elevator-and-Escalator-Availability-Begin/9hjt-526f)     |
|MTA Metro-North Elevator & Escalator Availability (2021–present)|Supplemental training        |[data.ny.gov](https://data.ny.gov/Transportation/MTA-Metro-North-Elevator-and-Escalator-Availabilit/ax67-8386)     |
|SEPTA Elevator Outages API                                      |Philadelphia validation      |[OpenDataPhilly](https://opendataphilly.org/datasets/septa-elevator-outages/)                                      |
|WMATA Elevator & Escalator Incidents API                        |DC Metro validation          |[developer.wmata.com](https://developer.wmata.com)                                                                 |
|NOAA Climate Data Online                                        |Weather enrichment by station|[ncei.noaa.gov](https://www.ncei.noaa.gov/cdo-web/)                                                                |
|NTD Breakdown Records 2022–2024                                 |National maintenance baseline|[USDOT Open Data](https://datahub.transportation.gov/Public-Transit/2022-2024-NTD-Annual-Data-Breakdowns/amkt-4ehs)|
|MetroPT Dataset (Porto Metro)                                   |Cross-system validation      |[arXiv:2207.05466](https://doi.org/10.48550/arXiv.2207.05466)                                                      |

-----

## How to Swap In Real Data

The pipeline is designed so that replacing the synthetic dataset requires changes in only one place.

**Step 1 — Prepare your agency data file**

Your real data file must match the schema above. Column names must be identical. The label column (`outage_within_30_days`) must be derived from your agency’s outage logs: a binary flag indicating whether an unplanned outage was recorded within 30 days of each observation window.

Save the prepared file as `uplift_equipment_data.csv` and place it in this `/data` folder, replacing the synthetic version.

**Step 2 — Rerun the notebooks in order**

```
notebooks/01_data_generation_eda.ipynb   ← skip data generation cells; run EDA only
notebooks/02_preprocessing_smote.ipynb   ← runs as-is on any conforming dataset
notebooks/03_model_training_evaluation.ipynb  ← runs as-is; outputs update automatically
```

All outputs in `/outputs` will regenerate with real data.

**Step 3 — Adjust the decision threshold if needed**

The default threshold is **0.30**. Transit agencies with tighter maintenance budgets may prefer a higher threshold (fewer flags, higher precision). Agencies with stronger ADA compliance obligations may prefer a lower threshold (more flags, higher recall). The Precision-Recall curve in `outputs/uplift_performance_curves.png` shows the full trade-off — select the operating point that matches your agency’s cost tolerance.

-----

## A Note on Class Imbalance

In real transit outage data, unplanned failures are the minority class — typically 15–30% of all equipment-window records. UpLift addresses this through SMOTE oversampling in `notebooks/02_preprocessing_smote.ipynb`, applied to the training set only. The holdout set is never resampled, ensuring evaluation reflects real-world class distributions.

-----

## Data Privacy

UpLift is designed for federated deployment. Agencies share **model parameters**, not raw equipment records. No agency is required to expose proprietary maintenance data to participate in a multi-agency UpLift consortium.

-----

*UpLift — Equitech Futures Civic Tech Institute, CTI 2026 · Philadelphia, PA*