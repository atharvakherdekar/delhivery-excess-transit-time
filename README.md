# Delhivery Excess Transit-Time Prediction

Explainable machine learning for excess transit-time prediction and risk
prioritization in Indian logistics, built on the publicly circulated Delhivery
dataset (September–October 2018).

**Course:** Big Data Analytics, Goa Institute of Management
**Authors:** Atharva Kherdekar (B2026067), Manjit Kalita (B2026087), Yash Tewari (B2026108)

## What this project does

- Rebuilds 144,867 segment-level records into 14,817 trip-level observations
  (via 26,369 origin–destination legs).
- **Regression:** predicts excess transit time `E = actual − OSRM` in minutes.
- **Classification:** flags a trip as *high-excess* when actual transit time is
  more than twice the OSRM estimate (`(actual − OSRM) / OSRM > 1.0`).
  This is a study-defined threshold, **not** a Delhivery SLA.
- Evaluates on a chronological hold-out set and explains the classifier with SHAP.

## Headline results (4,163 held-out test trips)

| Task | Model | Result |
|---|---|---|
| Regression (MAE, min) | OSRM zero-excess baseline | 191.61 |
| | Random Forest | 68.84 |
| | CatBoost | **64.20** |
| High-excess classification (F1) | Flag every trip (trivial) | 0.722 |
| | Random Forest (t = 0.5) | 0.766 |
| | CatBoost (t = 0.355) | **0.795** (recall 0.874, ROC-AUC 0.836) |

See the paper for full tables, route-type errors and SHAP importance.

## Data

The raw CSV is **not** included in this repository.

1. Download the Delhivery dataset:
   <https://d2beiqkhq929f0.cloudfront.net/public_assets/assets/000/001/551/original/delhivery_data.csv>
2. Save it as `data/raw/delhivery_data.csv`.

## How to reproduce

```bash
git clone <this-repository-url>
cd delhivery-excess-transit-time
pip install -r requirements.txt
# place the CSV at data/raw/delhivery_data.csv (see Data above)
python src/run_pipeline.py
```

The pipeline fixes random seeds, rebuilds trips, records the chronological
partitions, trains the models, and writes metrics, predictions and figures to
`results/`.

**Environment:** Python `<fill in version>`; package versions are listed in
`requirements.txt`.

## Method summary

- **Trip construction (`src/reconstruct_trips.py`):** group raw records by
  `trip_uuid`, `od_start_time`, `od_end_time`; keep the last record in event
  order for each leg as its terminal cumulative observation; sum terminal leg
  values across the trip for actual time, OSRM time, OSRM distance and actual
  distance. Cumulative values are never summed across segments.
- **Chronological split by trip-creation date:** fit 12–22 Sep (7,981 trips),
  validation 23–26 Sep (2,673), test 27 Sep–3 Oct (4,163). Trips are rebuilt
  before splitting, so no trip crosses partitions.
- **Features (dispatch-time only):** route type, source/destination centre IDs,
  OSRM time and distance, departure hour and weekday, trip-creation hour,
  creation-to-departure lead time, OSRM distance-to-time ratio. Post-outcome
  fields (actual time, end times, segment durations, etc.) are excluded.
- **Models:** zero-excess and training-median baselines, Random Forest, CatBoost.
  The CatBoost classification threshold (0.355) is chosen on validation data only.

## Repository layout

```
data/raw/          source CSV goes here (git-ignored)
data/processed/    reconstructed trip-level dataset
src/               run_pipeline.py, reconstruct_trips.py
results/metrics/   regression, classification and route-type metrics
results/predictions/
results/figures/
notebooks/         optional exploration
```

## Limitations

- The data covers only 22 days in 2018 and may not reflect current operations
  or other Indian logistics operators.
- There is no promised delivery time or SLA in the data; the high-excess label
  is relative to OSRM, not an SLA breach.
- The terminal OSRM snapshot may differ from what a dispatcher sees at departure;
  availability of the full multi-leg plan at dispatch is assumed.
- No threshold-sensitivity study, scaled-OSRM or linear baseline, lane-level
  analysis, per-trip SHAP or calibration is reported.
- SHAP values describe the model, not causal effects.

## AI use

AI tools were used for brainstorming, code assistance and language editing. The
authors prepared the data, ran the experiments and verified the results.

## License

Code is released under the MIT License (see `LICENSE`). The Delhivery dataset
is not redistributed and remains subject to its original terms.
