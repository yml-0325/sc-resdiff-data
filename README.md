# SC-ResDiff: joint wind–PV–load and weather data

Core research data accompanying **Support-conditioned residual diffusion for joint wind–PV–load scenario generation in day-ahead scheduling under underrepresented weather conditions**.

[中文说明](README_zh.md)

## Coverage

Germany (DE), Austria (AT), and France (FR), **24 March 2021–31 December 2025**, hourly UTC. The snapshot contains 1,744 days, 41,856 hours per country, and 125,568 country-hour CSV rows.

| Split | Dates | Days |
|---|---|---:|
| Training | 2021-03-24–2023-12-31 | 1,013 |
| Validation | 2024-01-01–2024-12-31 | 366 |
| Test | 2025-01-01–2025-12-31 | 365 |

## Download and files

Download the ZIP asset from [release v1.0.0](https://github.com/yml-0325/sc-resdiff-data/releases/tag/v1.0.0), or clone this repository.

- `data/hourly_data.csv`: physical-unit observations, weather conditions, source codes and capacity fields.
- `data/wind_pv_load_weather_2021_2025.npz`: original snapshot, including 29 arrays with provenance and missingness masks.
- `splits/train.csv`, `splits/validation.csv`, `splits/test.csv`: dates, zero-based day indices and forecast initialization times.
- `DATA_DICTIONARY.json`: array schemas, variable names, source codes and counts.
- `VALIDATION.json`: checks performed during packaging.
- `SHA256SUMS.txt`: file checksums.

## Variables

Both `targets` and `conditions` have shape `(1744, 24, 3, 3)`: day × hour × country × variable. Country order is **DE, AT, FR**.

| Array | Variable order | Units |
|---|---|---|
| targets | load, wind, solar PV | MW, MW, MW |
| conditions | 2 m temperature, 100 m wind speed, downward shortwave radiation | °C, m/s, W/m² |

No new normalization is applied. Any fitted normalization should use training data only. Capacity fields are auxiliary fields retained from the research snapshot.

```python
import numpy as np

with np.load('data/wind_pv_load_weather_2021_2025.npz', allow_pickle=False) as data:
    targets = data['targets']
    weather = data['conditions']
    train_mask = data['split'] == 'train'
    print(targets[train_mask].shape, weather[train_mask].shape)
```

Reading NPZ requires Python and NumPy. CSV files can be read independently.

## Forecast timing and processing

Weather conditions are NOAA/NCEP GFS **forecasts**, initialized at 00 UTC on the day before the target day. Instantaneous fields use forecast leads +24 to +47 h; radiation uses interval means at +25 to +48 h, reconstructed to hourly values. Regional aggregation uses grid-cell intersection areas with NUTS 2016 level-0 boundaries. All time alignment is UTC, without local daylight-saving adjustments.

Final target and condition arrays contain no missing values. Raw provenance arrays intentionally retain NaN. Targets primarily use ENTSO-E observations; French gaps use RTE data. Some French wind totals combine ENTSO-E onshore and RTE offshore generation. One French load value at 2021-10-31 00:00 UTC is linearly interpolated from adjacent hours.

Source codes: -1 missing; 0 ENTSO-E; 1 RTE; 2 short-gap linear interpolation; 3 ENTSO-E onshore plus RTE offshore. Historical source revisions may prevent a fresh download from matching this snapshot exactly.

## Sources and terms

- [ENTSO-E Transparency Platform](https://transparency.entsoe.eu/): actual load and generation.
- [RTE éCO2mix](https://odre.opendatasoft.com/explore/dataset/eco2mix-national-cons-def/): French fallback observations; Licence Ouverte v2.0.
- [NOAA/NCEP GFS](https://registry.opendata.aws/noaa-gfs-bdp-pds/): day-ahead weather forecasts.
- [Eurostat GISCO NUTS](https://gisco-services.ec.europa.eu/distribution/v2/nuts/): boundaries used in aggregation. © EuroGeographics for the administrative boundaries.
- [Fraunhofer ISE Energy-Charts](https://api.energy-charts.info/): installed capacities.

Third-party data remain subject to the terms of their original providers. See [ENTSO-E terms](https://transparencyplatform.zendesk.com/hc/article_attachments/40921869376401) and [GISCO terms](https://ec.europa.eu/eurostat/web/gisco/geodata/statistical-units).

## Dataset and citation

This repository provides processed wind, PV, and load data, aligned weather forecasts, and dataset splits.

For a stable reference, cite **SC-ResDiff core data, version v1.0.0** and link to the [tagged snapshot](https://github.com/yml-0325/sc-resdiff-data/tree/v1.0.0). The original NPZ is preserved byte-for-byte; every CSV target and weather value was checked against it.
