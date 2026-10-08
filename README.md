# Upwelling Analysis and Chlorophyll-a Prediction with ConvLSTM (WPP-NRI 572)

Undergraduate thesis — Data Science.
Spatio-temporal analysis of upwelling intensity and chlorophyll-a prediction using ConvLSTM to identify potential fishing ground zones in Indonesian Fisheries Management Area WPP-NRI 572, 2014–2024.

## Problem
WPP-NRI 572 is a strategic fisheries area in the Indian Ocean, west of Sumatra. Its productivity is driven by upwelling, which brings cooler, nutrient-rich water to the surface and raises chlorophyll-a concentration, an indicator of primary productivity. This project detects and classifies upwelling from SST, predicts chlorophyll-a with ConvLSTM, and combines both to map potential fishing ground zones.

## Data
- Source: [NASA OceanColor — MODIS Level-3](https://www.earthdata.nasa.gov/data/tools/ocean-color-level-3-4-browser)
- Variables: Chlorophyll-a (`chlor_a`) and Sea Surface Temperature (SST)
- Dimensions: time × latitude × longitude
- Temporal resolution: 8-day composite, 2014–2024
- Spatial resolution: 4 km
- Area: WPP-NRI 572, 90°E–110°E, 10°S–7°N
- Raw data is not included; see `data/README.md`.

## Approach
1. **Data merging** — combine MODIS files into one spatio-temporal dataset.
2. **Spatial masking** — restrict analysis to the valid WPP-NRI 572 area.
3. **Gap filling** — reconstruct missing values (cloud cover) with 3D-DINEOF, followed by outlier handling.
4. **EDA** — spatial and temporal patterns of chlorophyll-a and SST.
5. **ConvLSTM modelling** — baseline model for chlorophyll-a prediction.
6. **Fine-tuning** — hyperparameter tuning.
7. **Masked ConvLSTM** — spatial mask added as an extra input channel.
8. **Upwelling index** — SST-Based Upwelling Index (UI_SST), with IQR-based classification into weak, intermediate, and strong upwelling.
9. **Final model** — refit the best configuration and integrate predictions with upwelling detection.

## Results

**Chlorophyll-a prediction — best model** (baseline, timestep = 3, spatial mask as extra channel; evaluated within the valid WPP-NRI 572 area)

| Metric | Value |
|---|---|
| MAE | 0.0230 |
| RMSE | 0.0317 |
| R² | 0.677 |
| Correlation (r) | 0.824 |

**Upwelling intensity distribution**

| Class | Proportion |
|---|---|
| Weak | 16.78% |
| Intermediate | 15.95% |
| Strong | 10.17% |

**Key findings**
- Valid upwelling events accumulate mostly offshore west of Sumatra, in the central to southern part of WPP-NRI 572.
- Potential fishing ground zones are historically concentrated in the same area during September–December.
- January 2025 is the most prominent period in the initial forecast.
- The fine-tuned configurations did not outperform the masked baseline.

## How to Run
1. Download the data (see `data/README.md`).
2. Install dependencies:
```bash
   pip install -r requirements.txt
```
3. Run the notebooks in order, from `1-data-modis-merge.ipynb` to `9-best-model-refit.ipynb`.`1-data-modis-merge.ipynb` to `9-best-model-refit.ipynb`.

## Results

**Chlorophyll-a prediction** (evaluated on the test set within the valid WPP-NRI 572 area)

| Model | MAE | RMSE | R² | Correlation (r) |
|---|---|---|---|---|
| **Baseline, timestep 3 + spatial mask (final)** | **0.0230** | **0.0317** | **0.677** | **0.824** |
| Hyperparameter-tuned (trial 24) | 0.0232 | 0.0325 | 0.662 | 0.823 |

Adding the spatial mask as an extra input channel reduced prediction error compared with the same configuration without the mask. Timesteps 2, 3, 4, 6, and 8 were tested; timestep 3 performed best.

**Upwelling intensity** (UI_SST, IQR-based classes, 9.37M valid observations)

| Class | Proportion |
|---|---|
| Non-upwelling | 57.11% |
| Weak | 16.78% |
| Intermediate | 15.95% |
| Strong | 10.17% |

**Key findings**
- Upwelling events peak in September and remain high through December; offshore waters west of Sumatra (central to southern WPP-NRI 572) show the highest accumulation.
- Potential fishing ground zones are concentrated around 96°E–103°E and 8°S–2°S, covering the central–southern west coast of Sumatra, the Sunda Strait, and the offshore area to the west.
- January 2025 is the most prominent period in the forecast.

## Limitations & What I Learned
- **Gap filling introduced extreme values.** 3D-DINEOF preserved the mean and median of chlorophyll-a but raised the maximum from 85.9 to 1,212.9 mg/m³, likely because SVD is sensitive to outliers. Lesson: handle outliers *before* reconstruction, not after.
- **Compute constraints limited the search.** Batch size was fixed at 2, the model used 3 ConvLSTM layers, and timesteps were capped at 8 to avoid out-of-memory errors. Longer timesteps may capture seasonal patterns better.
- **Tuning did not beat the baseline.** Likely causes are a limited number of trials, a narrow search space, and a 90:5:5 split that leaves small validation and test sets. Lesson: choose the final model on test performance, not validation loss alone.
- **Long-horizon forecasts flatten out.** Recursive forecasting accumulates error, and predictions become nearly flat after January 2025. Periodic refitting on new observations would help.
- **No ground-truth validation.** Fishing ground zones are based on oceanographic conditions only and have not been validated against actual catch data.

## Report
Full thesis: [docs/paper.pdf](docs/paper.pdf)