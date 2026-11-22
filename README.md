# Spend Crew Time Where the Oil Is

**Team:** Data Visionaries  
**Members:** Hassan Adnan, Abdur Rafay, Aryan Kalia

## Research question

Which wells need attention, and which should we repair first to reduce lost oil and improve estimated financial returns?

## Motivation

Repair crews have limited time. A well that produces more oil can lose more production for every day it waits, while a well with low output and expensive maintenance may need an economic review. We combined production, maintenance history and economic benchmarks to make those decisions more transparent.

## Five-minute summary and slides

- **Video:** [Watch the five-minute project summary](https://drive.google.com/file/d/1cDwEO8R8mojYMmqibw1-XyB8p7cVnpJQ/view?usp=sharing)
- [Presentation slides (PDF)](presentation/slides.pdf)
- [Speaking script](presentation/speaker_script.txt)
- [Databricks notebook](code/Databricks_Calculations.ipynb)

## Data and scope

The organizers supplied three datasets:

| Dataset | Rows | Use |
|---|---:|---|
| daily_production | 5,075,590 | Production trends, water, operating hours and downtime |
| well_jobs | 15,610 | Repair dates, prediction targets and field cost estimates |
| production_failures | 2,873 | Failure causes and context |

The daily production history covers 2,905 wells across 2021-2026. The annual economic cohort contains 1,056 wells with positive oil production under the notebook filters, from **28 October 2025 to 27 October 2026**. Well ID (`uwi`) connects the records. The economic cohort, prediction panel and scheduling history are different analysis populations. The repair model learns from jobs, rather than using failure records as its target.

Raw competition data and well-level result exports are omitted from this repository. Authorized users need their own competition data to reproduce the calculations.

## Summary of findings

### Economic review

The main scenario uses a constant **C$65.98/oil barrel** margin before separately deducted costs, **C$5,500/well-month** in allocated remaining operating costs, and each well's repair estimates.

- **162 of 1,056 wells** have negative estimated annual contribution.
- This group produces **1.88% of the cohort's oil**, with combined negative estimated contribution of **C$10.13M**.
- The nine +/-20% margin/cost scenarios flag **125-218 wells**; **125 wells remain negative in every case**.
- Keeping C$5,000/month instead flags **156 wells**. Using quarter-specific margins with C$5,500/month flags **159 wells**.

These are scenario estimates and review candidates. Negative allocated contribution does not establish avoidable loss, closure savings or an abandonment recommendation.

### Repair ranking

The original team's model results report **AUC 0.726**. In the highest-scoring 10% of test well-days, **38.3%** have a repair job starting within 30 days, compared with **14.6%** overall. Five held-out-well splits produce AUC **0.701-0.758**. The target includes planned work and follow-up maintenance, so this is repair-activity ranking, not physical failure diagnosis.

### Historical repair order

The historical 2022-2026 assignment simulations reduce estimated waiting-related oil loss by **14.5% with a seven-day limit** and **17.3% with a fourteen-day limit**. The limit is extra delay after each well's actual historical repair date, not total time since breakdown.

The presentation's rounded historical loss estimates are 526k barrels for actual order, 450k for seven days, 435k for fourteen days, and 396k uncapped. These imply approximately **76k-91k barrels potentially preserved with delay limits**, or 130k uncapped, over the historical period. These are simulated reductions in exposure, not annual or realized savings.

## Methods and validation

**Economics:** PySpark aggregates annual oil volumes and eligible Rod Truck/Workover costs per well. Positive `field_est_total` values are retained. Sixteen unavailable/nonpositive estimates use the selected cohort's positive category medians: rod C$4,811 and workover C$36,813.255. Total estimated repair cost is C$43.82M. The code checks cohort size and required cost coverage, runs the financial scenarios, and creates economic tiers.

**Prediction:** SQL/Spark rolling windows create production, water-cut, uptime and repair-history features. A scikit-learn `HistGradientBoostingClassifier` ranks repair jobs within 30 days. Training dates are before September 2024; testing begins January 2025, with labels through September 27, 2026. Validation includes chronological testing, comparison with simple repair-history rankings, five entire-well holdouts and score-band calibration checks. Retrospective score thresholds and repeated well-day observations limit interpretation as a prospective alerting system.

**Scheduling:** Consecutive qualifying downtime records form episodes, matched to repair starts. Prior operating-day oil rate times waiting days estimates oil exposure. SciPy's `linear_sum_assignment` reassigns observed historical repair slots within area and job type. Repairs cannot precede breakdowns. Capped assignments enforce the additional-delay constraint. The simulation knows future slots and omits travel, job duration and parts availability.

The updated financial arithmetic was independently recalculated. The notebook retains the team's model and scheduling methods and must be run in Databricks to regenerate all outputs; it is distributed with saved outputs cleared.

## External resources and financial assumptions

We used Strathcona's **Lloydminster Conventional** segment results, with printed page numbers:

- [Q4 2025 MD&A, p.22](https://www.strathconaresources.com/wp-content/uploads/2024/05/Strathcona-Managements-Discussion-and-Analysis-Q4-2025-Executed.pdf)
- [Q1 2026 MD&A, p.18](https://www.strathconaresources.com/wp-content/uploads/2026/05/Strathcona-Managements-Discussion-and-Analysis-Q1-2026-Executed.pdf)
- [Q2 2026 MD&A, p.21](https://www.strathconaresources.com/wp-content/uploads/2026/08/Strathcona-Managements-Discussion-and-Analysis-Q2-2026-Executed.pdf)
- [Q3 2026 MD&A, p.21](https://www.strathconaresources.com/wp-content/uploads/2026/11/Strathcona-Managements-Discussion-and-Analysis-Q3-2026-Final-Executed-Version.pdf)

Margin = sales net of blending minus royalties, transport and energy. Quarterly margins are 47.32, 59.05, 76.96 and 73.24 C$/boe. Weighting them by the cohort's production gives C$65.9761, rounded to C$65.98. October 1-27, 2026 uses Q3 2026 as a proxy.

Quarterly non-energy benchmarks applied to the cohort give about C$113.43M. Subtracting C$43.82M in repair estimates and dividing by 1,056 x 12 gives C$5,493.52/well-month, rounded to C$5,500.

This assumes repair estimates belong within reported operating expenses and allocates the remainder equally. Workover capitalization or mismatched accounting scope can invalidate that allocation. The filings use sales boe, while the model applies segment averages approximately to produced oil barrels. Full-year overhead is charged to each eligible well, including partial-year producers. The +/-20% stress range is a modeling choice, not a forecast or confidence interval. Public quarterly results are retrospective information.

Software: [PySpark](https://spark.apache.org/docs/latest/api/python/), [scikit-learn gradient boosting](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.HistGradientBoostingClassifier.html), [SciPy assignment solver](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.linear_sum_assignment.html), pandas, NumPy and Matplotlib. Computation runs in Databricks. Canary was not used in this implementation.


## AI disclosure

AI assistance, including OpenAI Codex, contributed to notebook edits and debugging, financial-source research, economic recalculations. Assistance in beautifying README.
