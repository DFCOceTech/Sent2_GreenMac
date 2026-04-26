# Epic 01: ACOLITE Atmospheric Correction

> Status: In Progress | Last updated: 2026-04-26

## Goal

Download Sentinel-2 L1C scenes and run ACOLITE to produce Rayleigh-corrected
reflectance (`rhorc`) with DSF residual glint correction.  Post-process the
L2R output: filter bands, compute masks, embed masks in `_RRC.nc`.

## Dependencies
- Depends on: nothing
- Blocks: Epic 02 (consumes `_RRC.nc`)

## Stories

| ID | Story | Status | OpenSpec Refs |
|----|-------|--------|---------------|
| S01-01 | Per-scene ACOLITE output directories | Done | REQ-ACOL-006 |

## Acceptance Criteria
- [x] ACOLITE produces `rhorc_*` bands with DSF glint correction
- [x] Band exclusion toggles work independently
- [x] Masks computed and embedded in `_RRC.nc`
- [x] Each scene written to `{acolite_out}/{sat}_{yyyymmdd}_{tile}/`
- [x] L2R post-processing correctly locates output via `find_l2r_files`
