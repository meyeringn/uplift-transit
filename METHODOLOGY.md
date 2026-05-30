# UpLift — Methodology

This document explains the technical and ethical design decisions behind UpLift. Every choice in the model — the algorithm, the threshold, the metric, the sampling strategy — was made deliberately, with a specific equity rationale. This document is for transit agency partners, fellowship reviewers, and technical collaborators who want to understand not just *what* UpLift does, but *why* it was built this way.

-----

## The Core Design Question

Before any technical decisions were made, one question had to be answered:

> **What does it actually cost when a Disabled rider shows up and the elevator is broken?**

That question shapes everything that follows.

-----

## Algorithm Selection: Why XGBoost

UpLift uses **XGBoost** (eXtreme Gradient Boosting), a supervised binary classification algorithm.

XGBoost was selected over alternatives for four reasons:

**1. Performance on tabular data**
Transit maintenance records are structured, tabular data — equipment IDs, inspection scores, outage counts, temperature readings. XGBoost consistently outperforms deep learning approaches on this data type without requiring large datasets or GPUs.

**2. Native handling of mixed feature types**
The UpLift feature schema includes continuous variables (equipment age, temperature), count variables (prior outages), and encoded categoricals (equipment type, agency). XGBoost handles these natively without extensive preprocessing.

**3. Compatibility with SHAP explainability**
Transit agencies cannot act on a black box. Maintenance schedulers need to know *why* a unit was flagged — is it age? Weather? A pattern of prior failures? XGBoost’s tree structure integrates cleanly with SHAP (SHapley Additive exPlanations), producing feature-level explanations for every prediction.

**4. Interpretable probability outputs**
XGBoost produces calibrated probability scores (0.00–1.00) per equipment unit, not just binary predictions. This enables the tiered risk ranking system (Monitor / Elevated / High / Critical) that makes UpLift operationally useful.

-----

## The Asymmetric Cost Framework

Every binary classifier makes two kinds of errors. UpLift treats them as fundamentally unequal.

|Error             |What Happens                                     |Cost                                                                                                                                                                      |
|------------------|-------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|**False Positive**|Equipment flagged as at-risk. It does not fail.  |A service technician is dispatched unnecessarily. Estimated cost: $500–$2,000. Recoverable. No rider is harmed.                                                           |
|**False Negative**|Failure missed. Equipment breaks without warning.|A Disabled rider is stranded. The agency may face ADA enforcement action. Legal settlements in transit accessibility cases have reached nine figures. **Not recoverable.**|

This asymmetry is not subtle. A wasted service call costs money. A missed failure costs a person their commute, their dignity, and potentially their safety. These are not equivalent outcomes, and the model should not treat them as such.

Every downstream technical decision — the decision threshold, the evaluation metric, the sampling strategy — follows directly from this framework.

-----

## Decision Threshold: Why 0.30, Not 0.50

A standard binary classifier flags a unit as at-risk when its predicted probability exceeds **0.50**. UpLift uses **0.30**.

The threshold controls the trade-off between precision and recall:

- **Higher threshold** → fewer flags, higher precision, more missed failures
- **Lower threshold** → more flags, higher recall, fewer missed failures

At 0.50, the model minimizes total errors. At 0.30, the model minimizes false negatives — the costlier error — at the expense of some additional false positives.

Given the asymmetric cost framework above, this is the correct trade-off. A maintenance team dispatched to a unit that turns out to be fine loses a few hours. A Disabled rider stranded at a broken elevator loses their access to the city.

**Threshold is not fixed.** The Precision-Recall curve published in `outputs/uplift_performance_curves.png` shows the full operating range. Transit agencies with tighter maintenance budgets may reasonably select a higher threshold. Agencies facing active ADA compliance pressure may select a lower one. UpLift surfaces the trade-off; the agency makes the call.

-----

## Class Imbalance: Why SMOTE

In real transit outage data, unplanned equipment failures are the minority class. Across the MTA, SEPTA, and NTD datasets that informed UpLift’s design, unplanned failures represent approximately 15–30% of all equipment-window records.

Without intervention, a naive classifier learns to predict “no failure” for every record and achieves high accuracy while being operationally useless. Predicting “no failure” 80% of the time when 80% of units don’t fail is not intelligence — it is inertia.

UpLift addresses class imbalance through **SMOTE** (Synthetic Minority Over-sampling Technique), applied exclusively to the training set.

SMOTE generates synthetic examples of the minority class (failure events) by interpolating between existing minority examples in feature space — not by simple duplication. This forces the model to learn the boundary between failure and non-failure more precisely.

**Critical constraint:** SMOTE is applied to the training set only. The holdout evaluation set is never resampled. Applying SMOTE to the holdout set would produce an artificially optimistic evaluation that does not reflect real-world class distributions. UpLift’s reported metrics reflect performance on unmodified, real-distribution holdout data.

-----

## Primary Metric: Why F2-Score

UpLift reports **F2-Score** as its headline metric, not accuracy or F1-Score.

**Why not accuracy?**
Accuracy rewards the model for correct predictions regardless of which class it gets right. On an imbalanced dataset, accuracy is gamed by predicting the majority class. A model that predicts “no failure” for every record achieves 80% accuracy and catches zero failures.

**Why not F1-Score?**
F1-Score weights precision and recall equally. UpLift’s asymmetric cost framework establishes that recall (catching real failures) is more valuable than precision (avoiding false alarms). F1 does not reflect this.

**Why F2-Score?**
F2-Score weights recall twice as heavily as precision. Formally:

```
F2 = (1 + 2²) × (precision × recall) / (2² × precision + recall)
   = 5 × (precision × recall) / (4 × precision + recall)
```

This directly encodes the judgment that missing a real failure is twice as costly as a false alarm. The metric matches the ethics.

Secondary metrics reported: AUC-ROC (overall discriminative performance), Average Precision (area under the Precision-Recall curve), and the full classification report at the operating threshold.

-----

## Feature Schema: Design Rationale

The 12 input features were selected to be:

1. **Universal** — every transit agency tracks these variables in some form, enabling cross-agency deployment without custom data pipelines
1. **Publicly accessible** — no proprietary data required; all features can be derived from open agency records and NOAA weather data
1. **Physically meaningful** — each feature has a clear mechanical or operational interpretation that maintenance staff can validate

|Feature                     |Why It’s Included                                                                       |
|----------------------------|----------------------------------------------------------------------------------------|
|Equipment age               |Older equipment fails more frequently — fundamental mechanical reality                  |
|Days since last maintenance |Recency of service is a direct failure risk indicator                                   |
|Unplanned outages (12 mo)   |The single strongest predictor; past failure patterns predict future failures           |
|Inspection score            |Agency-validated quality signal; low scores correlate with near-term failure            |
|Days since parts replacement|Component age independent of unit age; critical for escalator chains and elevator cables|
|Total usage load            |Cumulative mechanical stress; high-ridership stations wear equipment faster             |
|Average monthly temperature |Weather-correlated failure spikes documented across MTA and WMATA data                  |
|Average daily ridership     |Demand proxy; transfer hubs experience more stress than local stations                  |
|Floors served               |Mechanical complexity; more floors = more cable, more motor load                        |
|Transfer hub status         |Binary flag capturing extreme demand concentration at major stations                    |
|Equipment type              |Escalators and platform lifts fail at systematically different rates than elevators     |
|Manufacturer                |Encoded categorical; manufacturer-specific failure patterns exist in real data          |

-----

## Scaling Strategy: From Prototype to Deployment

UpLift is designed to generalize beyond any single agency through four mechanisms:

**Standardized Feature Schema**
The 12 input variables are universal. Every transit system tracks them in some form. An agency integrating UpLift maps their existing data fields to the schema — no custom model architecture required.

**Transfer Learning**
A base model trained on MTA NYC data (the largest publicly available elevator/escalator dataset) can be fine-tuned on smaller agency datasets with limited local records. The base model provides strong priors; local fine-tuning corrects for agency-specific equipment fleets and operating environments.

**Federated Learning**
Agencies share model parameters, not raw data. A WMATA-trained model update improves the SEPTA deployment without WMATA ever exposing proprietary maintenance records. Privacy is preserved by design.

**Multi-Agency Data Consortium**
Participating agencies contribute anonymized outage records to a shared training pool in exchange for access to UpLift predictions and model updates. Each agency’s contribution improves the model for all participants.

-----

## What This Is Not

UpLift does not replace human maintenance judgment. It ranks equipment by failure probability so that maintenance teams can allocate limited resources more effectively. The maintenance scheduler decides what to do with the ranked list — UpLift informs that decision, it does not make it.

UpLift also does not predict every failure. A 65% recall rate on the holdout set means approximately 35% of failures in that window were not caught. Real-world deployment will require continued monitoring, threshold adjustment, and model retraining as new outage data accumulates.

The goal is not perfection. The goal is that fewer Disabled riders show up to a broken elevator with no warning.

-----

*UpLift — Equitech Futures Civic Tech Institute, CTI 2026 · Philadelphia, PA*  
*Nico Meyering, MPA · [LinkedIn](https://www.linkedin.com/in/nicolasmeyering)*