# Testing for Carbon Leakage: The Kyoto Protocol, Consumption-Based Emissions and the EKC

Bachelor's thesis project, B.Sc. Data Science, AI and Digital Business, GISMA University of Applied Sciences, Berlin (2026).

This repository contains the data pipeline, exploratory analysis and econometric estimation for a thesis that asks whether binding climate commitments under the Kyoto Protocol pushed emissions abroad.

## Research question

> Did binding commitment under the Kyoto Protocol cause Annex B countries to become larger net importers of embodied carbon, relative to countries without binding targets?

Emissions in many rich countries have fallen since 1990 while emissions in emerging economies have risen. One explanation is genuine decarbonisation. Another is carbon leakage, where emission-intensive production moves to countries without climate targets and returns as imports. This project uses the Kyoto Protocol as a natural experiment to test between the two. Annex B countries took on binding targets from 2005, while China, India and other emerging economies did not.

The key measure is the gap between a country's consumption-based and production-based emissions, which equals the net CO₂ embodied in its trade.

## Main findings

| Outcome | Estimate | SE | p-value | 95% CI |
|---|---|---|---|---|
| Gap share (primary) | −0.070 | 0.073 | 0.338 | [−0.214, 0.073] |
| Net transfer per capita, t/person (secondary) | +0.053 | 0.348 | 0.880 | [−0.629, 0.734] |
| Scaled IHS of net transfer (robustness) | +0.002 | 0.192 | 0.992 | [−0.375, 0.379] |

*Two-way fixed effects, country and year effects, standard errors clustered by country. N = 4,029, 119 countries, 1990 to 2023.*

- No detectable Kyoto-induced leakage in national gap measures. The primary estimate has the opposite sign to the leakage hypothesis.
- The result is an imprecise null. The confidence interval admits moderate effects in either direction, with a minimum detectable effect of about 0.20.
- The joint test of pre-treatment event-study coefficients rejects parallel trends for both main outcomes (p = 0.022 and p = 0.033). This is reported openly as the main limitation. Shortening the pre-period makes the test pass but leaves the estimate unchanged to four decimal places, so it is not used as a fix.
- The null holds when the 13 transition economies are excluded, when treatment is dated to 2008, and when large countries are dropped one at a time.
- Descriptively, within-country EKC estimates put the turning point at about $38,700 per capita for production emissions and $41,900 for consumption emissions. Consumption emissions turning at a similar income level is hard to reconcile with relocation as the main driver of rich-country declines.

## Data

| Variable | Source | Coverage |
|---|---|---|
| Territorial fossil CO₂ | [Global Carbon Budget 2025](https://globalcarbonbudget.org/), national emissions workbook | 1850 to 2024 |
| Consumption-based fossil CO₂ | Global Carbon Budget 2025, national emissions workbook | 1990 to 2024 |
| GDP per capita (PPP, constant 2021 int. $) | World Bank, World Development Indicators | 1990 to 2023 |
| Trade openness (% of GDP) | World Bank, World Development Indicators | 1990 to 2023 |
| Population | UN World Population Prospects | 1990 to 2023 |
| Treatment (Annex B status) | UNFCCC, Kyoto Protocol Annex B | |

Both emission series come from the same source so that the gap between them reflects trade and not differences in coverage. EDGAR was not used because it reports all greenhouse gases in CO₂-equivalents, which would not match the fossil CO₂ scope of the consumption series. Emissions are converted from MtC to MtCO₂ by multiplying by 3.664.

The raw data files are not included in this repository. Download them from the sources above and place them in `Data/`.

## Repository structure

```
.
├── Data/
│   ├── National_Fossil_Carbon_Emissions_2025_consumption_emission.xlsx   # GCB workbook (download)
│   ├── <World Bank GDP and trade files>                                   # (download)
│   ├── Population_with_ISO3.csv                                          # UN population (download)
│   └── data_final.csv                                                    # merged panel, created by data_integ.ipynb
├── output/                        # tables and figures written by the notebooks
├── data_integ.ipynb               # 1. data integration
├── thesis_EDA.ipynb               # 2. feature engineering and exploratory analysis
├── thesis_estimation.ipynb        # 3. difference-in-differences estimation
├── requirements.txt
└── README.md
```

## How to reproduce

1. Clone the repository and create a virtual environment.

   ```bash
   git clone https://github.com/Indranil3001/<repo-name>.git
   cd <repo-name>
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```

2. Download the raw data files listed above into `Data/`.

3. Run the notebooks in order, each with **Kernel → Restart & Run All**:

   | Notebook | What it does |
   |---|---|
   | `data_integ.ipynb` | Reshapes the GCB workbook to long format, drops aggregates and bunkers, maps country names to ISO3 codes, merges emissions with GDP, trade and population, codes treatment, and writes `Data/data_final.csv`. |
   | `thesis_EDA.ipynb` | Builds the outcome variables, handles Panama's negative consumption values, checks covariate balance and pre-treatment trends, and estimates fixed-effects EKC curves on both accounting bases. |
   | `thesis_estimation.ipynb` | Estimates the TWFE model, the event study, joint pre-trend tests, and robustness checks (transition economies, 2008 treatment date, 1997 pseudo-treatment, leave-one-out). Writes tables and figures to `output/`. |

Run the notebooks from the project root, because `thesis_estimation.ipynb` reads `Data/data_final.csv` with a relative path.

### Main packages

`pandas`, `numpy`, `statsmodels`, `linearmodels`, `country_converter`, `matplotlib`, `seaborn`, `openpyxl`

## Method in brief

**Panel.** 119 countries from 1990 to 2023 (4,032 country-years). The treated group has 34 Annex B countries and the comparison group has 85 never-treated countries. The United States signed but did not ratify, so it is coded as never treated.

**Outcomes**, fixed in priority order before estimation:

- **Gap share (primary)**: `(C − P) / P`, net transfer as a share of production emissions. It is scale-free, and its denominator is strictly positive.
- **Net transfer per capita (secondary)**: `(C − P) × 10⁶ / N`, in tonnes of CO₂ per person.
- **Scaled IHS (robustness)**: `asinh(transfer / θ)` with θ = 3.755 Mt, the pre-2005 median absolute transfer, following Chen and Roth (2024).

Emission levels are not used as outcomes, because log consumption per capita shows a clear differential pre-trend.

**Estimation.** A two-way fixed-effects difference-in-differences model with treatment `AnnexB × (year ≥ 2005)`, plus an event study with 2004 as the reference year. With a single adoption date for all treated countries, staggered-adoption corrections such as Callaway and Sant'Anna (2021) give the same estimate as the standard design.

## Limitations

- Parallel pre-treatment trends are rejected for both main outcomes.
- Treatment is coded uniformly from 2005 to 2023. This ignores Australia's and Croatia's 2007 ratification, Canada's 2012 withdrawal and the limited participation in the second commitment period.
- Kyoto cannot be separated from the EU Emissions Trading System, which also started in 2005.
- Inner joins exclude economies missing from World Bank data, such as Taiwan.
- National net positions can miss leakage that is concentrated in particular sectors.

## Possible extensions

- Rambachan and Roth (2023) sensitivity analysis for the parallel-trends violation
- Treatment coded by actual ratification dates, with staggered-adoption estimators
- A first-commitment-period-only sample ending in 2012
- Wild cluster bootstrap inference
- Sector-level analysis with multi-regional input-output data (Eora, OECD ICIO)

## Key references

- Aichele, R. and Felbermayr, G. (2015). Kyoto and carbon leakage: An empirical analysis of the carbon content of bilateral trade. *Review of Economics and Statistics*, 97(1), 104-115.
- Friedlingstein, P. et al. (2025). Global Carbon Budget 2025. *Earth System Science Data*.
- Peters, G.P., Minx, J.C., Weber, C.L. and Edenhofer, O. (2011). Growth in emission transfers via international trade from 1990 to 2008. *PNAS*, 108(21), 8903-8908.
- Rambachan, A. and Roth, J. (2023). A more credible approach to parallel trends. *Review of Economic Studies*, 90(5), 2555-2591.

## Author

**Indranil** · [GitHub @Indranil3001](https://github.com/Indranil3001)

## License

<Choose a license, for example MIT for code. Data remain subject to the terms of their original providers.>
