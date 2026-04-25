# Epic 02: RRC Post-Processing — Spectral Indices, GeoTIFF Export, and Patch Detection

> Status: Done | Last updated: 2026-04-25

## Goal

Transform ACOLITE `_RRC.nc` files into analysis-ready GeoTIFF products and
detect floating algae patches: an unmasked RGB composite, masked NDVI and FAI
rasters with median-filtered inputs, connected-components patch detection, and
a text report with patch centroids and areas.

## Dependencies
- Depends on: Epic 01 (produces the `_RRC.nc` files this epic consumes)
- Blocks: nothing yet

## Stories

| ID | Story | Status | OpenSpec Refs |
|----|-------|--------|---------------|
| S02-01 | RRC post-processing: RGB + NDVI + FAI GeoTIFFs | Done | REQ-RRC-001 through REQ-RRC-007 |
| S02-02 | FAI patch detection and report | Done | REQ-FAI-001 through REQ-FAI-006 |

## Acceptance Criteria
- [x] `write_rgb_geotiff` produces an unmasked 3-band GeoTIFF with correct CRS
- [x] `compute_indices` applies 7×7 median filter and returns NDVI + FAI arrays
- [x] NDVI NaN guard: pixels with non-positive red or NIR yield NaN
- [x] FAI is negative over clear open-ocean pixels
- [x] `write_index_geotiff` sets masked pixels to NaN in output GeoTIFFs
- [x] All three GeoTIFFs verified against test scene `T22WDS_L2R_RRC.nc`
- [x] `compute_fai_patches` thresholds masked FAI, labels components, filters by area
- [x] `write_fai_report` writes `%`-header + CSV with lon/lat centroids and areas
