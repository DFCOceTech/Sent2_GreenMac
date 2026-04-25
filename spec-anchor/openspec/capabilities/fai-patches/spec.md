# FAI Patch Detection — Specification

> Version: 1.0 | Status: Done | Last updated: 2026-04-25

## Purpose

Detect floating algae patches in Sentinel-2 scenes by thresholding the FAI
(Floating Algae Index) computed in Step 4, running connected-components
labeling, filtering by minimum patch area, and writing a text report with
patch centroids (lon/lat) and pixel areas.

Derived from the `FAI_connected_components.ipynb` batch workflow; adapted
to operate on single `_RRC.nc` scenes within the CDSE_OData workflow and
to use the `mask_combined` already embedded by Step 3 rather than
re-computing masks.

## Functional Requirements

### REQ-FAI-001: FAI binary image
The system SHALL produce a binary array from the masked FAI field: pixels
with FAI ≥ `fai_thrsh` are True; NaN pixels (masked or invalid) are False.

### REQ-FAI-002: Connected components
The system SHALL label the binary image using 4-connectivity
(`skimage.measure.label`, `connectivity=1`).

### REQ-FAI-003: Area filter
The system SHALL discard components whose pixel area is ≤ `ar_thrsh`.
Only components with area > `ar_thrsh` are returned.

### REQ-FAI-004: Centroid coordinates
The system SHALL look up the geographic centroid of each surviving component
from the `lon` and `lat` 2-D arrays in the RRC NetCDF, using the
rounded centroid pixel indices from `regionprops`.

### REQ-FAI-005: Text report
The system SHALL write a text file whose first block consists of `%`-prefixed
metadata lines (creation timestamp, file analyzed, FAI threshold, area
threshold), followed by a CSV section with header `num,longitude,latitude,area`
and one row per surviving patch.

### REQ-FAI-006: Output naming and integration
Output filename SHALL be derived from the `_RRC.nc` basename by replacing
`_RRC.nc` with `_FAI_report.txt`.  The functions SHALL be called from Step 4
in `CDSE_OData_Workflow_v1.ipynb`, after the GeoTIFF outputs.

## Acceptance Scenarios

### SCENARIO-FAI-001: FAI threshold detection
**GIVEN** a masked FAI array with known positive values  
**WHEN** `compute_fai_patches` runs with `fai_thrsh=0.002`  
**THEN** pixels with FAI ≥ 0.002 appear in the labeled output

### SCENARIO-FAI-002: Masked pixels excluded
**GIVEN** pixels flagged in `mask_combined`  
**WHEN** `compute_fai_patches` runs  
**THEN** those pixels are NaN before thresholding and are not detected as patches

### SCENARIO-FAI-003: Small patches filtered
**GIVEN** connected components with area ≤ `ar_thrsh`  
**WHEN** `compute_fai_patches` runs  
**THEN** those components are absent from the returned patch list

### SCENARIO-FAI-004: Report CSV format
**GIVEN** N surviving patches  
**WHEN** `write_fai_report` runs  
**THEN** the output file has N data rows, each formatted as `num,lon,lat,area`

### SCENARIO-FAI-005: No patches
**GIVEN** a scene where no FAI pixel ≥ `fai_thrsh`  
**WHEN** `compute_fai_patches` runs  
**THEN** an empty list is returned; `write_fai_report` writes the header only (0 data rows)

## Implementation Status (2026-04-25)

**Status**: Done

### What's Built
- `compute_fai_patches(fai, mask_combined, fai_thrsh, ar_thrsh)` — mask, threshold, label, filter
- `write_fai_report(patches, rrc_path, out_path, fai_thrsh, ar_thrsh)` — write text report
- Step 4 config: `fai_thrsh`, `ar_thrsh` variables in config cell
- Step 4 execution: patch detection + report writing appended after GeoTIFF outputs

### Deviations from Old Workflow
- Mask source: uses `mask_combined` from RRC NetCDF (Step 3) rather than
  re-computing cloud/bright masks — avoids duplicate logic
- GeoTIFF georeferencing: uses `_build_geotiff_profile` (CRS from WKT, no hardcoded EPSG),
  replacing the old `get_rng_prj` proj4-string approach
- Report metadata: simplified header (no creator affiliation block); fields
  can be extended via the notebook config cell

### Deferred
- None
