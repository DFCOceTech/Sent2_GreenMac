# Changelog — Sent2_GreenMac

Rolling 2-week work log.

## 2026-04-26 (3)
- Bug fix: S2A/S2B band name differences in Step 4 and masking
  - `write_rgb_geotiff`: green band now auto-detected from filename — `rhorc_560` (S2A) or `rhorc_559` (S2B)
  - `compute_indices`: SWIR band (`rhorc_1614` / `rhorc_1610`) and FAI baseline wavelength (1614 / 1610 nm) now auto-detected
  - `mask_config.yml` cloud fallback: was `rhot_1614` (wrong), now `rhot_1377` (S2B cirrus band)
  - `mask_config.yml` land fallback: was `rhorc_2202` (SWIR2, wrong), now `rhorc_1610` (S2B SWIR1)
  - Updated rrc-post-processing spec REQ-RRC-001 and REQ-RRC-004; masking spec REQ-MASK-001/002 and SCENARIO-MASK-001

## 2026-04-26 (2)
- S01-01: per-scene ACOLITE output directories
  - Added `scene_output_dir(acolite_out, l1c)` helper: parses sat/date/tile from SAFE name → `{sat}_{yyyymmdd}_{tile}`
  - Replaced `acolite_output_dir` (used full product ID) with `scene_output_dir`
  - Updated `find_l2r_files` to search per-scene subdir first, recursive glob fallback
  - Removed `output={acolite_out}` from `set_str`; written as `output={scene_out}` per-iteration in run cell
  - Fixed hardcoded product name in `acolite-run-cc` → `.SAFE`
  - Fixed `dd8af76b` post-processing cell: now loops `l1c_dirs`, uses `find_l2r_files` (flat glob removed)
  - Updated `secrets_config.yml`: `acolite_out` now points to root output dir (not a scene subdir)
  - Created Epic 01, Story S01-01; updated ACOLITE spec with REQ-ACOL-006 and two new scenarios

## 2026-04-26
- S03-01: introduced `secrets_config.yml` pattern for credentials and local paths
  - Created `secrets_config.yml` (gitignored) with CDSE, Earthdata credentials and local paths
  - Created `secrets_config.yml.example` (committed) as onboarding template
  - Created `.gitignore` (excludes `secrets_config.yml`, `.DS_Store`, `.ipynb_checkpoints/`)
  - Added `secrets-load-cc` cell to notebook (after imports): loads YAML and assigns all variables
  - Stripped `config-cc` of `username`, `password`, `download_path`
  - Stripped `acolite-config-cc` of `earthdata_user`, `earthdata_pass`, `acolite_dir`, `acolite_out`, `mask_config_path`
  - Created openspec capability `secrets-config`, Story S03-01, Epic 03

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
