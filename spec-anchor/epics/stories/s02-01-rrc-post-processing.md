# S02-01: RRC Post-Processing — RGB, NDVI, FAI GeoTIFFs

> Status: Done | Epic: 02 | Last updated: 2026-04-14

## Description

Add Step 4 to `CDSE_OData_Workflow_v1.ipynb` so that any existing `_RRC.nc`
file can be processed into three GeoTIFF outputs:
1. Unmasked RGB composite (`rhorc_665 / rhorc_560 / rhorc_492`)
2. Masked NDVI (`rhorc_833`, `rhorc_665` — 7×7 median filtered; combined mask applied)
3. Masked FAI (`rhorc_833`, `rhorc_665`, `rhorc_1614` — same filter and mask)

## OpenSpec References
- Spec: `openspec/capabilities/rrc-post-processing/spec.md`
- Requirements: REQ-RRC-001, REQ-RRC-002, REQ-RRC-003, REQ-RRC-004, REQ-RRC-005, REQ-RRC-006, REQ-RRC-007
- Scenarios: SCENARIO-RRC-001 through SCENARIO-RRC-006

## Acceptance Criteria
- [x] `write_rgb_geotiff` writes unmasked 3-band float32 GeoTIFF (REQ-RRC-001, REQ-RRC-006)
- [x] `compute_indices` applies 7×7 median filter before index arithmetic (REQ-RRC-002)
- [x] NDVI pixels with non-positive red or NIR are NaN (REQ-RRC-003, SCENARIO-RRC-003)
- [x] FAI is negative at a sampled clear-ocean pixel in the test scene (REQ-RRC-004, SCENARIO-RRC-004)
- [x] Masked pixels are NaN in NDVI and FAI GeoTIFFs (REQ-RRC-005, SCENARIO-RRC-005)
- [x] GeoTIFF CRS matches `transverse_mercator.crs_wkt` in the source file (REQ-RRC-006, SCENARIO-RRC-006)
- [x] Output filenames follow the `_RGB.tif` / `_NDVI.tif` / `_FAI.tif` convention (REQ-RRC-007)

## Tasks
1. Add `_build_geotiff_profile`, `write_rgb_geotiff`, `compute_indices`, `write_index_geotiff` helper cells to notebook
2. Add Step 4 markdown, config, and execution cells
3. Verify against `S2A_MSI_2017_07_10_15_04_47_T22WDS_L2R_RRC.nc` (run Step 4, inspect outputs)

## Implementation Notes
- **Key files**: `CDSE_OData_Workflow_v1.ipynb`
- **Dependency fix**: recompiled `$JRE/lib/libiconv.2.dylib` to export `_locale_charset`
  (unblocks rasterio on macOS ARM64 with conda cdsetool environment)
- **Deviations**: None
