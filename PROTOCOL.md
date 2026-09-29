# Research protocol
## Machine Learning for German Redispatch Forecasting under Data Delays and Distribution Shift

**Version:** 0.1.0, 28 September 2026. **Design:** retrospective, reproducible, probabilistic forecasting benchmark. **Execution:** Kaggle notebooks; no internal utility data. **Primary empirical period:** 2021-2024. This protocol defines planned analyses, not findings.

## 1. Research decision and scope

Proceed with a public-data reliability benchmark, reusing a pinned public author data snapshot and independently downloading official electricity-market covariates. The author-hosted files are themselves derived from public sources. They are not private site telemetry, instruction acknowledgements, or operator workload records.

The main contribution is a transparent combination of (i) directional daily target construction, (ii) explicitly simulated information-age restrictions, (iii) temporally separate model selection and uncertainty calibration, (iv) delayed-feedback calibration, and (v) tests under reporting changes and distribution shift. This is not the first application of ML, quantile forecasting, or conformal inference to Redispatch. A literature update immediately before submission remains necessary.

**Primary question:** How do predictive accuracy and interval reliability change when historical Redispatch observations are only available with a specified minimum age, and can delayed-feedback calibration improve reliability on a later period?

**Not claimed:** automated physical dispatch, reduced measured operator hours, economic congestion-cost savings, all German distribution-level redispatch coverage, a causal regulatory effect, or an audited first-publication-time backtest.

The phrase *publication-time-aware* is intentionally not used in the title. The accessible retrospective datasets do not establish every historical first-release value and timestamp. Forecast issue time, event time, retrieval time and revision time are distinct.

## 2. Datasets and frozen source cohort

### 2.1 Primary target

Repository: `teodora-dobos/redispatch-forecasting`.
Commit: `a349d8ed5472629556ad6e9bcc9abdb280bf87b8`.
File: `data/raw/Redispatch_Daten.csv`.
Git blob SHA-1: `502426295ca78120d66a410f5fe9805d4c453a6f`.
Listed file size: **10,039,640 bytes**. Exact raw/retained row counts are calculated after download; the protocol does not invent those counts.

This is a related public author repository, not a verified complete replication bundle of the WITS 2025 clustered-region Transformer experiment. Its README describes an earlier ARIMAX/LSTM project. We use the raw data, cite the provenance and do not label our results an exact reproduction of WITS.

Read semicolon-delimited text, decimal-comma numbers and local German dates. Expected fields include `GRUND_DER_MASSNAHME`, `BEGINN_DATUM`, `BEGINN_UHRZEIT`, `ENDE_DATUM`, `ENDE_UHRZEIT`, `GESAMTE_ARBEIT_MWH`, `RICHTUNG`, `ANWEISENDER_UENB`, and an asset identifier such as `BETROFFENE_ANLAGE`.

Retain the three explicit electrical redispatch reasons: current-related, voltage-related, and combined current/voltage-related. Exclude countertrading, tests and other reasons. Use positive daily MWh separately for increases and decreases. Regions are **instructing TSOs**: 50Hertz, Amprion, TenneT DE and TransnetBW. These are not eight independent physical sites and are not the WITS paper's K-means clusters.

Do not calculate energy as mean MW multiplied by elapsed event duration. Official publications warn that events may contain gaps. Sum the supplied daily energy when a record lies inside a local day or ends exactly at the following midnight. Quarantine genuinely multi-day or otherwise ambiguous records; mark affected series-days missing. Stop if their known energy exceeds 1% of eligible energy. Do not distribute multi-day energy uniformly over hourly slots.

No-event observations require care. Within the exported date range, an absent directional-series event is treated as zero only on dates with at least one eligible event nationally. Dates with no eligible event anywhere remain missing rather than automatically becoming zero. This is an explicit completeness assumption, not independently verified proof that every zero is correct. Exact duplicate raw rows are counted and retained pending interpretation; the dataset is not silently deduplicated without an event key.

A fixed-cohort sensitivity target includes only assets observed by **31 October 2022**. This is a pre-validation cohort sensitivity analysis, not a complete harmonization of reporting coverage. Original and fixed-cohort outcomes must not be treated as interchangeable.

### 2.2 Official explanatory series

Notebook 00 downloads SMARD daily data for Germany for 2021-2024:

| Name | Chart identifier | Main use |
|---|---:|---|
| Onshore wind generation | 4067 | Lagged observed covariate |
| Offshore wind generation | 1225 | Lagged observed covariate |
| Solar generation | 4068 | Lagged observed covariate |
| Load | 410 | Lagged observed covariate |
| Gas generation | 4071 | Lagged observed covariate |
| Hydropower | 1226 | Lagged observed covariate |
| Onshore wind forecast | 123 | Supplemental, unverified-vintage forecast proxy |
| Offshore wind forecast | 3791 | Supplemental, unverified-vintage forecast proxy |
| Solar forecast | 125 | Supplemental, unverified-vintage forecast proxy |

The client requests `index_day.json`, selects overlapping chunks, parses `series`, converts timestamps to Europe/Berlin dates, detects conflicting duplicate values and checks coverage. Required observed series need at least 98% date coverage; remaining missing covariates are handled by training-only median imputation with missingness indicators. Forecast-proxy failures are explicitly recorded and do not silently become observed future values. Daily generation/load series are energy quantities; target units are MWh, not average MW.

### 2.3 Auxiliary source audits

Download `data/redispatch.csv` from the same author commit for **net-series reconciliation**, not as the main directional target. The related parser signs decreases negative and divides event energy across hourly slots before aggregating. Reconciliation differences are reported for inspection; they are not automatically attributed to an error in the paper.

Download geolocation/plant reference files from `teodora-dobos/redispatch`, commit `0b318b2a487350423eaed438d811796beda246e1`. That repository documents a **2024** dashboard, not the whole WITS input pipeline. Geography is auxiliary and is not used to form future-informed clusters.

Download AutoWP example `data/wind.csv` (2,103,139 bytes) and `data/supply__wind_turbine_library.csv` (105,936 bytes), commit `1153c8a4c2166ac245bcc7ee7675088053b14cbe`. Those public examples are not substituted for the two real German turbines in Meisenbacher et al.'s study and are not added to the Redispatch target panel.

### 2.4 Optional sources, off by default

`optional_weather=True` downloads fixed-48-hour-lead GFS 2 m temperature at four explicitly specified German reference coordinates through the Open-Meteo Previous Runs API. Most models have shorter archives; the GFS temperature history extends to March 2021. These data still do not prove every historical ingestion timestamp. They enter the forecast-proxy sensitivity, not the verified-as-of main design.

`optional_kaggle=True` downloads `jorgesandoval/wind-power-generation` with KaggleHub for a source audit. It contains German TSO wind generation for August 2019-September 2020, not Redispatch labels or the 2021-2024 primary cohort. It is not required to run the study. No credible matching Kaggle Redispatch target dataset was verified in the search used for this package.

ENTSO-E and the official Netztransparenz API can support a future extension, but account/API credentials and historical-vintage availability must be established first. They are not hidden requirements of this pipeline. Do not append 2025/2026 test data after inspecting outcomes and call the choice pre-specified; archive a new protocol first.

## 3. Unit, target and sample size

Forecast at **12:00 Europe/Berlin on D-1** for the total energy on local calendar day D. Predict upward and downward redispatch separately. Calendar days can contain 23, 24 or 25 hours; a known calendar feature encodes day length. There is no hourly target manufactured from event duration.

2021-2024 contains **1,461 calendar days** and, nominally, **11,688 TSO-direction-days** (1,461 x 4 x 2). These are not 11,688 independent samples. A 120-day shared warm-up yields an analysis start of 1 May 2021.

| Period | Dates | Nominal dates | Nominal panel rows |
|---|---|---:|---:|
| Initial training | 1 May 2021-31 December 2022 | 610 | 4,880 |
| Validation | 1 January-30 June 2023 | 181 | 1,448 |
| Calibration | 1 July-30 September 2023 | 92 | 736 |
| Reporting-shift stress test | 1 October-31 December 2023 | 92 | 736 |
| Primary held-out test | 1 January-31 December 2024 | 366 | 2,928 |
| Total after warm-up | 1 May 2021-31 December 2024 | 1,341 | 10,728 |

These are **nominal counts**, before missingness and availability purges. Notebook 01 produces exact retained counts by scenario, split, dates and rows. No power claim is based on inflated site-day or seed counts. There is no arbitrary random 80/20 split.

At target day D, a historical outcome y(s) is usable only when **s <= D-L**. L denotes a simulated minimum calendar-day age relative to the target day, not a measured release latency. The main L is 7; sensitivities use 2, 30 and 60.

For L=7, initial training stops on **25 December 2022** (604 nominal dates; 4,832 rows). Validation selection uses **1 January-24 June 2023** (175 nominal dates; 1,400 rows). Final models are refit once on **1 May 2021-24 June 2023** (785 nominal dates; 6,280 rows). The late-December labels excluded at the initial origin are now mature and may be used in this final fit.

Final models are frozen during calibration and testing. The empirical seasonal baseline and rolling/adaptive calibration update only from labels that have matured. This distinction is intentional and must be reported. At the first test origin, only calibration outcomes through **24 September 2023** are available under L=7: 86 nominal calibration dates, not all 92.

## 4. Features and leakage controls

Use calendar sine/cosine encodings for weekday and annual seasonality, weekend status, local-day length, and pre-defined one-hot TSO-direction identifiers. Do not fit clusters on the full target history.

Target-history features at delay L: lags L, L+1, L+2, L+7, L+14, L+28 and L+56; rolling mean, standard deviation and maximum over 7, 28 and 56 days **after shifting by L**. Windows require at least half their observations, with a minimum of three. Missingness is preserved until training-only imputation.

Observed exogenous features use a two-calendar-day minimum lag and a trailing seven-day mean after that lag. This is also a simulated availability assumption on final-vintage snapshots, not a first-release guarantee. Forecast-proxy values are included only in the explicitly named sensitivity. Same-day realized exogenous values appear only in the `realized_oracle` diagnostic and must never be described as usable day-ahead predictors.

Imputation, scaling, target normalization, hyperparameter selection and residual calibration are fitted within their permitted training windows. Neural sequences use 28 successive feature rows, ending at the target row whose features are already availability-restricted. They never contain an unshifted future target.

Evaluation scales and extreme-day thresholds are frozen from 1 May 2021-31 October 2022: per-series mean energy, lower-bounded by 1 MWh, and per-series 90th percentile. These are available before every validation regime, including L=60. Model-training target scales are estimated separately from the permitted fit data.

## 5. Exact models

All models return quantiles **0.025, 0.05, 0.10, 0.25, 0.50, 0.75, 0.90, 0.95, 0.975**. These define central 50%, 80%, 90% and 95% intervals. Final predictions are nonnegative and noncrossing. Post-hoc rearrangement is reported where used.

### M0: empirical seasonal distribution

Use the most recent 12 available observations with the same weekday for the same directional TSO series. Require four matching weekdays; otherwise use up to the last 84 available observations. Report their empirical quantiles. Availability respects L=7. This baseline is allowed to learn from newly matured labels during testing.

### M1: regularized ARX

A Ridge autoregression with external covariates, training-only median imputation/missingness indicators, and standardization. Tune alpha over **0.1, 1, 10, 100**. Obtain residual quantiles from an internal chronological **60-day** holdout of each permitted fit window, then refit the point model on the complete permitted window. This is ARX-Ridge, not a falsely labelled replication of the papers' ARIMAX specification.

### M2: quantile LightGBM

A pooled model, with a separate quantile-loss regressor for each of nine quantiles. Normalize targets by each series' training mean. Use subsample=0.9, subsample frequency=1, column subsample=0.9, two CPU threads, deterministic/column-wise settings, and the following four configurations:

| Trees | Learning rate | Leaves | Minimum child samples |
|---:|---:|---:|---:|
| 400 | 0.03 | 15 | 40 |
| 800 | 0.03 | 15 | 80 |
| 400 | 0.05 | 31 | 80 |
| 800 | 0.03 | 31 | 40 |

Choose by validation macro normalized WIS. Hyperparameters selected on the main design transfer unchanged to sensitivity analyses. This makes the sensitivities controlled comparisons; it is not exhaustive per-scenario tuning.

### M3: quantile GRU

Two recurrent layers, hidden size **64**, dropout **0.1**, 28-step context; a 64-unit GELU/dropout head. Nine ordered nonnegative quantile outputs use a softplus base and cumulative positive increments.

### M4: small quantile Transformer

Feature projection to **64** dimensions, fixed sinusoidal positional encoding, **2 encoder layers**, **4 attention heads**, feed-forward width **128**, dropout **0.1**, pre-normalization; last-token representation and the same quantile head. This is a controlled compact Transformer baseline, not TFT, PatchTST, or an exhaustive state-of-the-art search.

Neural optimization: AdamW, weight decay **1e-4**, batch size **256**, gradient clipping **1.0**, candidate learning rates **0.001 and 0.0003**, maximum **120 epochs**, validation patience **12**. Choose learning rate by validation normalized WIS and the epoch using validation pinball loss. Refit for the selected epoch count on the final permitted data, without test-set early stopping. CUDA runs use mixed precision; CPU smoke tests do not.

### Seeds and ensembles

Main LightGBM, GRU and Transformer use **42, 17, 123, 2026, 3407**. Average their corresponding quantiles for the main comparison. This is quantile averaging, not a claim that the result is the exact quantile function of a mixture distribution. Also report individual-seed scores.

Sensitivity analyses use **seed 42**. `delay07` and `train_fraction_100` are copies of the main seed-42 predictions, not copies of the five-seed ensemble. This avoids confounding information delay or training fraction with ensemble size.

## 6. Pre-specified experiments

| ID | Analysis | Models and settings |
|---|---|---|
| E1 | Main model benchmark | M0-M4; frozen 2024 test; raw probabilistic outputs |
| E2 | Calibration comparison | All five main models; raw, static, rolling and adaptive adjustment |
| E3 | Information-age sensitivity | LightGBM; L=2, 7, 30, 60; matched seed 42 |
| E4 | Historical-target value | LightGBM with versus without target-history features |
| E5 | Minimum calendar baseline | LightGBM with calendar and series identity only |
| E6 | Forecast-vintage sensitivity | LightGBM with supplemental SMARD forecasts; optional weather proxy |
| E7 | Oracle diagnostic | Add realized target-day exogenous values; explicitly invalid for operational deployment |
| E8 | Reporting-cohort sensitivity | LightGBM on the fixed pre-validation asset cohort; separate target definition |
| E9 | Training-history sensitivity | Most recent 25%, 50% and 100% of eligible fit dates; fixed parameters, seed 42 |
| E10 | Reliability subgroups | TSO, direction, month, extreme versus other days, 2023 Q4 versus 2024 |
| E11 | Selective forecast use | Uncertainty thresholds frozen on pre-test calibration forecasts; descriptive retention versus error |
| E12 | Compute and reproducibility | Fit time, seed variability, environment, source hashes, missingness and label reconciliation |

The primary conclusion must not depend on an optional source whose download failed. Every skipped optional scenario is recorded. Do not silently discard a failed core model or select favorable seeds. The pipeline refuses a final analysis without expected core prediction files.

## 7. Calibration implementation

For each model, series and interval, use nonconformity **max(lower-y, y-upper, 0)**. The zero floor makes this **non-shrinking CQR**: it can widen but not contract the base interval. The finite-sample radius is the ceil((n+1)(1-alpha))-th ordered calibration score; require at least **32** finite scores. Clip physical lower limits at zero. Enforce nested intervals by expanding outer bounds.

**Static:** freeze the available calibration pool at the first test origin. **Rolling:** use the last **90 calendar days** of scores whose labels have matured. **Adaptive:** use that rolling pool and an ACI-style alpha update with **gamma=0.005**, processing prediction errors only when their historical outcomes mature. The adaptive alpha is clipped to [1/(n+1)+epsilon, 0.5] to keep finite ranks.

The adaptive procedure is a deliberately specified, clipped, delayed-feedback variant. Do not attribute the unmodified ACI theorem to it. Serial dependence, drift, zero-inflation and changing reporting mean empirical interval coverage is the outcome of interest; no blanket finite-sample conditional safety guarantee is claimed.

## 8. Outcomes and statistical inference

**Primary outcome:** 2024 daily macro-average normalized weighted interval score (WIS). Compute the mean across all eight series on dates with complete paired outcomes, then average dates. Normalize by the frozen per-series reference mean. WIS combines median error and central-interval width/miss penalties. Lower is better.

Secondary outcomes: unnormalized WIS, quantile loss, MAE, RMSE, empirical 50/80/90/95% coverage, interval width, normalized width, extreme-day reliability, directional/TSO differences, and fitted-model cost. Pair coverage with width: wider intervals can obtain higher coverage without being more useful.

Report retained counts and missingness. Non-complete primary dates are not filled with artificial observations. All-model comparative tests use matched paired dates. Gaps remain gaps in calendar-time block resampling.

**Uncertainty:** paired moving-block bootstrap with **5,000 replicates**, **7 calendar-day blocks** as primary; sensitivity block lengths **14 and 28**. Each block jointly carries all TSO-direction errors; a seed is not a new observation. Use percentile 95% confidence intervals. Two-sided bootstrap p-values come from centered paired daily loss differences with a plus-one correction.

Three confirmatory comparisons on 2024 normalized WIS:

1. Raw LightGBM ensemble minus raw ARX.
2. Rolling-calibrated LightGBM minus static-calibrated LightGBM.
3. Adaptive-calibrated LightGBM minus rolling-calibrated LightGBM.

Apply **Holm correction across those three tests**, family alpha=0.05. The reported effect CIs remain pointwise, not simultaneous. GRU, Transformer and seasonal-versus-LightGBM comparisons form a separate exploratory family. Block-length sensitivity is not a chance to select the most favorable p-value.

An approximate secondary Diebold-Mariano comparison uses a Bartlett/Newey-West long-run variance with lag **7** and a two-sided normal approximation. Omit it when paired dates are nonconsecutive; do not silently reinterpret calendar gaps as adjacent days. It is corroborative, not the primary inferential basis.

Do not use an ordinary independent-samples t-test on all panel rows or compare five seed scores as if they were independent datasets. No causal interrupted-time-series effect of regulation is claimed from the October 2023 split: many reporting, system, fuel and weather conditions co-vary.

## 9. Planned tables and figures

Tables: exact dataset/split counts; input availability ledger; hyperparameters and seed manifests; main and per-series accuracy; calibration/width; tails/months; primary and secondary paired tests; delay/cohort/history/forecast-proxy sensitivities; runtime; optional-source status; author-net reconciliation.

Eleven generated figures, each exported separately as **300-dpi PNG, vector PDF and SVG**:

1. Directional daily target history with reporting-date markers.
2. Chronological design and split timeline.
3. Main 2024 model WIS with day-block intervals.
4. Nominal versus empirical interval coverage.
5. Rolling 90-day coverage.
6. Matched-seed error versus simulated data age.
7. Extreme versus other-day interval coverage.
8. A pre-specified forecast example: TenneT downward, January-March 2024.
9. Training-history fraction sensitivity.
10. Selective-use retention versus error, with frozen thresholds.
11. Median final-fit compute cost by model.

A generated manuscript Markdown/LaTeX scaffold inserts run-specific numeric facts and tables, but leaves scientific interpretation to the authors. It is not automatically a submission-ready paper. Smoke results are visibly labelled synthetic and cannot be used as evidence.

## 10. Compute and storage

Primary raw Redispatch snapshot is about 10 MB; AutoWP examples about 2.2 MB; selected daily time series and feature matrices are small. Model checkpoints dominate storage. Exact footprint is logged; do not pretend it is known before the run.

The downloader enforces a **2 GB raw-download budget**, a **512 MB per-file maximum**, and a **18 GB working-directory ceiling**, leaving room below the user's 20 GB target. It never downloads continental weather grids. Cumulative stage archives avoid requiring the user to create or upload local data files.

CPU suffices for downloads, audits, ARX, LightGBM, statistics and figures. T4 GPUs are optional for the small neural models. With T4 x2, two independent model/seed jobs run in separate subprocesses, one device each. VRAM is **not pooled** and a single model does not require 32 GB. Each network is small enough to be expected to fit well within one T4; actual use must be observed in Kaggle. GPU parallelism was implemented but cannot be inferred as validated by CPU tests.

Training time depends on Kaggle contention, early stopping and CPU resources. Do not reserve an asserted exact number of hours from an unrun full experiment. The smoke profile, tuning logs and per-job runtime files provide measurements before committing more GPU quota. Checkpoint at every neural epoch and after each completed CPU model/seed. Do not reinstall PyTorch and replace Kaggle's CUDA build.

## 11. Execution, versioning and publication

1. Create a Git repository for this code, not for bulky datasets. Record your own authorship and institution truthfully.
2. Import Notebook 00 into Kaggle, turn Internet on, choose CPU and run/save all cells. It initializes a local source Git snapshot and downloads the datasets itself.
3. Import Notebook 01; attach the saved output of Notebook 00. Inspect `audit.json`, exact counts, quarantine and source status. The code rejects inadequate coverage and schema inconsistencies rather than fabricating replacements.
4. Run Notebook 02 on CPU, using only the previous saved output.
5. Run Notebook 03 with T4 x2 (or one GPU/CPU), using only Notebook 02's saved output. Independent jobs and epoch checkpoints provide resumability.
6. Run Notebook 04 on CPU using Notebook 03's saved output. It checks that expected core seeds/models completed, calibrates, scores and performs the statistical tests.
7. Run Notebook 05 on CPU using Notebook 04's saved output. Review every figure and the generated result tables. Export the cumulative manuscript bundle; no local dataset download was required.
8. Write the introduction and discussion around the measured results, including negative findings. Verify the literature, source licenses, source attribution and all captions. Do not claim labor savings, causal regulatory effects or safe control.
9. Archive the code version, protocol lock, data hashes, public acquisition instructions and permitted artifacts. A code license does not grant redistribution rights for third-party data. Keep Kaggle datasets/notebooks private until those rights have been checked.
10. Seek a power-systems co-author or independent methods review before submission. Journal fit depends on actual results and contribution, not a promised acceptance tier.

**Go/no-go:** proceed only if raw-date/coverage/energy audits pass, future-perturbation tests pass, core predictions are complete, and the result includes a meaningful reliability or reproducibility finding. If advanced models do not improve upon ARX/seasonality, report that result rather than retuning against the held-out test. If the apparent contribution vanishes under proper timing, that may itself support a rigorously framed negative-result paper.

## 12. Software validation versus scientific validation

The delivered code has been syntax-checked, unit-tested and exercised end-to-end on explicitly synthetic data, including CPU execution of both neural architectures and chart/report generation. Full real-data downloads, full hyperparameter/seed runs, and actual dual-T4 execution are **not completed by those tests**. Live endpoints can change. Notebook 00 records exact download failures and checksums; network failures are not replaced by synthetic research data.

See DATA_SOURCES.md for the dataset feasibility review, README.md for the shortest Kaggle run path, and TESTING.md for the exact validation status of this release.
