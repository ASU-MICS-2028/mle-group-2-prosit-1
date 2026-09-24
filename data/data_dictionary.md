# data_dictionary.md  (student "datasheet" - ties to LO7)

The package uses **real data**: open-access sources (WHO/GHO, DHS subnational
indicators, Ghana Health Service routine surveillance, geoBoundaries) plus a
de-identified extract of the **licensed Ghana 2022 Demographic and Health Survey
(DHS)** household microdata (`data/ghana_mis_sample.csv`), used with the
instructor's DHS data-use authorization.

**Region scheme:** Ghana reorganised from 10 to 16 administrative regions
(2018/2022). The 2022 DHS uses the **16-region** coding, and all course datasets
are aligned to it via `hv024` (1-16).

## data/ghana_mis_sample.csv  (Ghana 2022 DHS household extract - de-identified)
One row per household (17,933 households, 618 clusters, 16 regions). Extracted
from the licensed `GHHR8CFL.DTA` (household recode) by
`data_prep/build_mis_sample.py`. No identifying variables are retained.
Licensed - treat as confidential.

| column | type | source (DHS code) | notes |
|---|---|---|---|
| cluster | int | hv001 | survey cluster - the sampling-unit anchor (do NOT average across clusters as i.i.d.) |
| household | int | hv002 | household number within cluster |
| region | int | hv024 | **2022 16-region code** (1=Western ... 16=Upper West) |
| residence | str | hv025 | 'urban' / 'rural' (1/2 recoded) |
| sample_weight | float | hv005 / 1e6 | sampling weight factor (design-effect / weighted-estimate lesson) |
| wealth_index | int | hv270 | combined wealth index (1 poorest ... 5 richest) |
| has_net | int | hv227 | household has mosquito bed net for sleeping (1/0) |
| num_nets | int | hml1 | number of mosquito bed nets |
| children_under_net_last_night | int | hml2 | children under a net the previous night |
| eligible_children | int | hv035 | eligible children for hemoglobin / malaria testing |

**Real patterns you will find (verify in your notebook):**
- Unweighted vs weighted net ownership differ - the survey-design effect.
- A strong regional and urban/rural gap in net ownership (equity material for LO5).
- Many households own **zero** nets (5,208 of 17,933) - count-data material for Theme A.
- Per-cluster net counts are **over-dispersed** (variance/mean well above 1) - Poisson vs Negative-Binomial material.

## data/ghana_region_malaria.csv  (DHS subnational indicators, all regions)
One row per (region x survey), covering **every** Ghana region across six surveys
(2003, 2008, 2014, 2016, 2019, 2022). Built by `data_prep/build_region_dataset.py`
from the DHS subnational tables. This is the file that removes the 3-region
limitation of the routine data: it has region-level prevalence and ITN coverage
for the whole country, **with published confidence intervals**.

| column | type | source | notes |
|---|---|---|---|
| region_name | str | DHS CharacteristicLabel | cleaned region name (pre/post-2022 variants labelled) |
| dhs_region_code | int | DHS CharacteristicId - 408000 | DHS API's own code (stable within DHS) |
| hv024_16region | int | name -> hv024 | **2022 16-region code** (join key to `ghana_mis_sample.csv`); empty for pre-split aggregates |
| survey_year / survey_id | int/str | DHS | e.g. 2022 / GH2022DHS |
| rdt_prevalence_pct (+_ci_low/_high) | float | DHS | malaria prevalence by RDT, children 6-59 months |
| microscopy_prevalence_pct (+_ci_low/_high) | float | DHS | prevalence by microscopy |
| n_children_tested_rdt_weighted / _unweighted | int | DHS | denominators (bootstrap material) |
| net_ownership_pct (+_ci_low/_high) | float | DHS | households with >=1 ITN |
| u5_itn_use_pct (+_ci_low/_high) | float | DHS | children under 5 who slept under an ITN |
| mean_itns_per_hh | float | DHS | mean ITNs per household |
| n_households | int | DHS | households in the denominator |

**Why it matters:** the 2022 survey gives 16 regions; older surveys give the
then-current 10 regions. The `_ci_low`/`_ci_high` columns are the *official*
design-based intervals, so you can compare your own bootstrap intervals (Task A4)
against them - a rare chance to check your method against the reference.

## data/ghana_district_cases.csv  (built by data_prep/build_district_dataset.py)
One row per district (built from the real northern-Ghana routine surveillance,
with net coverage joined from the licensed Ghana 2022 DHS extract).

| column | type | source | notes |
|---|---|---|---|
| district | str | Ghana district name | from routine surveillance workbook |
| region_name | str | workbook sheet | 'Northern Region' / 'Upper East Region' / 'Upper West Region' |
| region_code | int | 2022 DHS `hv024` | **12** = Northern, **15** = Upper East, **16** = Upper West (join key to the survey) |
| year_start / year_end | int | raw | first / last year with reported months |
| months_reported | int | raw | number of months with data (watch: gap analysis, B1) |
| suspected_cases | int | Ghana Health Service routine data | cumulative suspected over period |
| tested_cases | int | Ghana Health Service routine data | cumulative tested |
| positive_cases | int | Ghana Health Service routine data | cumulative positive (RDT/microscopy) |
| mean_population | float | routine surveillance population column | mean reported population |
| positive_per_100k | float | derived | positive_cases / mean_population * 100000 |
| net_coverage_pct | float | Ghana 2022 DHS (weighted `has_net`) | **region-level** weighted household net ownership; varies by region |

## data/raw/select-malaria-indicators_national_gha.csv  (DHS Program / MIS)
National malaria indicators across Ghana surveys (2003-2022), 29 cols:
`Indicator`, `Value`, `SurveyId` (e.g. GH2022DHS, GH2019MIS, GH2016MIS),
`SurveyYear`, `CILow`, `CIHigh`, `Denominator*`.  Key indicators include
"net access", "slept under net", "RDT prevalence".  Source is perfect for the
national time series and the 2022 survey underpinning the Prosit.

## data/raw/malaria_indicators_gha.csv  (WHO Global Health Observatory)
WHO GHO Ghana malaria indicators, 2000-2024 (140 rows x 17 cols):
`GHO (CODE)`, `YEAR (DISPLAY)`, `Numeric`, `Value`, `Low`, `High`.
Indicators: MALARIA_EST_INCIDENCE (per 1000), MALARIA_TOTAL_CASES,
MALARIA_CONF_CASES, MALARIA_INDIG, MALARIA_PF_INDIG, MALARIA_SUSPECTS,
MALARIA_EST_MORTALITY, MALARIA_RDT_POS, MALARIA_PRES_CASES,
MALARIA_MICR_TEST, MALARIA_MICR_POS.  Use as external benchmark / recency check.

## data/raw/northern-ghana-districts-routine-data-2014-17.xlsx
Ghana Health Service routine monthly surveillance, 3 regions (Northern 26 ,
Upper East 13, Upper West 11 districts), 2014-2017:
`Districts, Year, Month, Malaria Susp, Malaria Test, Malaria Positive,
Total Population, IRS Status, SMC Status`.  Real counts = the count-data
response for the Poisson / Negative-Binomial tasks.

## data/raw/gha_boundaries/
Ghana administrative boundaries (GeoJSON): `gha_admin0/1/2`, edge-matched `_em`,
`gha_adminpoints`, `gha_adminlines`, `gha_admincapitals`.  From HDX / COD
(geoBoundaries-indicative).  Used for the geospatial EDA map (B1).

## data/raw/ (DHS subnational + boundary intake)
- `malaria-parasitemia_subnational_gha.csv` - region-level RDT/microscopy
  prevalence, 2014/2016/2019/2022 (source for `ghana_region_malaria.csv`).
- `insecticide-treated-nets_subnational_gha.csv` - region-level ITN ownership/use,
  2003-2022.
- `dhs-quickstats_subnational_gha.csv` - extra region-level indicators.
- `gha_boundaries/` - Ghana administrative boundaries (GeoJSON): `gha_admin0/1/2`,
  edge-matched `_em`, points/lines/capitals. From HDX / COD. Used for the EDA map (B1).

## Known caveats
- **Coverage differs by dataset.** `ghana_region_malaria.csv` covers **all 16
  regions** (survey-based, 2003-2022). `ghana_district_cases.csv` covers only the
  **3 northern regions** (Northern, Upper East, Upper West) because district-level
  monthly routine case counts are not published openly for the other regions.
- **Region reorganisation.** Pre-2022 rows use the old 10-region scheme; the
  split regions (Western, Volta, Brong-Ahafo, Northern) have no single 16-region
  code, so `hv024_16region` is left empty for those aggregates. Only 2022 rows
  carry the full 16-region coding.
- `net_coverage_pct` is the **region-level weighted** household net-ownership from
  the Ghana 2022 DHS (Northern 67.7%, Upper East 79.6%, Upper West 69.8%). It
  varies by region, not by district, and the survey (2022) post-dates the routine
  surveillance window (2014-17) - a temporal-mismatch caveat worth flagging in
  your leakage audit (B4).
- `positive_per_100k` is cumulative positives over the whole reporting period
  divided by mean population, so values are scaled up versus a single-year
  prevalence; treat it as a *relative* burden indicator across districts.
- Values in the district file are aggregated routine counts; treat as
  surveillance, not survey-estimated.
- Licensed DHS microdata was used under the instructor's data-use authorization to
  build `data/ghana_mis_sample.csv`. The raw DHS archive stays in `data/raw/` and
  must **not** be redistributed or uploaded to generative-AI tools.