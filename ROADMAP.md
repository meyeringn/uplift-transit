# UpLift — Development Roadmap

> Predicting transit accessibility failures before they strand a rider.

This roadmap tracks UpLift from its current state (fully designed,
seeking partners) through pilot deployment with a transit agency.
Each phase has a clear goal, defined outputs, and identified
resource needs.

---

## Current Status: Phase 1 Complete ✅

The model architecture, feature schema, data strategy, equity framework,
and asymmetric error cost structure are fully designed and documented.
The design memo is available in this repository.

**What's needed to move forward:** A transit agency data partner,
ML practitioner, or fellowship/funding opportunity to support Phase 2.

---

## Phase 1 — Design & Framework ✅ Complete
**Goal:** Define the problem, the model, and the equity principles
before writing a single line of code.

- [x] Problem scoping: identified the gap between reactive
maintenance (fix it after it breaks) and the cost borne
by Disabled riders when that failure arrives without warning
- [x] Feature schema: 12 input variables covering equipment age,
usage load, maintenance history, weather, and station context
- [x] Label definition: binary classification — unplanned outage
within the next 30 days, derived from agency outage logs
- [x] Asymmetric error cost framework: false negatives
(missed failures that strand riders) defined as categorically
worse than false positives (unnecessary service calls)
- [x] Data source inventory: 8 publicly accessible datasets
identified, documented, and linked — MTA NYC primary training,
MetroPT cross-system validation, SEPTA and WMATA real-time feeds
- [x] Algorithm selection: XGBoost with SMOTE, class weighting,
F2-Score as headline metric (recall weighted 2x over precision),
decision threshold set at 0.30
- [x] Scaling strategy: standardized feature schema, transfer learning,
federated learning, multi-system data consortium
- [x] Design memo published

---

## Phase 2 — Data Assembly & Exploration 🔄 In Progress
**Goal:** Pull real outage data from identified public sources,
understand its structure, and confirm the feature schema
holds against actual records.

- [ ] Download and clean MTA NYC Subway Elevator & Escalator
Availability data (2015–present, data.ny.gov)
- [ ] Download MTA LIRR and Metro-North supplemental datasets
- [ ] Pull MetroPT dataset for cross-system validation
(Porto Metro, Portugal — arXiv:2207.05466)
- [ ] Connect to SEPTA Elevator Outages API (OpenDataPhilly)
- [ ] Connect to WMATA Elevator & Escalator Incidents API
- [ ] Pull NOAA Climate Data Online weather records
matched by station location
- [ ] Pull NTD 2022–2024 national maintenance baseline
- [ ] Build unified training dataset: one row per equipment unit
per 30-day rolling window, 2015–2022
- [ ] Publish exploratory data analysis notebook (EDA)
- [ ] Validate label distribution: confirm class imbalance
and SMOTE parameters

**Outputs:** Clean training dataset · EDA notebook ·
Data dictionary · Feature correlation analysis

---

## Phase 3 — Prototype Model 🔲 Planned
**Goal:** Train an initial XGBoost model on MTA historical data,
evaluate on holdout set, and publish results transparently.

- [ ] Train XGBoost baseline model on MTA data, 2015–2022
- [ ] Evaluate on holdout set: January 2023 – December 2024
(truly unseen — no hyperparameter tuning on holdout)
- [ ] Generate SHAP feature contribution summaries
— identify which variables drive failure prediction most
- [ ] Publish Precision-Recall curve and threshold
selection rationale at 0.30
- [ ] Validate F2-Score as headline metric;
confirm false negative rate is acceptably low
- [ ] Test transfer: does MTA-trained model generalize
to SEPTA outage patterns?
- [ ] Document failure cases: which equipment types
or failure modes does the model miss?

**Outputs:** Trained model · Evaluation report ·
SHAP outputs · Published notebook

---

## Phase 4 — Transit Agency Pilot 🔲 Planned
**Goal:** Run UpLift against real upcoming maintenance windows
with a transit agency partner and validate predictions
against observed outages.

- [ ] Identify pilot partner: transit agency with accessible
outage logs and willingness to share equipment metadata
- [ ] Establish data sharing agreement
- [ ] Run shadow deployment: generate 30-day failure predictions
without operational use, compare to actual outage records
- [ ] Present ranked maintenance list to agency maintenance team
for review and feedback
- [ ] Debrief: what did the model catch? What did it miss?
What would proactive scheduling have changed?
- [ ] Publish pilot results openly — including what didn't work

**Target partners:** SEPTA · WMATA · MTA NYC ·
Any transit agency with public outage data and an ADA compliance team

---

## Phase 5 — Deployment & Scale 🔲 Future
**Goal:** Move from shadow deployment to operational use
in at least one agency, with a path to multi-system scale.

- [ ] Build a maintenance dashboard: ranked at-risk equipment list,
probability scores, recommended service windows
- [ ] Integrate with existing agency work order systems
- [ ] Build optional rider-facing alert layer:
"this elevator is flagged for service this week"
pushed through existing transit apps
- [ ] Develop federated learning framework:
agencies share model parameters, not raw data
- [ ] Launch multi-agency data consortium:
contribute anonymized outage records,
receive access to UpLift predictions
- [ ] Publish replication guide for agencies
without in-house data teams

---

## How to Help

If you work in transit, disability access, civic tech,
or public infrastructure — or if you're a data engineer
or ML practitioner interested in this problem — I want to hear from you.

**I am actively looking for:**
- Transit agencies interested in a shadow deployment pilot
- Data engineers or ML practitioners to collaborate on Phases 2–3
- Fellowship or funding opportunities to support full development

**Contact:** [LinkedIn](https://www.linkedin.com/in/nicolasmeyering)
· meyeringn@gmail.com
