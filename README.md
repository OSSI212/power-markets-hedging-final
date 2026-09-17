# Power Markets - Trading and Hedging a Retail Electricity Portfolio

Case study: hedge a retail electricity portfolio for 2024. The analysis builds a customer
load forecast and an Hourly Price Forward Curve (HPFC), optimizes a futures hedge, runs
Day-Ahead shaping and imbalance settlement, and recommends a procurement strategy
(UNHEDGED / COARSE_CAL / GRANULAR).

This repository contains the reproducible code for the submission: the data briefing
(Exhibits 1-4) and the main analysis notebooks (Stages 1-8).

## Setup

    pip install -r requirements.txt

## Structure

- `data briefing/` - initial data exploration, per the assignment's Data Briefing task
  - `Exhibit 1 - Customer portfolio/`
  - `Exhibit 2 - Futures market/`
  - `Exhibit 3 - Historical price shape/`
  - `Exhibit 4 - Realised market prices/`
  - `Close the briefing/` - the three review questions, answered in English
- `notebooks/` - the main analysis, run in order:
  - `1_Forcast.ipynb` - customer load forecast
  - `2_hpfc.ipynb` - Hourly Price Forward Curve
  - `3_hedge.ipynb` - futures hedge optimization
  - `4_delivery.ipynb` - Day-Ahead shaping
  - `5_settlement.ipynb` - imbalance settlement
  - `6_analysis.ipynb` - strategy comparison and recommendation
  - `7_event.ipynb` - interpret a severe Day-Ahead price event and its effect on each strategy
  - `8_extensions_pricing.ipynb` - optional extensions: an ex-ante fixed customer price, and a
    QUARTER_ONLY hedge design tested against UNHEDGED/COARSE_CAL/GRANULAR
- `data/` - source CSVs and intermediate results produced by the notebooks; see
  `data/README.md` for a description of every file

## Notes

- All notebooks read data via a relative path (`../data`, or `../../data` for the two-levels-deep
  briefing notebooks), so run them from the `notebooks/` or `data briefing/<exhibit>/` folder.
- Each notebook writes its outputs to `data/` as CSV so later stages can read them - run the
  notebooks in the order listed above for a clean, reproducible pipeline.
