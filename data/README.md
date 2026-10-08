# Data

Raw data is not included in this repository due to size (~152 MB compressed).

- Source: [NASA OceanColor — MODIS Level-3](https://www.earthdata.nasa.gov/data/tools/ocean-color-level-3-4-browser)
- Variables: Chlorophyll-a (`chlor_a`) and Sea Surface Temperature (SST)
- Dimensions: time × latitude × longitude
- Temporal resolution: 8-day composite, 2014–2024
- Spatial resolution:  4 km 
- Area: WPP-NRI 572 (Indian Ocean, west of Sumatra and Sunda Strait), 90°E–110°E, 10°S–7°N

Download the files from the source above and place them in this folder before running `notebooks/1-data-modis-merge.ipynb`.