# Data Dictionary — UpLift

One row in the training dataset = one piece of accessibility equipment
(elevator, escalator, or platform lift) over one 30-day rolling window.

---

## Equipment Features

| Column Name | Data Type | Description | Source |
|---|---|---|---|
| equipment_id | string | Unique identifier for the equipment unit | Agency outage logs |
| equipment_type | string | Elevator, escalator, or platform lift | Agency records |
| equipment_age_years | float | Years since installation | Agency records |
| manufacturer | string | Equipment manufacturer (encoded categorical) | Agency records |
| model | string | Equipment model (encoded categorical) | Agency records |
| floors_served | integer | Number of floors the unit connects | Agency records |
| transfer_hub | integer | 1 = major transfer station; 0 = standard station | Agency records |

---

## Maintenance History Features

| Column Name | Data Type | Description | Source |
|---|---|---|---|
| days_since_last_maintenance | integer | Days since most recent scheduled service call | Agency maintenance logs |
| unplanned_outages_12mo | integer | Number of unplanned outages in the prior 12 months | Agency outage logs |
| days_since_parts_replacement | integer | Days since most recent component replacement | Agency maintenance logs |
| last_inspection_score | float | Most recent agency inspection score | Agency inspection records |
| total_usage_load | float | Estimated daily trips × total days in service | Agency ridership data |

---

## Environmental Features

| Column Name | Data Type | Description | Source |
|---|---|---|---|
| avg_monthly_temp | float | Average monthly temperature at station location (°F) | NOAA CDO |
| station_avg_daily_ridership | integer | Average daily riders at the station | Agency ridership data |

---

## Label

| Column Name | Data Type | Description | Source |
|---|---|---|---|
| failure_next_30_days | integer | 1 = unplanned outage occurred within the next 30 days; 0 = no outage | Agency outage logs |

---

## Notes

- Unit of analysis: one equipment unit × one 30-day rolling window = one training row
- Training set: MTA NYC data, January 2015 – December 2022
- Holdout set: January 2023 – December 2024 (unseen during training)
- Class imbalance expected: failures are the minority class
- SMOTE oversampling applied during training to address imbalance
- Decision threshold set at 0.30 (not default 0.50) to minimize false negatives
- Primary metric: F2-Score (recall weighted 2x over precision)
