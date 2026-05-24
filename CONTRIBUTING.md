# Contributing to UpLift

Thank you for your interest in UpLift. This project is currently in the design and partner-seeking phase — the model architecture, feature schema, and data strategy are fully specified, and implementation is the next step.

Contributions are welcome across several areas.

-----

## What We’re Looking For

### Transit Agency Partners

If you work at a transit agency with historical elevator and escalator maintenance records, we’d like to talk. The data strategy is designed around publicly available MTA NYC data, but any agency with structured outage logs can participate. Anonymized data sharing and federated learning approaches are both on the table.

### ML / Data Engineering

If you have experience with:

- Supervised classification (XGBoost, Random Forest)
- Imbalanced class handling (SMOTE, class weighting)
- Time-series feature engineering from maintenance logs
- Federated learning frameworks

…this project needs you. The design is ready. The build is not.

### Disability Justice and Transit Equity Advocates

UpLift is a disability equity tool first and a machine learning project second. If you work in transit accessibility advocacy, ADA compliance, or Disabled rider organizing, your perspective on how outputs are communicated and deployed matters enormously. We want community validators in the loop — not just at launch, but throughout design.

### Civic Tech and Open Data

If you have experience scraping and archiving real-time transit APIs (WMATA, SEPTA, CTA, TfL), building out the historical data record is one of the most valuable contributions you can make right now.

-----

## How to Reach Out

This project is not yet at the pull request stage. If you’re interested in contributing, please reach out directly:

**Nico Meyering, MPA**
[LinkedIn](https://www.linkedin.com/in/nicolasmeyering) · [nicmeyering@gmail.com](mailto:nicmeyering@gmail.com)

Please include a brief note on your background and how you’d like to be involved. I respond to everyone.

-----

## Equity Commitments

Any implementation of UpLift will include:

- A post-training equity audit testing whether predicted maintenance priority correlates with neighborhood income or demographics
- Published Precision-Recall curves so each agency selects its own threshold — we do not make that decision for them
- Uncertainty estimates included in all outputs
- Minimum manual inspection schedules preserved in all agency contracts — UpLift is a decision-support tool, not a replacement for human judgment

Contributors are expected to uphold these commitments in any work they submit.

-----

## Code of Conduct

This project is committed to a welcoming environment for everyone, with particular attention to Disabled contributors and community members most affected by transit accessibility failures. Harassment of any kind will not be tolerated.

-----

*UpLift was developed through the Equitech Futures Civic Tech Institute, CTI 2026 Cohort.*
