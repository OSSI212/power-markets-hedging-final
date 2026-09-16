# Data

All files provided by the course. `data_dictionary.csv` has the per-column
definitions, `sources.csv` the publishers and links.

## Common conventions

- Time series are quarter-hourly. Join on `timestamp_utc`; `timestamp_local` is
  the same instant in Europe/Berlin.
- 2024 has 35,136 quarter-hours (leap year). The clock changes on 31 Mar and
  27 Oct are already in the timestamps, no extra adjustment needed.
- Energy in MWh, prices in EUR/MWh. A quarter-hour MWh value = 0.25 h x MW.
- Peak = Monday-Friday 08:00-20:00 local time. Everything else is Off-Peak.

## Files

| File | What it is | Key columns / notes |
|------|------------|---------------------|
| `slp_profiles.csv` | Standard load profiles for the three customer groups (Stromnetz Berlin) | `hb_normalized_kwh` households, `gb_normalized_kwh` general commercial, `lb_normalized_kwh` agriculture. Each column is a normalised shape, annual sum exactly 1,000,000 kWh. Scale with the customer table: HB 10,000 x 3.5 MWh, GB 200 x 30 MWh, LB 20 x 100 MWh (43,000 MWh total). |
| `futures_prices.csv` | Base/Peak futures snapshot, quote date 2023-09-29, one row per product | `product_type` YEAR/QUARTER/MONTH, `load_type` BASE/PEAK, `delivery_period` (CAL, Q1..Q4, M01..M12), `delivery_start` / `delivery_end_exclusive`, `price_eur_mwh`, `market_activity` = synthetic tradability score (higher = more liquid), used to pick usable products for the hedge. |
| `shape_factors.csv` | Relative Day-Ahead price shape from 2019-2022 | `historical_shape_factor` normalised to annual mean 1 (>1 above average, <1 below). Helper columns `month`, `hour`, `weekday` (Mon=0), `is_peak`. Used in Stage 2 to turn block futures into an hourly curve. |
| `day_ahead_prices.csv` | Realised 2024 DE/LU Day-Ahead price | `day_ahead_price_eur_mwh`. Hourly values repeated across the four quarter-hours of each hour. |
| `actual_portfolio_load.csv` | Realised 2024 consumption per group | `hb_actual_mwh`, `gb_actual_mwh`, `lb_actual_mwh`, `total_actual_mwh` (MWh per quarter-hour). Differs from the Stage 1 forecast - drives the imbalance in Stage 5. |
| `imbalance_prices.csv` | Realised 2024 German imbalance price (reBAP) | `imbalance_price_eur_mwh`, quarter-hourly. |
| `data_dictionary.csv` | Field definitions for every file | one row per column. |
| `sources.csv` | Publishers and links | — |

Note: the futures snapshot and the actual portfolio load are synthetic teaching
data; the load profiles and market prices come from public sources; the shape
factors are derived from public Day-Ahead data.
