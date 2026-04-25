# Changelog — Sent2_GreenMac

Rolling 2-week work log.

## 2026-04-25
- Fixed missed credential leak: `earthdata_pass` placeholder was a real NASA Earthdata password in `acolite-config-cc` — replaced with `'your_earthdata_password'`
- S02-02: added FAI patch detection to Step 4 of `CDSE_OData_Workflow_v1.ipynb`
  - New helpers: `compute_fai_patches`, `write_fai_report` in the RRC analysis helpers cell
  - Ported connected-components logic from `FAI_connected_components.ipynb`
  - Uses `mask_combined` from RRC NetCDF (no mask recomputation); geocodes centroids via `lon`/`lat` arrays
  - Step 4 config: added `fai_thrsh = 0.002`, `ar_thrsh = 10`
  - Step 4 execution: patch detection and report (`_FAI_report.txt`) appended after GeoTIFF outputs
  - Added `skimage.measure` import and `datetime` to imports cell
- Created openspec capability `fai-patches`, Story S02-02, updated Epic 02

## 2026-04-14
- Adopted `ops/tasks.md` workflow (spec-anchor story tracking)
- S02-01: added Step 4 to `CDSE_OData_Workflow_v1.ipynb` — RGB, NDVI, FAI GeoTIFF export
  - New helpers: `_build_geotiff_profile`, `write_rgb_geotiff`, `compute_indices`, `write_index_geotiff`
  - 7×7 median filter applied to rhorc bands before index computation
  - NDVI NaN guard (non-positive bands → NaN); FAI baseline interpolation (red–SWIR)
  - `mask_combined` applied to NDVI and FAI; RGB intentionally unmasked
- Installed rasterio 1.5.0 into cdsetool environment
- Fixed macOS ARM64 rasterio/osgeo import failure: recompiled `$JRE/lib/libiconv.2.dylib` to
  also export `_locale_charset` (needed by libspatialite → rasterio)
- Created openspec capability `rrc-post-processing`, Epic 02, Story S02-01
- Verified against `T22WDS_L2R_RRC.nc`: RGB 1830×1830 unmasked; NDVI in [-0.65, 0.37];
  FAI ocean median −0.000279 (negative); 1.53M masked pixels → 100% NaN in NDVI/FAI

## 2026-04-11
- Modified `CDSE_OData_Workflow_v1.ipynb` (21 cells):
  - Added index-labelled thumbnail display and `selected_indices` / `download_list` pattern
  - Added ACOLITE helpers: `acolite_output_dir`, `find_l2r_files`, `get_bands_to_exclude`
  - Added mask helpers: `load_mask_config`, per-mask compute functions, `combine_and_dilate`, `compute_masks`
  - Added `nc_copy_filtered` / `_write_mask_vars` for filtered NetCDF output with embedded or separate masks
  - Updated ACOLITE config cell with `exclude_rhot`, `exclude_rhos`, `exclude_rrc_edges` toggles and mask output options
  - Kernelspec updated to `cdsetool`
- Created `mask_config.yml` with default thresholds (cloud 0.025, land 0.05, ice/bright 0.25, dilation 3px)
- Created openspec capabilities: `thumbnail-selection`, `acolite-processing`, `masking`
