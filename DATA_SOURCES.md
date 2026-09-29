# Dataset feasibility review and download registry

Research checked on 28 September 2026. File sizes below are GitHub repository metadata, not guesses about retained analytical sample sizes. A publicly accessible repository is not necessarily a complete replication package or a blanket data-redistribution license.

## 1. Titz, Putz and Witthaut, Applied Energy 2024

**Paper:** Identifying drivers and mitigators for congestion and redispatch in the German electric power system with explainable AI. Applied Energy 356, 122351. DOI: https://doi.org/10.1016/j.apenergy.2023.122351

Open paper: https://juser.fz-juelich.de/record/1018615/files/1-s2.0-S0306261923017154-main.pdf

**Data used:** hourly observations from May 2019 to January 2023. Redispatch/countertrading records came from Netztransparenz; explanatory day-ahead forecasts and market information came from ENTSO-E. Features included forecast load, renewable and other generation by control area, scheduled cross-border exchanges and electricity prices/differences. The authors explicitly describe an explanatory/ex-post analysis, not a fully operational forecasting design. Do not wrongly claim they simply used realized future generation as if it were available day-ahead.

**Accessibility:** the paper's data-availability statement says data will be made available on request. No exact downloadable processed bundle for this paper was verified. The underlying public sources can be obtained independently; the on-request assembled data may help with replication, but are not evidence of private site-level instruction/workload data.

**Decision:** cite for context and explanatory benchmarks. Do not make successful author permission or credentialed historical downloads a prerequisite for this Kaggle package.

## 2. Dobos and Bichler, WITS 2025

**Paper:** Probabilistic Forecasting of Regional Redispatch Volumes in the German Electricity Market.
https://teodora-dobos.github.io/assets/WITS_Camera_Ready.pdf

**Data used:** 2021-2024 public Redispatch records; the paper reports 264 redispatched units. Geographic coordinates were assembled using Open Power System Data plant references, fuzzy matching and manual resolution. Four regions were constructed using location and redispatch information. ARIMAX/bootstrap and Transformer/quantile approaches were compared.

**Verified related public repositories:**

### Forecasting repository
https://github.com/teodora-dobos/redispatch-forecasting

Commit: `a349d8ed5472629556ad6e9bcc9abdb280bf87b8`.

| File | Repository size | Role here |
|---|---:|---|
| `data/raw/Redispatch_Daten.csv` | 10,039,640 bytes | Primary raw target source |
| `data/redispatch.csv` | 135,232 bytes | Released signed-net series; reconciliation only |
| `data/generation.csv` | 36,112 bytes | Available related series; not substituted for independently downloaded SMARD covariates |
| `data/load.csv` | 39,028 bytes | Same distinction |
| `data/prices.csv` | 44,164 bytes | Same distinction |
| `data/data_parsing.ipynb` | 939,498 bytes | Inspect the original aggregation logic |
| `forecast.ipynb` | 33,033 bytes | Related ARIMAX/LSTM implementation |

The README names a related project using ARIMAX and LSTM. It is **not verified as the complete WITS Transformer/four-cluster replication package**. This protocol therefore makes an extension from a related open data snapshot, not an exact reproduction claim.

The raw parser includes current-related, voltage-related and combined reasons, signs reductions negative, allocates event energy across hourly slots, and aggregates daily. Our main target preserves direction and avoids inferring activity across gaps. Any discrepancies with the released net series are audited, not automatically called paper errors.

Direct pinned target URL:
https://raw.githubusercontent.com/teodora-dobos/redispatch-forecasting/a349d8ed5472629556ad6e9bcc9abdb280bf87b8/data/raw/Redispatch_Daten.csv

### Geographic/dashboard repository
https://github.com/teodora-dobos/redispatch

Commit: `0b318b2a487350423eaed438d811796beda246e1`.

The README documents a 2024 dashboard, specifically a 31 December 2023-31 December 2024 raw source range. Do not conflate this with a full 2021-2024 clustered forecasting replication.

| File | Bytes | Role |
|---|---:|---|
| `redispatch/data/input/redispatch_data.csv` | 3,237,762 | Related raw 2024 data, not the primary four-year target |
| `redispatch/data/input/conventional_power_plants_DE.csv` | 317,384 | Auxiliary plant metadata |
| `redispatch/data/output/geo_redispatch_data.csv` | 3,602,289 | Related geolocated events |
| `redispatch/data/output/geo_redispatched_units.csv` | 17,352 | Auxiliary geolocated unit list |

**Decision:** this is the most practical foundation for the proposed paper. It improves reproducibility and lowers acquisition friction; it does not turn public data into private operational evidence.

## 3. Meisenbacher et al. and AutoWP

Paper: https://arxiv.org/html/2412.00423v1

**Real study data:** two southern German wind turbines, 1.5 MW each; 15-minute power and hub-height wind measurements for 2019-2020, with ECMWF day-ahead weather information. Training used 2019 and testing 2020. The theoretical complete calendar contains 70,176 intervals per turbine, or 140,352 turbine-intervals total, before missingness/cleaning. This is a calendar calculation, not a verified actual retained sample size.

The provider did not explicitly label shutdown events, and the paper notes that these events predate Redispatch 2.0. They are therefore not labelled post-2021 German Redispatch interventions.

**Accessibility:** no public download of that exact two-turbine/ECMWF dataset was verified. Do not treat the public example below as a release of the real study dataset.

AutoWP repository: https://github.com/SMEISEN/AutoWP
Commit: `1153c8a4c2166ac245bcc7ee7675088053b14cbe`.

| Public example | Bytes | Interpretation |
|---|---:|---|
| `data/wind.csv` | 2,103,139 | GEFCom2014 example data according to the README |
| `data/supply__wind_turbine_library.csv` | 105,936 | Wind-turbine power-curve library |

**Decision:** useful for a separate wind-forecasting/curtailment-robustness benchmark. It is not a route to measuring German site-level manual Redispatch work. These files are downloaded only for an auxiliary source audit, not mixed with the primary Redispatch target.

## 4. Recent stochastic and conformal Redispatch studies

**Towards Stochastic (N-1)-Secure Redispatch**, Molodchyk et al., arXiv:2510.23551.
https://arxiv.org/html/2510.23551v1

The reported experiment modifies the MATPOWER IEEE 118-bus system (186 branches, 54 conventional generators, 99 loads) with 24 renewable resources, including seven wind and seventeen solar units, and stochastic uncertainty assumptions. This is numerical power-system experimentation, not a released internal German utility workflow dataset. A complete author replication bundle was not verified in this search.

**Conformal Margins for Electrical Transmission Capacity Under Fixed Balancing Policies**, Balaraman, Srinivasan and Sundar, arXiv:2609.11823, September 2026.
https://arxiv.org/abs/2609.11823

The case study uses RTS-GMLC under specified power-flow and balancing-policy assumptions. Public benchmark network inputs can support simulation research, but the paper is not evidence that private German site-level logs are freely downloadable. Its conformal security claims are conditional on its assumptions; they cannot simply be transferred to this forecasting benchmark.

**Decision:** these studies support a different, physics/optimization-oriented paper. That could be a good project with a power-systems collaborator, but adding those simulated labels to the empirical German forecast panel would not strengthen its validity.

## 5. Official target documentation

https://www.netztransparenz.de/de-de/Systemdienstleistungen/Betriebsfuehrung/Redispatch

Important documented limitations: daily event definitions can include inactive gaps; published events do not cover every foreign-plant intervention; central reporting changed in October 2021 and October 2023; testing categories must not be mixed with the target. The paper must cite the version/date of documentation actually used.

Netztransparenz API portal: https://api-portal.netztransparenz.de/
The API requires registration/client credentials. This package's primary pinned raw download does not require those credentials.

## 6. SMARD download registry

Official source: https://www.smard.de/home/downloadcenter/download-marktdaten/

The public chart endpoints used by the code are:
```
https://www.smard.de/app/chart_data/{filter}/DE/index_day.json
https://www.smard.de/app/chart_data/{filter}/DE/{filter}_DE_day_{timestamp}.json
```

Actual-series identifiers: onshore wind 4067; offshore wind 1225; solar 4068; load 410; gas 4071; hydro 1226. Supplemental forecasts: onshore 123; offshore 3791; solar 125. All HTTP requests, retrieval times, source URLs, byte sizes and SHA-256 digests are recorded. A changing public endpoint can fail; the code does not manufacture a substitute series.

## 7. Additional weather sources

Previous Runs: https://open-meteo.com/en/docs/previous-runs-api
Historical Forecast: https://open-meteo.com/en/docs/historical-forecast-api

Previous Runs supplies fixed lead-time offsets: day1=24 hours, day2=48 hours. Most model archives start in 2024; GFS 2 m temperature extends to March 2021, and the documentation identifies longer histories for JMA models. Variable- and model-specific coverage still needs verification before adoption.

The Historical Forecast API stitches early forecast hours from successive runs. It is not automatically an archive of forecasts available at a chosen day-ahead deadline. The Single Runs API preserves initialization/run structure, but its model-dependent coverage is shorter. Reanalysis is not a real-time forecast.

The optional downloader in this release uses only GFS temperature at a fixed 48-hour lead. It does not claim to implement or verify every model in the archive.

## 8. Kaggle search findings

**Wind Power Generation Data**, `jorgesandoval/wind-power-generation`:
https://www.kaggle.com/datasets/jorgesandoval/wind-power-generation

Four German TSOs, 15-minute wind-generation observations, 23 August 2019-22 September 2020. Useful related data, but no Redispatch labels and no overlap with the primary 2021-2024 cohort. The data card's unit wording should not be accepted without validation. Optional audit only.

**Ensimag IF - Algorithmic trading 2025**:
https://www.kaggle.com/competitions/ensimag-if-2025/data

The listed 11.82 MB data contain German day-ahead wind/solar/load forecasts and imbalance/day-ahead price information. The target is a price spread, not Redispatch. Access requires accepting competition rules. It is not a silent dependency of this study.

**CARE to Compare** is a separate wind-turbine anomaly-detection direction, not a Redispatch dataset. The primary record is https://doi.org/10.5281/zenodo.15846963. A large SCADA benchmark could support a cross-farm fault-detection paper but would change the research question. No oversized SCADA archive is downloaded here.

## 9. Licensing and citation policy

Public availability, permissive code licenses, and permission to redistribute third-party data are separate. The code repository's MIT license does not automatically relicense Netztransparenz, SMARD, GEFCom, plant metadata or weather inputs. Keep initial Kaggle outputs private; retain attribution; inspect the source terms before a public dataset release. Release download scripts and checksums when redistribution rights remain unclear.
