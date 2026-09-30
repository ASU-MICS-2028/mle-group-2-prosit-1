# Limited Nets, Five Districts: Data-Driven Allocation of Insecticide-Treated Nets in Northern Ghana

Group 2
ICS553 Machine Learning Essentials, Ashesi University, 2026

---

## Abstract

A limited consignment of 100,000 insecticide-treated nets (ITNs) must be allocated across Northern Ghana. We combine routine clinic records, a national household survey and regional survey indicators to decide where the nets should go and to measure how uncertain the estimates are. District case counts are over-dispersed (variance-to-mean ratio 77,183): a Poisson model places 1 of 50 districts inside its 95% range, while a Negative-Binomial model (α = 0.342) places 47. Ignoring the survey's village clustering makes the confidence interval about 22% too narrow. Preprocessing is fitted on training districts only, and a leakage audit quantifies the effect of pooled preprocessing. We rank districts by confirmed cases among people without a net and recommend allocating the nets to Wa, Bolgatanga, Garu-Tempane, Jirapa and Bawku, which raises coverage in each above 90%, past the WHO 80% target.

---

## Contents

1. [Research question](#1-research-question)
2. [Methods](#2-methods)
3. [Results](#3-results)
4. [Reproducing the results](#4-reproducing-the-results)
5. [Repository structure](#5-repository-structure)
6. [Limitations](#6-limitations)
7. [Use of AI tools](#7-use-of-ai-tools)
8. [Citation](#8-citation)

---

## 1. Research question

Where should Ghana's limited supply of ITNs go, and how confident can we be in the district-level malaria estimates behind that decision? We address four points set by the case brief: the estimate used, its uncertainty, how sampling bias and data leakage were avoided, and which districts are prioritised and why.

---

## 2. Methods

**Distributions (Theme A).** We examine the response `positive_cases` for skew and over-dispersion, fit an intercept-only Poisson model by maximum likelihood (λ = sample mean), and compare it with a Negative-Binomial (NB2) model fitted in `statsmodels`, using log-likelihood, AIC, binned probability plots, Q-Q plots and each model's 95% range.

**Uncertainty (Theme A).** National net ownership is estimated with and without survey weights. For rural Upper East, we compare a simple bootstrap (resampling households) with a cluster bootstrap (resampling villages), each with 2,000 replicates, to show the effect of the two-stage cluster design.

**Preprocessing and leakage (Theme B).** We map data gaps and testing rates, check for missing values and duplicates, and split the 50 districts 75/25, stratified by region. Imputation, scaling and one-hot encoding sit inside a scikit-learn `Pipeline` fitted on the training districts only. A leakage audit compares this with preprocessing fitted on pooled data and discusses temporal and ecological leakage.

**Allocation.** Each district's priority score is

```
priority = (cases per 100,000 / 1,000) × unprotected population
         = 100 × confirmed cases × (1 − regional net coverage)
```

that is, confirmed cases among people without a net. The 100,000 nets are split across the five highest-ranked districts in proportion to their unprotected population.

---

## 3. Results

**Table 1. Poisson vs. Negative-Binomial fit to district case counts (n = 50).**

| Model | Parameter | Log-likelihood | AIC | 95% range | Districts inside |
|---|---|---|---|---|---|
| Poisson | λ = 195,640 | −1,693,731 | 3,387,464 | 194,773 to 196,507 | 1 of 50 |
| Negative-Binomial | α = 0.342 | −647 | 1,298 | 39,270 to 475,108 | 47 of 50 |

**Table 2. Survey uncertainty.**

| Estimate | Value |
|---|---|
| National net ownership, unweighted | 70.96% |
| National net ownership, survey-weighted | 66.77% |
| Rural Upper East, simple bootstrap 95% CI | 79.70% to 85.60% (width 5.89 pts) |
| Rural Upper East, cluster bootstrap 95% CI | 78.85% to 86.44% (width 7.59 pts) |
| Cluster / simple interval width | 1.29× (simple interval about 22% too narrow) |

**Table 3. Leakage audit.** Fitting the imputer on pooled data shifts the imputed median population by 8,984 people (9.13%). Test RMSE (41,666) and MAE (35,009) are unchanged, because the district data contains no missing values; the leak affects the fitted statistics rather than the score.

**Table 4. Recommended allocation of 100,000 nets.**

| Rank | District | Region | Nets | Coverage before → after |
|---|---|---|---|---|
| 1 | Wa | Upper West | 24,463 | 69.8% → 90.2% |
| 2 | Bolgatanga | Upper East | 20,307 | 79.6% → 93.4% |
| 3 | Garu-Tempane | Upper East | 19,952 | 79.6% → 93.4% |
| 4 | Jirapa | Upper West | 20,152 | 69.8% → 90.2% |
| 5 | Bawku | Upper East | 15,125 | 79.6% → 93.4% |

The tranche closes about 68% of the combined unprotected gap in these five districts (100,000 of about 147,600 people). The rule favours preventing the most cases now: Nabdam has the highest rate in the north (713,356 per 100,000) but a small population, so it falls from 11th to 20th and is recommended for the next tranche.

---

## 4. Reproducing the results

### 4.1 Environment

Python 3.11 is required. Using conda:

```bash
conda create -n ghana-itn python=3.11 -y
conda activate ghana-itn
pip install -r requirements.txt
```

or using venv:

```bash
python3.11 -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 4.2 Running

The notebook reads the course data files from a `data/` folder next to it. The data is not included in this repository.

Open `ghana_itn_allocation.ipynb` in Jupyter or VS Code, select the environment above as the kernel, and run all cells. Random seeds are fixed, so the outputs match the saved ones.

---

## 5. Repository structure

```
.
├── ghana_itn_allocation.ipynb   Analysis notebook, with saved outputs
├── requirements.txt             Pinned Python dependencies
├── README.md                    This file
└── data/                        Course data (not included)
```

---

## 6. Limitations

- **Temporal mismatch.** Net coverage is from the 2022 survey, while case counts cover 2014-2017.
- **Regional coverage only.** All districts in a region share one coverage value, so within-region differences in net ownership are not captured.
- **Counts are clinic visits, not people.** Suspected cases exceed the population in all 50 districts, and confirmed cases do in 39; figures are used to rank districts, not as prevalence.
- **Uneven testing.** Testing rates range from 41.8% (Karaga) to above 95%, so some districts are likely under-counted.
- **Geographic coverage.** 11 of Ghana's 16 regions have no district-level case data, so the ranking applies to the north only.
- **Point estimates.** The ranking uses point estimates; survey uncertainty is reported but not built into the allocation rule.

---

## 7. Use of AI tools

AI tools were used in line with the ICS553 course policy. Google Antigravity assisted with scikit-learn and statsmodels syntax, matplotlib layout and draft LaTeX derivations. Claude Code (Anthropic) assisted with environment setup, district boundary matching, the distribution map, notebook tidying and consistency checks between the notebook and the presentation. All code was run and checked by the team, every reported figure is traced to a notebook output, and interpretations and the final recommendation are the team's own. No MIS household records were uploaded to any AI tool. The full declaration is at the end of the notebook.

---

## 8. Citation

```bibtex
@misc{group2_2026_itn,
  title        = {Limited Nets, Five Districts: Data-Driven Allocation of
                  Insecticide-Treated Nets in Northern Ghana},
  author       = {{Group 2}},
  year         = {2026},
  howpublished = {ICS553 Machine Learning Essentials, Ashesi University},
  url          = {https://github.com/ASU-MICS-2028/mle-group-2-prosit-1}
}
```
