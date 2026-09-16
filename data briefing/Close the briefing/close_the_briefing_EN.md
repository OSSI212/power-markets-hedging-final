# Close the briefing

Short answers to the three review questions from the Data Description (section 3),
drawing on Exhibits 1-4. All figures are rounded to two decimal places.

---

## 1. What was known on 29 September 2023, and what became known only during delivery?

**Available when the hedge was designed (quote date 29 Sep 2023):**

- The futures snapshot in `futures_prices.csv`: Base and Peak forward prices and the
  `market_activity` tradability score for CAL, the four quarters and the twelve months of 2024.
- The customer book: number of customers and annual consumption per customer for the three
  groups HB, GB, LB (Exhibit 1).
- The published 2024 standard load profiles (`slp_profiles.csv`) - normalised shapes, not volumes.
- The historical Day-Ahead price shape derived from 2019-2022 (`shape_factors.csv`, Exhibit 3).

Everything needed to build the forecast load, the HPFC and the hedge is in this set. The hedge is
an **ex-ante** decision.

**Known only during delivery (2024):**

- Realised Day-Ahead prices (`day_ahead_prices.csv`) and realised imbalance prices / reBAP
  (`imbalance_prices.csv`), Exhibit 4.
- Actual customer consumption per quarter-hour (`actual_portfolio_load.csv`): 43,138.00 MWh
  realised against the 43,000.00 MWh forecast, and - more important than the annual total - the
  quarter-by-quarter deviations that drive the imbalance.
- The weather, the December 2024 cold spell (Day-Ahead peak 936.28 EUR/MWh on 12 December), the
  isolated June reBAP blow-out (14,999.99 EUR/MWh on 3 June), plant outages and fuel-price moves -
  every driver of the gap between the ex-ante plan and the realised outcome.

Stages 4-7 measure the hedge against information that did not exist when it was taken: the futures
payoff is locked in on 29 September 2023, the residual Day-Ahead and imbalance costs are not.

---

## 2. Data checks performed before trusting the analysis

- **Coverage.** Every 2024 series (`slp_profiles`, `shape_factors`, `day_ahead_prices`,
  `imbalance_prices`, `actual_portfolio_load`) has exactly 35,136 quarter-hours = 366 days x 96.
  `futures_prices` has 34 contracts (2 year, 8 quarter, 24 month).
- **Time base.** All series join on `timestamp_utc`. `timestamp_local` carries changing UTC
  offsets (+01:00 / +02:00), so it is used only for wall-clock features. The spring and autumn
  clock changes are already baked into the timestamps: Q1 = 8,732 quarter-hours, Q4 = 8,836;
  M03 = 2,972, M10 = 2,980.
- **Missing values.** None in any of the six files.
- **Units.** Energy in MWh, prices in EUR/MWh, SLP columns in normalised kWh. Day-Ahead is one
  hourly value repeated across its four quarter-hours (verified).
- **Internal consistency:**
  - each SLP column sums to exactly 1,000,000.00 kWh; the scaled forecast reconciles to
    E_p = 35,000.00 / 6,000.00 / 2,000.00 MWh and 43,000.00 MWh in total, to floating-point
    precision;
  - CAL Base (77.26) equals the N_all-weighted average of the four quarterly Base futures;
    CAL Peak (86.60) equals the N_peak-weighted average; Q1 Base (66.42) equals the average of
    M01-M03; the same holds for Q1 Peak (77.70);
  - the historical shape factor has annual mean 1.00;
  - the implied Off-Peak identity `P_base * N_all = P_peak * N_peak + P_off * N_off` reproduces the
    quoted Base price when the reconstructed hourly curve is averaged back over a block.
- **Tradability.** `market_activity` splits cleanly - illiquid <= 10, tradable >= 182, nothing in
  between. The usable / illiquid partition is unchanged for any threshold in that gap and matches a
  "at least 5 % of the same-load-type CAL activity" rule.
- **Plausibility.** Quantiles and extreme values of every price series were inspected; the
  Day-Ahead maximum (936.28 on 12 Dec) and the reBAP cap (14,999.99 on 3 Jun) were traced to
  specific dates and cross-checked against the daily-mean series.

---

## 3. What would still be missing for a real retail portfolio?

Five simplifications, with the likely direction of their effect:

1. **No trading frictions.** `market_activity` is a synthetic score; bid-ask spreads, transaction
   costs, broker fees and integer contract lot sizes are not modelled, and positions are
   continuous MW. *Effect:* the granular hedge would cost more to execute than shown - spreads
   widen for quarter and month products - which narrows, and could reverse, its advantage over the
   coarse CAL hedge.
2. **Standard load profiles instead of metered load.** The forecast applies fixed SLP shapes and
   assumes the customer count and per-customer consumption are known exactly, with no switching,
   churn or new acquisition during the year. *Effect:* real shape risk and volume risk are larger
   than the model shows, so the residual Day-Ahead cost (Stage 4) and the imbalance settlement
   (Stage 5) are understated - the hedge looks better than it would in practice.
3. **Historical price shape from 2019-2022.** Those years span the gas-price crisis and carry less
   installed solar than 2024. *Effect:* the HPFC shape can misprice the midday hours (more solar in
   2024) and the evening ramp, biasing the value-neutrality split and the residual-shape (RMSE)
   estimate in Stage 3; a systematic midday over-pricing is the most likely direction.
4. **A single quote date and no rebalancing.** The hedge is set once on 29 September 2023; there is
   no intraday market, no continuous trading and no adjustment as the load forecast is updated
   through 2024. *Effect:* a real desk would leg into the position over months and trim residual
   risk as information arrives, so the single-shot result is a conservative (pessimistic) estimate
   of residual exposure.
5. **Symmetric reBAP and no price impact.** Imbalance is settled at one published symmetric price,
   and the portfolio is assumed too small to move it. *Effect:* during a stressed, system-wide
   period such as the December 2024 cold spell, a large retail portfolio that is short would face
   worse settlement than the historical reBAP implies - the imbalance premium in Stage 5 is a lower
   bound for such events.

Further omissions: no credit, margin or collateral requirements; no risk limits or regulatory /
grid constraints; no renewable PPAs or demand-response; and reBAP is a settlement price, not a
forecast, so Stage 5 relies on a realised 2024 series that was itself unknown ex ante.
