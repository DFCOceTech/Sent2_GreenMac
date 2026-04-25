# S02-02: FAI Patch Detection and Report

> Status: Done | Epic: 02 | Last updated: 2026-04-25

## Description

Extend Step 4 of `CDSE_OData_Workflow_v1.ipynb` with FAI connected-components
analysis: threshold the masked FAI field, label connected components, filter
by minimum area, and write a text report with patch centroids (lon/lat) and
pixel areas.  Ported from `FAI_connected_components.ipynb`.

## OpenSpec References
- Spec: `openspec/capabilities/fai-patches/spec.md`
- Requirements: REQ-FAI-001 through REQ-FAI-006
- Scenarios: SCENARIO-FAI-001 through SCENARIO-FAI-005

## Acceptance Criteria
- [x] `compute_fai_patches` masks FAI using `mask_combined`, thresholds at `fai_thrsh`, labels with 4-connectivity, filters area ≤ `ar_thrsh` (REQ-FAI-001, REQ-FAI-002, REQ-FAI-003, SCENARIO-FAI-002, SCENARIO-FAI-003)
- [x] Patch centroids geocoded via `lon`/`lat` arrays from RRC NetCDF (REQ-FAI-004)
- [x] `write_fai_report` writes `%`-header then CSV `num,longitude,latitude,area` (REQ-FAI-005, SCENARIO-FAI-004)
- [x] Report filename follows `_FAI_report.txt` convention (REQ-FAI-006)
- [x] Both functions called from Step 4 after GeoTIFF outputs (REQ-FAI-006)

## Tasks
1. Add `compute_fai_patches` and `write_fai_report` to the RRC analysis helpers cell
2. Add `fai_thrsh`, `ar_thrsh` to Step 4 config cell
3. Append patch detection and report writing to Step 4 execution cell
4. Add `skimage.measure` import and `from datetime import datetime` to imports cell

## Implementation Notes
- **Key files**: `CDSE_OData_Workflow_v1.ipynb`
- **Source**: ported from `FAI_connected_components.ipynb`
- **Deviations**: uses `mask_combined` from RRC file (not re-computed); uses WKT CRS not proj4
