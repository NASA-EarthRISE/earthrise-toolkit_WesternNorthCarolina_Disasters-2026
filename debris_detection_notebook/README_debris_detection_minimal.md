# NASA HLS and OPERA Storm-Debris Mapping

Map likely post-storm forest debris concentrations using NASA HLS optical imagery, OPERA RTC-S1 radar imagery, NLCD forest cover, robust multi-year change metrics, and optional Local Moran's I clustering.

The notebook is interactive. Nearly every analysis setting is entered through checked prompts, and the imagery-access method is selected later when the workflow reaches the streaming/downloading stage.

## Documentation

| File | Description |
|------|-------------|
| `debris_detection_user_input.ipynb` | Main interactive notebook. Run the cells from top to bottom. |
| `README_debris_detection.md` | System requirements, inputs, configuration reference, instructions, workflow, and outputs. |
| `environment.yml` | Optional Conda environment definition when included with the larger repository. |

---

## Workflow Steps

| Step | Notebook Section | Description |
|------|------------------|-------------|
| 1 | Import packages | Load the required Python libraries. |
| 2 | Enter analysis settings | Prompt for the study area, dates, grid, imagery, indices, composites, runtime, plots, and Local Moran settings. |
| 3 | Sign in | Authenticate with NASA Earthdata. |
| 4 | Create the study area | Merge county or vector features, save the AOI, build the search rectangle, and define the raster grid. |
| 5 | Search for imagery | Search HLS and OPERA RTC-S1 for every baseline and target year. |
| 6 | Choose imagery access | Prompt for streaming, downloading, or automatic selection, plus pilot/full processing settings. |
| 7 | Process imagery | Align bands, apply quality masks, calculate optical and radar indices, and cache each scene. |
| 8 | Build composites | Align NLCD, keep forest classes, and create annual median composites. |
| 9 | Calculate change | Build the multi-year median/MAD baseline and calculate difference, percentage change, and robust anomaly layers. |
| 10 | Create scores | Create spectral, radar, and combined 0-1 debris-likelihood scores, tables, and figures. |
| 11 | Local Moran's I | Optionally aggregate the score and identify significant HH, LH, LL, and HL clusters. |
| 12 | Display and export | Display the latest maps and save configuration, system, software, and output inventories. |

---

## System Used

The workflow was developed for and previously run on the following reference system:

| Component | Reference System |
|-----------|------------------|
| Operating system | Windows 64-bit |
| Python interface | Jupyter Notebook/JupyterLab in a Conda environment |
| Reference Python | Python 3.13.10 (`earthrise` kernel) |
| Supported Python target | Python 3.11 or newer |
| Processor availability | 16 physical cores / 22 logical processors |
| Memory | Approximately 64 GB RAM |
| Storage | Local SSD recommended for downloaded imagery and cached rasters |



---

## Required Files and Accounts

| Requirement | Required | Description |
|-------------|----------|-------------|
| `debris_detection_user_input.ipynb` | Yes | The notebook containing the complete workflow. |
| Local NLCD raster | Yes | A land-cover raster covering the study area. The file path is entered at the start of the notebook. |
| AOI vector file | Conditional | Required only when `aoi_method = vector`. Supported by GeoPandas, such as GeoJSON, shapefile, or GeoPackage. |
| County boundary file | No | Downloaded automatically from Census TIGER/Line when county FIPS input is used. |
| NASA Earthdata Login | Yes | Required to search and access HLS and OPERA data. |
| ASF data-use agreement | Yes for OPERA | Accept once through ASF Vertex before accessing OPERA RTC-S1. |


Large HLS and OPERA files are not included in the repository. They are streamed or downloaded during the run.

---

## Python Packages

### Required packages

| Package | Purpose |
|---------|---------|
| `earthaccess` | NASA Earthdata authentication, search, streaming, and downloading |
| `geopandas` | Study-area and vector processing |
| `shapely` | Geometry validation, merging, and bounding boxes |
| `rasterio` | Raster alignment, masking, windowed processing, and GeoTIFF output |
| `pyproj` | Coordinate reference systems and transformations |
| `numpy` | Array calculations and index formulas |
| `pandas` | Search summaries, statistics, and CSV outputs |
| `matplotlib` | Figures and raster previews |
| `ipyleaflet` | Drawing the AOI and displaying interactive maps |
| `scipy` | False-discovery-rate correction and supporting statistics |
| `libpysal` | Spatial weights for Local Moran's I |
| `esda` | Local Moran's I calculations |
| `pyyaml` | Saving the completed configuration as YAML |
| `psutil` | Recording CPU and memory information |
| `jupyterlab` or `notebook` | Running the `.ipynb` file |

### Example environment setup

```bash
conda create -n nasa-debris-workflow python=3.13 -y
conda activate nasa-debris-workflow
pip install earthaccess geopandas shapely rasterio pyproj numpy pandas matplotlib \
    ipyleaflet scipy libpysal esda pyyaml psutil jupyterlab
jupyter lab
```

---

## User Configuration Reference

Every setting below is entered through notebook prompts. Press **Enter** to accept the displayed default.

### Study Area and Time

| Setting | Default | Description |
|---------|---------|-------------|
| `output_directory` | `debris_output` | Folder receiving all outputs. |
| `baseline_years` | `2021,2022,2023` | Years representing normal pre-event conditions. |
| `target_years` | `2024,2025` | Years compared with the baseline. |
| `season_start` | `10-01` | First month-day searched in every year. |
| `season_end` | `10-30` | Last month-day searched in every year. |
| `analysis_crs` | `EPSG:5070` | Projected CRS used for the analysis grid. |
| `analysis_resolution_m` | `30` | Analysis pixel size in meters. |
| `aoi_method` | `fips` | Use county FIPS codes, a vector file, or a drawn polygon. |
| `county_fips` | Western NC county list | Counties merged when the FIPS method is selected. |
| `tiger_year` | `2023` | Census TIGER/Line county-boundary year. |
| `aoi_path` | Required for vector mode | Path to an existing AOI vector file. |
| `nlcd_source_path` | Required | Path to a local NLCD raster. |
| `nlcd_product_year` | `2024` | NLCD year recorded in the run configuration. |
| `forest_classes` | `41,42,43` | NLCD classes retained as forest. |
| `nlcd_resampling` | `nearest` | Categorical resampling used to align NLCD. |

### HLS and OPERA Imagery

| Setting | Default | Description |
|---------|---------|-------------|
| `hls_products` | `HLSL30,HLSS30` | HLS Landsat and Sentinel-2 collections searched. |
| `max_cloud_cover` | `20` | Maximum reported HLS granule cloud cover percent. |
| `hls_minimum_observations` | `3` | Minimum valid HLS scenes required for an annual pixel. |
| `hls_indices` | `ndvi,evi,nbr,ndmi,dfi,ndti` | Optical indices calculated and composited. |
| `mask_flag_names` | all six flags | HLS cirrus, cloud, adjacent-cloud, shadow, snow, and high-aerosol flags removed. |
| `reflectance_scale` | `0.0001` | Scale applied to HLS reflectance values. |
| `opera_short_name` | `OPERA_L2_RTC-S1_V1` | OPERA collection searched through Earthdata. |
| `opera_required_assets` | `VV,VH,mask` | OPERA files required for each burst. |
| `opera_minimum_observations` | `3` | Minimum valid OPERA scenes required for an annual pixel. |
| `radar_indices` | `vv_db,vh_db,rvi,mrdfi,vh_vv_ratio,dprvi` | Radar layers calculated and composited. |
| `opera_valid_mask_value` | `0` | OPERA mask value treated as valid. |
| `opera_resampling` | `bilinear` | Resampling used for continuous OPERA data. |

### Baseline, Change, and Scores

| Setting | Default | Description |
|---------|---------|-------------|
| `minimum_baseline_year_count` | `3` | Minimum annual composites needed to build a baseline. |
| `mad_scale` | `1.4826` | Scale converting MAD to a robust spread estimate. |
| `minimum_spread_floors` | index-specific mapping | Minimum spread allowed for each optical and radar variable. |
| `percentage_excluded_variables` | `dfi,vv_db,vh_db` | Variables that do not receive percentage-change outputs. |
| `percentage_minimum_baseline_magnitude` | index-specific mapping | Smallest baseline magnitude allowed in percentage-change denominators. |
| `spectral_greenness` | `ndvi,evi` | Indices combined in the greenness group. |
| `spectral_moisture` | `nbr,ndmi` | Indices combined in the moisture group. |
| `spectral_exposed` | `dfi,ndti` | Indices combined in the exposed-material group. |
| `radar_inputs` | `rvi,mrdfi,dprvi` | Radar anomalies combined in the radar score. |
| `spectral_disturbance_direction` | decreases for greenness/moisture; increases for exposed material | Direction that contributes to optical disturbance. |
| `z_cap` | `3.0` | Robust-anomaly magnitude mapped to the maximum score of 1. |

### Performance, Display, and Sampling

| Setting | Default | Description |
|---------|---------|-------------|
| `analysis_chunk_pixels` | `1536` | Block size for windowed change calculations. |
| `composite_block_pixels` | `384` | Block size for annual composites and debris scores. |
| `preview_max_dimension_pixels` | `1000` | Maximum width or height of notebook previews. |
| `distribution_sample_size` | `20000` | Maximum sampled pixels per distribution plot. |
| `distribution_seed` | `0` | Random seed used for repeatable plot sampling. |

### Imagery Access Prompt

These values are requested only after the imagery search has finished.

| Setting | Default | Description |
|---------|---------|-------------|
| `hls_access_choice` | `auto` | Stream HLS, download HLS, or benchmark both and choose automatically. |
| `opera_access_choice` | `auto` | Stream OPERA, download OPERA, or benchmark both and choose automatically. |
| `run_size_choice` | `pilot` | Limit scenes for testing or run the full search result. |
| `pilot_max_scenes_per_product_year` | `3` | Maximum scenes used for each product-year in pilot mode. |
| `scene_workers` | `2` | Number of scenes processed in parallel. |
| `download_threads` | `4` | Number of threads available to download assets. |
| `auto_benchmark_scene_count` | `1` | Number of scenes used to compare streaming and downloading. |

### Local Moran's I

| Setting | Default | Description |
|---------|---------|-------------|
| `run_local_moran` | `yes` | Run or skip Local Moran's I. |
| `lisa_variable` | `index_composite_combined` | Combined, spectral, or radar score analyzed. |
| `lisa_support_resolution_m` | `90` | Resolution used for Local Moran units; coarser values reduce runtime and memory. |
| `lisa_permutations` | `99` | Random permutations used for local significance. |
| `lisa_alpha` | `0.05` | Significance level after false-discovery-rate correction. |
| `lisa_weights_method` | `queen` | Queen or rook raster-cell neighbors. |
| `lisa_row_standardize` | `yes` | Row-standardize spatial weights. |
| `lisa_minimum_valid_fraction` | `0.5` | Minimum valid area required within an aggregated support cell. |
| `lisa_maximum_units` | `250000` | Safety limit preventing an excessively large spatial-weights problem. |
| `lisa_random_seed` | `42` | Random seed used by permutation testing. |
| `lisa_alternative` | `two-sided` | Statistical alternative passed to the Local Moran calculation. |
| `lisa_n_jobs` | `1` | Worker processes used by `esda.Moran_Local`. |

---

## How to Run

1. Install and activate the Python environment.
2. Create and verify a NASA Earthdata Login.
3. Open ASF Vertex once, sign in, and accept the OPERA data-use agreement.
4. Place the local NLCD raster somewhere accessible to the notebook.
5. Open `debris_detection_user_input.ipynb` in Jupyter.
6. Run the cells from top to bottom.
7. Answer the main configuration prompts. Press **Enter** to accept a default.
8. Draw the AOI when `drawn` is selected, then continue to the next cell.
9. Review the AOI and imagery-search summary before continuing.
10. At the imagery-access section, choose streaming, downloading, or automatic selection.
11. Use `pilot` mode for the first test run, then use `full` mode for the complete analysis.
12. Review the final maps and the `tables/output_inventory.csv` file.

Do not use **Run All** when the drawn-AOI method is selected, because the notebook must pause while the polygon is drawn.

---

## What the Notebook Does

The workflow:

1. Creates one merged study area and a rectangular search extent.
2. Searches NASA Earthdata for HLS and OPERA RTC-S1 scenes.
3. Lets the user stream data, download data, or automatically benchmark both methods.
4. Aligns all raster inputs to one projected grid.
5. Masks HLS clouds, cirrus, adjacent cloud, cloud shadow, snow, and high aerosol.
6. Masks invalid OPERA radar pixels.
7. Calculates six optical and six radar variables.
8. Keeps NLCD forest classes and clips all analysis products to the AOI.
9. Creates annual median composites and valid-observation counts.
10. Builds a multi-year median baseline and robust MAD-based spread.
11. Calculates target-year difference, percentage change where appropriate, and robust anomaly.
12. Produces spectral, radar, and combined 0-1 debris-likelihood scores.
13. Optionally calculates Local Moran's I clusters at a user-selected support resolution.
14. Exports maps, rasters, figures, tables, configuration, system information, and an output inventory.

The 0-1 debris-likelihood score is a relative change score. It is not a probability and does not confirm debris without additional interpretation or validation.

---

## Output Structure

```text
debris_output/
├── study_area.geojson
├── search_bbox.geojson
├── config/
│   ├── workflow_config.yaml
│   ├── software_versions.csv
│   └── run_manifest.json
├── searches/
│   ├── hls_granules.json
│   └── opera_rtc_granules.json
├── downloads/                 # Created when download access is used
├── cache/
│   ├── hls/
│   └── opera_rtc/
├── masks/
├── composites/
├── baselines/
├── change/
├── candidates/
├── lisa/                      # Created when Local Moran's I is run
├── previews/
├── figures/
└── tables/
    └── output_inventory.csv
```

### Primary Outputs

| Output | Description |
|--------|-------------|
| `study_area.geojson` | Dissolved study-area boundary. |
| `search_bbox.geojson` | Rectangular imagery-search boundary. |
| `config/workflow_config.yaml` | Complete set of user-entered settings. |
| `config/software_versions.csv` | Operating system, hardware summary, Python, and package versions. |
| `config/run_manifest.json` | Run date, AOI, grid, software, and complete configuration. |
| `searches/*.json` | HLS and OPERA search results and asset URLs. |
| `cache/hls/*.tif` | Masked and aligned HLS index stacks for individual scenes. |
| `cache/opera_rtc/*.tif` | Masked and aligned OPERA radar index stacks for individual scenes. |
| `masks/forest_mask.tif` | NLCD-derived forest mask on the analysis grid. |
| `composites/hls_<year>.tif` | Annual HLS median index composite. |
| `composites/opera_rtc_<year>.tif` | Annual OPERA median radar composite. |
| `composites/*_count.tif` | Valid-observation count for each annual composite. |
| `baselines/*` | Baseline median, MAD, and robust-spread rasters. |
| `change/*` | Difference, percentage-change, and robust-anomaly rasters. |
| `candidates/spectral_score_<year>.tif` | Optical debris-likelihood score from 0 to 1. |
| `candidates/radar_score_<year>.tif` | Radar debris-likelihood score from 0 to 1. |
| `candidates/combined_score_<year>.tif` | Equal-sensor combined debris-likelihood score from 0 to 1. |
| `lisa/lisa_cluster_<variable>_<year>.tif` | Significant Local Moran cluster classes. |
| `tables/lisa_cluster_counts_*.csv` | Unit counts and area for each cluster class. |
| `figures/*.png` | Temporal plots, distributions, and map overlays. |
| `tables/output_inventory.csv` | Final list of every output file and its size. |



## Reproducibility

Each run saves:

- Every user-entered setting in `workflow_config.yaml`
- The full configuration again inside `run_manifest.json`
- The operating system and hardware summary
- Python and package versions
- Search-result metadata
- An inventory of every generated file

These files should be kept with the final rasters and tables when the analysis is shared or repeated.
