<img src=".github/banner.svg" width="100%" alt="Carbon-Footprint-LCA: lifecycle carbon of electricity, applied to China's grid">

[![tests](https://github.com/JoooDev-TPP/Carbon-Footprint-LCA/actions/workflows/test.yml/badge.svg)](https://github.com/JoooDev-TPP/Carbon-Footprint-LCA/actions/workflows/test.yml) [![License: MIT](https://img.shields.io/github/license/JoooDev-TPP/Carbon-Footprint-LCA)](LICENSE)
<!-- TODO: DOI badge once a release is archived on Zenodo -->

**The same 100 MW of utility solar avoids 65% more CO2 in the North grid than in the South.**

Lifecycle emissions of eleven electricity technologies from IPCC AR5 Table A.III.2, applied to China's actual generation mix and to the official 2023 regional grid factors. The question is where new renewable capacity avoids the most, and how long it takes to pay back its own footprint.

![Emissions avoided by 100 MW](outputs/avoided_emissions.png)

## Quick start

```bash
git clone https://github.com/JoooDev-TPP/Carbon-Footprint-LCA
cd Carbon-Footprint-LCA
pip install -r requirements.txt
make test
make notebook
```

`make test` pins every input to its source. `make notebook` reruns the analysis and regenerates every figure from the committed data.

## Findings

- Coal emits 820 gCO2eq/kWh over its lifecycle: 75 times onshore wind, 68 times nuclear and 17 times utility-scale solar (IPCC AR5 medians).
- Coal's share of China's generation fell from 69.6% in 2015 to 58.2% in 2024. Lifecycle-weighted grid intensity fell from 596 to 509 gCO2eq/kWh, or 14.6%.
- The South grid, which includes the Greater Bay Area, has the lowest official factor in China: 404 gCO2/kWh in 2023, 36% below the North grid's 636.
- Location matters as much as technology. 100 MW of utility solar avoids 49,925 t CO2 a year net in the South and 82,428 t in the North, 65% more for the same panels.
- In the South grid, carbon payback is 8 months for onshore wind and 3.0 years for utility solar (1.1 to 11 years across AR5's range).

| | |
|---|---|
| ![Lifecycle emissions by technology](outputs/lifecycle_comparison.png) | ![China's generation mix and lifecycle intensity](outputs/grid_decarbonisation.png) |
| ![Official 2023 grid factors by region](outputs/regional_grid_ef.png) | ![Carbon payback](outputs/carbon_payback.png) |

The full analysis is in [`notebooks/analysis.ipynb`](notebooks/analysis.ipynb). Its conclusions are generated from the tables.

## Data

| Input | Source | File |
|---|---|---|
| Lifecycle emissions, min / median / max, gCO2eq/kWh | IPCC AR5 WGIII Annex III (Schlömer et al., 2014), Table A.III.2 | `data/ar5_lifecycle_emissions.csv` |
| Plant lifetimes | same annex, Table A.III.1 | same file |
| China's generation shares, 2015–2024 | [Our World in Data energy dataset](https://github.com/owid/energy-data), from Ember and the Energy Institute (CC BY 4.0) | `data/china_generation_mix_2015_2024.csv` |
| Regional grid emission factors, 2023 | 生态环境部、国家统计局，关于发布2023年电力二氧化碳排放因子的公告 (2025-12-31). Ministry of Ecology and Environment and National Bureau of Statistics, 2023 electricity CO2 emission factors | `data/mee_2023_grid_factors.csv` |

The AR5 values were transcribed from the annex PDF, whose SHA-256 is pinned in `scripts/fetch_sources.py`. The grid factors carry the notice's URL and the SHA-256 of its PDF. The PDFs are not redistributed. `make sources` re-downloads the AR5 annex and the OWID dataset; this is optional.

## Method

- The figures are lifecycle, not operational. AR5 includes mining, fuel supply and methane leakage, so the lifecycle-weighted grid intensity runs higher than the official operational factor (0.5306 kg/kWh nationally in 2023). The two measure different things and are not compared directly.
- Shares sum to 100%. Every source of generation is kept. Oil (0.6–0.9% of the mix) has no row in Table A.III.2, so it is excluded from the weighting, and the covered share, 99.1–99.4% in every year, is reported alongside.
- Wind uses the onshore value (11; offshore is 12), solar the utility-scale value (48; rooftop is 41, so this errs slightly high), and bioenergy the dedicated-biomass value.
- Avoided emissions use the official average grid factor for the region and the technology's own lifecycle median. The capacity factors (solar 0.16, onshore wind 0.25, offshore 0.35) are assumptions, each inside AR5's full-load-hour range for the technology. Tonnages scale with them; the ratio between regions does not depend on them.
- Payback is lifecycle intensity × plant lifetime ÷ displaced grid factor. Capacity factor cancels out. Lifetimes are AR5's: 25 years for solar and wind.

## Limits

- The grid factor is an average. New capacity usually displaces fossil generation at the margin, which emits more than the grid average, so using the average understates what a new plant avoids, in every region alike.
- AR5 is from 2014. Solar manufacturing has since become less carbon-intensive per kWh, so today's solar figure is likely lower than 48. AR5 is used because it publishes a single, citable table with ranges for every technology, and the ranges are shown wherever a median is.
- AR5's technology values are global literature medians. They are not specific to Chinese supply chains.

## Corrections

> **2026-09-19.** An audit of the first version against its sources found errors that changed its conclusions. All are fixed, and each has a test that fails if it returns.
>
> - The lifecycle values were attributed to IPCC AR6. They are AR5's; AR6 does not publish this table.
> - Utility-scale solar was given 41, which is AR5's rooftop median. The utility-scale median is 48 (range 18–180). The nuclear and hydropower ranges are now as printed in Table A.III.2.
> - A construction / fuel / operation / decommissioning breakdown had no source, and for coal, gas and offshore wind its parts did not add up to the totals. It has been removed; AR5's own component columns are in the data file instead.
> - The generation shares summed to 95% by 2024. That counted the missing generation as zero-emission and overstated the decline in grid intensity (−22%, against −15% on real shares). Coal's 2024 share is 58%, not 54%. The mix now comes from OWID.
> - The regional grid factors did not match the official figures they were attributed to. The value labelled "South" (0.5271) is the Central grid's. They are now the official 2023 factors.
> - Avoided emissions were reported a thousand times too small, from a tonnes conversion error. Ratios between regions were unaffected, which is why the error went unnoticed.
> - A 2030 scenario based on an assumed generation mix has been removed. China's 2030 target is stated for primary energy, not electricity.

## Layout

```
data/
  ar5_lifecycle_emissions.csv         A.III.2, lifetimes from A.III.1
  china_generation_mix_2015_2024.csv  OWID / Ember / Energy Institute
  mee_2023_grid_factors.csv           2023 factors, URL and SHA-256
  china_grid_intensity.csv            derived intensity by year
scripts/
  fetch_sources.py    downloads sources, checks the annex hash
  lca_calculation.py  intensity, avoided emissions, payback
notebooks/
  analysis.ipynb      the analysis, executed
tests/                every input pinned to its source
outputs/              figures
```

## Citation and license

Cite this repository ([`CITATION.cff`](CITATION.cff)) and the sources it applies: Schlömer et al. (2014), IPCC AR5 WGIII Annex III; the Our World in Data energy dataset (Ember, Energy Institute); and the 2023 grid factor notice above.

Code is MIT. For data, see the sources above.

See also [green-ai-ledger](https://github.com/JoooDev-TPP/green-ai-ledger), which pins compute carbon to the same 2023 notice, and [GBA-Air-Quality-Analysis](https://github.com/JoooDev-TPP/GBA-Air-Quality-Analysis), Greater Bay Area air quality 2015 to 2024.
