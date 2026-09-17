# Data

The files under "Provided by the course" below are the raw inputs; `data_dictionary.csv` has
their per-column definitions and `sources.csv` the publishers and links. Everything else in this
folder is an intermediate or final result written by the notebooks in `notebooks/` - see each
notebook's own **Input**/**Output** note for exactly which file it reads and writes.

## Common conventions

- Time series are quarter-hourly. Join on `timestamp_utc`; `timestamp_local` is
  the same instant in Europe/Berlin.
- 2024 has 35,136 quarter-hours (leap year). The clock changes on 31 Mar and
  27 Oct are already in the timestamps, no extra adjustment needed.
- Energy in MWh, prices in EUR/MWh. A quarter-hour MWh value = 0.25 h x MW.
- Peak = Monday-Friday 08:00-20:00 local time. Everything else is Off-Peak.

## Files provided by the course

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

## Files written by the notebooks (pipeline outputs)

| File | Written by | What it is |
|------|------------|------------|
| `forecast_load.csv` | `1_Forcast.ipynb` | Quarter-hourly forecast load per customer group and total. |
| `hpfc.csv` | `2_hpfc.ipynb` | The constructed Hourly Price Forward Curve for 2024. |
| `selected_products.csv` | `2_hpfc.ipynb` | The eligible non-overlapping futures (COARSE_CAL and GRANULAR) used to build the curve and price the hedge. |
| `hedge_positions.csv` | `3_hedge.ipynb` | Optimal Base/Peak futures position per block, per strategy. |
| `hedge_energy.csv` | `3_hedge.ipynb` | Resulting hedge energy per quarter-hour, per strategy. |
| `residual_da.csv` | `4_delivery.ipynb` | Per-quarter-hour Day-Ahead residual for every strategy. |
| `forecast_costs.csv` | `4_delivery.ipynb` | Futures + Day-Ahead-residual cost per strategy (before imbalance). |
| `imbalance.csv` | `5_settlement.ipynb` | Per-quarter-hour imbalance volume and price. |
| `final_costs.csv` | `5_settlement.ipynb` | Final procurement cost per strategy, including imbalance settlement. |
| `strategy_comparison.csv` | `6_analysis.ipynb` | The full eight-measure strategy comparison behind the management recommendation. |
| `severe_event.csv` | `7_event.ipynb` | The event-window data behind the December 2024 price-event discussion. |
| `quarter_only_positions.csv` | `8_extensions_pricing.ipynb` (Extension 2) | Futures positions for the QUARTER_ONLY hedge design. |
| `quarter_only_costs.csv` | `8_extensions_pricing.ipynb` (Extension 2) | Full cost comparison including QUARTER_ONLY. |
