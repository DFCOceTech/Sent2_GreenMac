# Operational Status — Sent2_GreenMac

> Last updated: 2026-04-25

## Current Status

### What's Working
- CDSE OData query with cloud-cover and tile-area filters
- Numbered thumbnail display with index-based product selection
- Selective download of chosen products
- ACOLITE Rrc (rhorc) + DSF residual glint correction
- Configurable band exclusion (rhot, rhos, rhorc-edges independently togglable)
- Cloud / land / ice-bright masks with configurable thresholds (mask_config.yml)
- Combined mask with optional dilation
- Mask embedding in _RRC.nc and/or separate _mask.nc

### What's Working (updated 2026-04-14)
- Step 4 in `CDSE_OData_Workflow_v1.ipynb`: writes RGB, NDVI, FAI GeoTIFFs from any `_RRC.nc`
- 7×7 median filter on rhorc bands before index computation
- NDVI with NaN guard; FAI (NIR minus red–SWIR baseline)
- Combined mask applied to NDVI and FAI outputs; RGB intentionally unmasked
- Verified on `T22WDS_L2R_RRC.nc`: all acceptance criteria pass

### What's Working (updated 2026-04-26, update 2)
- Per-scene ACOLITE output: each scene written to `{acolite_out}/{sat}_{yyyymmdd}_{tile}/`
- Post-processing loops all scenes in `l1c_dirs`; `find_l2r_files` uses per-scene subdir

### What's Working (updated 2026-04-26)
- `secrets_config.yml` pattern: all credentials and local paths loaded from gitignored file
- `.gitignore` excludes `secrets_config.yml`, `.DS_Store`, `.ipynb_checkpoints/`
- `secrets_config.yml.example` documents schema for new users

### What's Working (updated 2026-04-25)
- FAI patch detection in Step 4: `compute_fai_patches` thresholds masked FAI, labels 4-connected components, filters by area
- `write_fai_report` writes `%`-header + CSV (num, longitude, latitude, area) keyed to `lon`/`lat` from RRC NetCDF
- Output filename: `_FAI_report.txt` alongside the GeoTIFFs in `analysis_out`

### What's Next
1. Verify FAI patch detection against a known scene (check report centroid accuracy)
2. Visualise outputs (QGIS or matplotlib quicklook cell)
3. Validate mask outputs against known scenes
4. Performance test on full workflow (download → ACOLITE → Step 4)
