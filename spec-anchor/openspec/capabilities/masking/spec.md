# Masking — Specification

> Version: 1.1 | Status: Implemented | Last updated: 2026-04-26

## Purpose

Compute and apply cloud, land, and ice/bright-pixel masks to ACOLITE L2R
NetCDF output.  Masks are combined and optionally dilated, then embedded in
the output file or saved separately.

## Functional Requirements

### REQ-MASK-001: Cloud mask
The system SHALL compute a cloud mask by thresholding TOA reflectance at the
cirrus band.  ACOLITE names the band from the sensor metadata centre wavelength
truncated to integer: `rhot_1373` for S2A (centre 1373.5 nm) and
`rhot_1377` for S2B (centre ~1376.9 nm).  Primary and fallback band names are
read from `mask_config.yml`.

### REQ-MASK-002: Land mask
The system SHALL compute a land mask by thresholding Rayleigh-corrected SWIR
reflectance: `rhorc_1614` (S2A B11) or `rhorc_1610` (S2B B11).  Primary and
fallback band names are read from `mask_config.yml`.

### REQ-MASK-003: Ice / bright-pixel mask
The system SHALL compute an ice/bright mask by thresholding Rayleigh-
corrected red-band reflectance (`rhorc_665`), with `rhos_665` as a fallback.

### REQ-MASK-004: Configurable thresholds
The system SHALL read all mask thresholds and band names from an external
YAML configuration file (`mask_config.yml`), not hardcoded in the notebook.

### REQ-MASK-005: Combined mask
The system SHALL combine individual masks with a logical OR to produce a
`mask_combined` array.

### REQ-MASK-006: Optional dilation
The system SHALL provide an option to dilate the combined mask boundary by N
pixels using a square (2N+1 × 2N+1) binary structuring element.  N and the
on/off toggle are read from `mask_config.yml` (`dilation.pixels`,
`dilation.apply`).

### REQ-MASK-007: Mask storage options
The system SHALL provide two independently togglable output options:
- `save_masks_in_rrc` — embed all four mask variables in the `_RRC.nc` file
- `save_masks_separate` — write a companion `_mask.nc` containing only the
  mask variables and spatial coordinates

### REQ-MASK-008: Resolution mismatch handling
The system SHALL handle the case where individual masks have different spatial
resolutions (e.g., 1375 nm at 60 m vs. 10 m main bands) by resampling with
nearest-neighbour interpolation before combining.

## Acceptance Scenarios

### SCENARIO-MASK-001: Cloud mask S2B fallback
**GIVEN** an S2B L2R file where `rhot_1374` (S2A primary) is absent but `rhot_1377` is present  
**WHEN** `compute_cloud_mask` runs  
**THEN** `rhot_1377` is used and a WARNING is printed; no exception is raised

### SCENARIO-MASK-002: All masks absent
**GIVEN** none of the configured bands are in the L2R file  
**WHEN** `combine_and_dilate` runs  
**THEN** `None` is returned (no crash)

### SCENARIO-MASK-003: Dilation applied
**GIVEN** `dilation.apply=true` and `dilation.pixels=3` in `mask_config.yml`  
**WHEN** `combine_and_dilate` runs  
**THEN** the combined mask is dilated with a 7×7 structuring element

### SCENARIO-MASK-004: Separate mask file
**GIVEN** `save_masks_separate=True`  
**WHEN** `nc_copy_filtered` runs  
**THEN** a `_mask.nc` file is written containing only mask variables and coordinates

## Implementation Status (2026-04-11)

**Status**: Implemented

### What's Built
- `load_mask_config`, `_read_band`, `_threshold_band` helpers (`mask-helpers-cc`)
- `compute_cloud_mask`, `compute_land_mask`, `compute_ice_bright_mask` (`mask-helpers-cc`)
- `_resize_to` for resolution mismatch (`mask-helpers-cc`)
- `combine_and_dilate` with scipy `binary_dilation` (`mask-helpers-cc`)
- `compute_masks` orchestrator (`mask-helpers-cc`)
- `_write_mask_vars`, `nc_copy_filtered` with `save_masks_in_rrc` / `save_masks_separate` (`nc-copy-cc`)
- `mask_config.yml` with documented default thresholds

### Deviations from Spec
- REQ-MASK-001/002 updated 2026-04-26 to reflect S2A/S2B band differences (bug fix)

### Deferred
- None
