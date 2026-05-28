# UpLift

### Predicting Transit Accessibility Failures Before They Strand a Rider

[![Status](https://img.shields.io/badge/status-design%20memo%20%7C%20seeking%20partners-orange)](https://github.com/)
[![Domain](https://img.shields.io/badge/domain-transit%20equity%20%7C%20disability%20justice-blue)](https://github.com/)
[![Algorithm](https://img.shields.io/badge/algorithm-XGBoost%20supervised%20classification-green)](https://github.com/)
[![Author](https://img.shields.io/badge/author-Nico%20Meyering%2C%20MPA-lightgrey)](https://www.linkedin.com/in/nicolasmeyering)

-----

## The Problem

I am a Disabled transit rider in Philadelphia. I depend on SEPTA elevators to reach platforms I cannot get to by stairs.

On more than one occasion, I have shown up at a station — sometimes running late, sometimes in the rain — and found the elevator out of service. No warning. No reroute. Just a sign taped to a gate.

That is not a Philadelphia problem. It is not even an American problem.

- **3.6 million** Americans with disabilities that affect travel do not leave their homes. Broken accessibility equipment is a direct contributor.
- The MTA in New York City has elevators at only **23% of its subway stations** — and those elevators break down regularly.
- A 2024 international review covering 34 studies found that physical infrastructure failure is one of the most consistently reported barriers to transit use for Disabled people, across every region studied.

Right now, transit agencies fix elevators and escalators **after they break**. A maintenance team responds to a failure report. By then, a Disabled rider has already been stranded.

**UpLift changes that.**

-----

## What It Does

UpLift is a supervised machine learning model that predicts which elevators, escalators, and platform lifts are likely to fail in the next **30 days** — so maintenance teams can act before the breakdown happens, not after.

Think of it like a **credit score for mechanical equipment**. A credit score synthesizes dozens of financial signals — payment history, utilization, age of accounts — into one number that predicts the likelihood of default. UpLift does the same with maintenance signals: equipment age, usage load, weather exposure, time since last service.

The output is one probability score per unit (0.00–1.00). The maintenance team receives a ranked list of at-risk equipment and uses it to schedule proactive service calls. Riders can optionally receive alerts through existing transit apps.

-----

```mermaid
flowchart LR
A[MTA Outage Data] --> C[XGBoost Model]
B[NOAA Weather] --> C
C --> D[Risk Score per Unit]
D --> E[Ranked Maintenance List]
D --> F[Rider Alert System]

## Model Design

**Algorithm:** XGBoost (eXtreme Gradient Boosting — supervised binary classification)  
**Prediction window:** 30-day forward failure probability  
**Unit of analysis:** Individual piece of accessibility equipment (elevator, escalator, platform lift)

### Inputs (X) — What the Model Learns From

|Feature                              |Description                            |
|-------------------------------------|---------------------------------------|
|Equipment type                       |Elevator, escalator, or platform lift  |
|Equipment age                        |Years since installation               |
|Manufacturer & model                 |Encoded categorical                    |
|Days since last scheduled maintenance|Recency of service                     |
|Unplanned outages (prior 12 months)  |Historical failure rate                |
|Total usage load                     |Estimated daily trips × days in service|
|Days since last parts replacement    |Component-level recency                |
|Most recent inspection score         |Agency inspection record               |
|Average monthly temperature          |NOAA weather data by station location  |
|Station average daily ridership      |Demand/stress proxy                    |
|Floors served                        |Mechanical complexity                  |
|Transfer hub status                  |Binary: major hub or not               |

### Label (Y) — What the Model Predicts

Binary: **1** = unplanned outage within the next 30 days / **0** = no outage

Labels are derived from official agency outage logs, matched to equipment ID by timestamp. Each training example represents one piece of equipment over one 30-day rolling window.

### Output

- Probability score (0.00–1.00) per unit
- Ranked priority list for maintenance scheduling
- Full Precision-Recall curve published per agency — each agency selects its own operating threshold based on budget and legal risk tolerance

-----

## The Two Errors — and Why They Are Not Equal

Every prediction model makes two kinds of mistakes. UpLift’s design is shaped by a deliberate asymmetric cost judgment.

|Error Type        |What Happens                                        |Cost                                                                                                                                |
|------------------|----------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
|**False Positive**|We flag equipment as at-risk. It does not break.    |Wasted service call: ~$500–$2,000. Recoverable. No one is harmed.                                                                   |
|**False Negative**|We miss a failure. Equipment breaks without warning.|A Disabled rider is stranded. Repeated patterns trigger ADA enforcement. Settlements have reached nine figures. **Not recoverable.**|


> **A false negative is categorically worse than a false positive.** Every technical decision in UpLift — the decision threshold (0.30, not 0.50), class weighting, SMOTE oversampling, F2-Score as the headline metric — reflects that asymmetry explicitly.

-----

## Data Sources

|Dataset                                                         |Use                          |Access                                                                                                             |
|----------------------------------------------------------------|-----------------------------|-------------------------------------------------------------------------------------------------------------------|
|MTA NYC Subway Elevator & Escalator Availability (2015–present) |Primary training dataset     |[data.ny.gov](https://data.ny.gov/Transportation/MTA-Subway-Elevator-and-Escalator-Availability-Beg/rc78-7x78)     |
|MTA LIRR Elevator & Escalator Availability (2019–present)       |Supplemental training        |[data.ny.gov](https://data.ny.gov/Transportation/MTA-LIRR-Elevator-and-Escalator-Availability-Begin/9hjt-526f)     |
|MTA Metro-North Elevator & Escalator Availability (2021–present)|Supplemental training        |[data.ny.gov](https://data.ny.gov/Transportation/MTA-Metro-North-Elevator-and-Escalator-Availabilit/ax67-8386)     |
|MetroPT Dataset (Porto Metro, Portugal)                         |Cross-system validation      |[arXiv:2207.05466](https://doi.org/10.48550/arXiv.2207.05466)                                                      |
|NOAA Climate Data Online                                        |Weather enrichment by station|[ncei.noaa.gov](https://www.ncei.noaa.gov/cdo-web/)                                                                |
|SEPTA Elevator Outages API                                      |Philadelphia real-time feed  |[OpenDataPhilly](https://opendataphilly.org/datasets/septa-elevator-outages/)                                      |
|WMATA Elevator & Escalator Incidents API                        |DC Metro real-time feed      |[developer.wmata.com](https://developer.wmata.com)                                                                 |
|National Transit Database (NTD) 2022–2024                       |National maintenance baseline|[USDOT Open Data](https://datahub.transportation.gov/Public-Transit/2022-2024-NTD-Annual-Data-Breakdowns/amkt-4ehs)|

All datasets are publicly accessible. No proprietary data required.

-----

## Evaluation Design

**Training set:** MTA outage data, January 2015 – December 2022  
**Holdout set:** January 2023 – December 2024 (truly unseen — no hyperparameter tuning on holdout)

**Primary metrics:**

- **F2-Score** (headline) — weights recall twice as heavily as precision; missing a real failure is twice as costly as a false alarm
- **AUC-ROC** — overall discriminative performance
- **Precision-Recall curve** — published per agency for threshold selection

-----

## Scaling Strategy

UpLift is designed to work for any transit system on earth, through four mechanisms:

1. **Standardized Feature Schema** — the 12 input variables are universal; every transit system tracks them in some form
1. **Transfer Learning** — train a strong base model on MTA NYC data; fine-tune locally for any new agency
1. **Federated Learning** — agencies share model parameters, not raw data; no agency needs to expose proprietary records
1. **Multi-System Data Consortium** — agencies contribute anonymized outage records in exchange for access to UpLift and its predictions

-----

## Current Status

> **Phase 2 - EDA in progress.** Design memo complete. Exploratory data analysis notebook now live. Seeking transit agency and development partners.

UpLift was developed as a capstone project through the [Equitech Futures Civic Tech Institute (CTI) 2026](https://www.equitechfutures.com/) cohort. The model architecture, feature schema, data strategy, and equity framework are fully designed. Implementation is the next step.

**I am actively looking for:**

- Transit agencies or civic tech collaborators interested in a pilot
- Data engineers or ML practitioners interested in building this out
- Fellowship or funding opportunities to support development
- See 'notebooks/01_eda_transit_outages.ipynb' for Phase 2 data exploration

-----

## Contact

**Nico Meyering, MPA**  
Philadelphia, PA  
[LinkedIn](https://www.linkedin.com/in/nicolasmeyering) · [nicmeyering@gmail.com](mailto:nicmeyering@gmail.com)

*Chairman, Philadelphia Mayor’s Commission on People with Disabilities*  
*Steering Committee, Transit Forward Philadelphia*  
*VP of Growth & Partnerships, Net Impact Philadelphia*

-----

## Key Citations

1. Bureau of Transportation Statistics. *Transportation Difficulties Keep Over Half a Million Disabled at Home.* <https://www.bts.gov/archive/publications/special_reports_and_issue_briefs/issue_briefs/number_03/entire>
1. TransitCenter. *Access Denied: Making the MTA Subway System Accessible to All New Yorkers.* 2019.
1. Veloso, B. et al. *The MetroPT Dataset for Predictive Maintenance.* arXiv, 2022. DOI: 10.48550/arXiv.2207.05466
1. Mwaka, C.R. et al. *Barriers and Facilitators of Public Transport Use Among People with Disabilities: A Scoping Review.* Frontiers in Rehabilitation Sciences, 2024.
1. DREDF. *ADA Topic Guide on Transit Equipment Maintenance.* <https://dredf.org/ADAtg/maint.shtml>

-----

*Developed through the Equitech Futures Civic Tech Institute, CTI 2026 Cohort · Philadelphia, PA*
