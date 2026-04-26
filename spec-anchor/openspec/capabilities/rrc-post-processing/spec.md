# RRC Post-Processing — Specification

> Version: 1.1 | Status: Done | Last updated: 2026-04-26

## Purpose

Transform an ACOLITE `_RRC.nc` file into analysis-ready GeoTIFF outputs:
an unmasked RGB composite, and masked spectral-index products (NDVI, FAI).
A 7×7 median filter is applied to all `rhorc_*` bands before index computation
to suppress salt-and-pepper noise from whitecaps and specular glint residuals.

## Functional Requirements

### REQ-RRC-001: Unmasked RGB GeoTIFF
The system SHALL write a 3-band float32 GeoTIFF from `rhorc_665` (red),
`rhorc_492` (blue), and the green band auto-detected from the filename:
`rhorc_560` for S2A, `rhorc_559` for S2B.  No mask is applied.

### REQ-RRC-002: 7×7 median filter
The system SHALL apply a 7×7 median filter (scipy `median_filter`, `size=7`)
to every `rhorc_*` band loaded for index computation, before any arithmetic.

### REQ-RRC-003: NDVI computation
The system SHALL compute NDVI as `(rhorc_833 - rhorc_665) / (rhorc_833 + rhorc_665)`.
Pixels where either band is ≤ 0 SHALL yield NaN (nodata guard).

### REQ-RRC-004: FAI computation
The system SHALL compute the Floating Algae Index (Hu 2009) as:

    FAI = rhorc_833 − [rhorc_665 + (rhorc_SWIR − rhorc_665) × (833−665)/(wl_SWIR−665)]

where SWIR band and wavelength are auto-detected from the filename:
`rhorc_1614` / 1614 nm for S2A; `rhorc_1610` / 1610 nm for S2B.
`rhorc_665`, `rhorc_833`, and the SWIR band are the median-filtered arrays.

### REQ-RRC-005: Combined mask applied to indices
The system SHALL apply `mask_combined` from the RRC file to NDVI and FAI
outputs by setting all masked pixels to NaN before writing.

### REQ-RRC-006: Georeferenced output
All GeoTIFF outputs SHALL carry the CRS and affine transform derived from the
`transverse_mercator` grid_mapping variable and the `x`/`y` coordinate arrays
in the RRC file.  No hardcoded EPSG codes.

### REQ-RRC-007: Output naming
Output filenames SHALL be derived from the input `_RRC.nc` basename:

| Output | Filename suffix |
|--------|-----------------|
| RGB    | `_RGB.tif`      |
| NDVI   | `_NDVI.tif`     |
| FAI    | `_FAI.tif`      |

### REQ-RRC-008: SWIR-based land mask supplement
`compute_indices` SHALL compute a land mask by thresholding the unfiltered
SWIR band (`rhorc_1614` for S2A, `rhorc_1610` for S2B) at
`swir_land_threshold` (default 0.008).  This mask SHALL be OR-combined with
the `mask_combined` variable read from the RRC file before any indices or
GeoTIFFs are written.  The SWIR mask is computed on the raw (pre-filter) SWIR
to preserve sharp land/water boundaries.  The threshold is exposed as a
configurable parameter in the Step 4 config cell.

Note: In ACOLITE rhorc, the aerosol residual dominates both SWIR and NIR over
dark ocean, causing water rhorc_SWIR to cluster around 0.004–0.009.  Dark
rocky Arctic land (e.g. Greenland tundra) has rhorc_SWIR ≈ 0.006–0.035.  A
threshold of 0.008 removes the bulk of land pixels while only flagging the
most turbid coastal water (which has near-zero FAI anyway and cannot support
macroalgae detection).  The remaining land pixels are handled by the ocean
context filter in `compute_fai_patches` (REQ-FAI-007).

## Acceptance Scenarios

### SCENARIO-RRC-001: RGB GeoTIFF is unmasked
**GIVEN** an RRC file where `mask_combined` flags some pixels  
**WHEN** `write_rgb_geotiff` runs  
**THEN** the output GeoTIFF contains valid float values at masked pixel locations
(the RGB is intentionally unmasked)

### SCENARIO-RRC-002: Median filter reduces extremes
**GIVEN** an RRC file with whitecap speckle  
**WHEN** `compute_indices` runs with `filter_size=7`  
**THEN** NDVI and FAI arrays have fewer extreme outlier values than
unfiltered equivalents (verified by max-abs comparison or visual inspection)

### SCENARIO-RRC-003: NDVI NaN guard
**GIVEN** a pixel where `rhorc_665 <= 0` or `rhorc_833 <= 0`  
**WHEN** NDVI is computed  
**THEN** that pixel is NaN in the output

### SCENARIO-RRC-004: FAI negative over open ocean
**GIVEN** a clear-water pixel over open ocean  
**WHEN** FAI is computed  
**THEN** the FAI value is negative (NIR below the red–SWIR baseline)

### SCENARIO-RRC-005: Masked pixels are NaN in index GeoTIFFs
**GIVEN** a pixel flagged in `mask_combined`  
**WHEN** `write_index_geotiff` runs  
**THEN** that pixel is NaN in both the NDVI and FAI GeoTIFFs

### SCENARIO-RRC-006: CRS matches source NetCDF
**GIVEN** an RRC file with UTM Zone 22N CRS  
**WHEN** any GeoTIFF is written  
**THEN** `rasterio.open(out_path).crs.wkt` matches the `crs_wkt` attribute of
the `transverse_mercator` variable in the source NetCDF

### SCENARIO-RRC-007: SWIR land mask excludes land from FAI patches
**GIVEN** an RRC file where `mask_combined` contains no land component  
**WHEN** `compute_indices` runs with default `swir_land_threshold=0.035`  
**THEN** `mask_combined` in the returned dict has land pixels set to True,
and no FAI patches in the report are located over land

## Implementation Status (2026-04-26)

**Status**: Done

### What's Built
- `_build_geotiff_profile` — derives CRS + affine transform from RRC NetCDF
- `write_rgb_geotiff` — 3-band unmasked RGB GeoTIFF; S2A/S2B green auto-detected
- `compute_indices` — median filter → NDVI + FAI; SWIR land mask OR-combined with embedded mask_combined (REQ-RRC-008)
- `write_index_geotiff` — masked single-band float32 GeoTIFF
- Step 4 config cell: exposes `swir_land_threshold` (default 0.035)
- Dependency fix: recompiled `$JRE/lib/libiconv.2.dylib` wrapper to also
  export `_locale_charset`, unblocking rasterio/libspatialite on macOS ARM64

### Deviations from Spec
- None

### Deferred
- None
