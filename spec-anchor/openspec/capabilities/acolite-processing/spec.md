# ACOLITE Processing — Specification

> Version: 1.0 | Status: Implemented | Last updated: 2026-04-11

## Purpose

Run ACOLITE atmospheric correction on downloaded Sentinel-2 L1C `.SAFE`
directories to produce Rayleigh-corrected reflectance (Rrc, `rhorc`) with
DSF residual sunglint correction, then filter the output to reduce file size.

## Functional Requirements

### REQ-ACOL-001: Rrc output
The system SHALL configure ACOLITE with `output_rhorc=True` to produce
Rayleigh-corrected reflectance (`rhorc_*`) bands.

### REQ-ACOL-002: Glint correction
The system SHALL configure ACOLITE with `dsf_residual_glint_correction=True`
and `dsf_residual_glint_correction_method=default`.

### REQ-ACOL-003: Independent band exclusion toggles
The system SHALL provide three independently configurable boolean flags:
- `exclude_rhot` — strip all `rhot_*` (TOA reflectance) bands
- `exclude_rhos` — strip all `rhos_*` (DSF surface reflectance) bands
- `exclude_rrc_edges` — strip `rhorc_*` at near-UV / red-edge wavelengths
  (S2A: 443, 704, 740, 783, 865 nm; S2B: 442, 704, 739, 780, 864 nm)

### REQ-ACOL-004: Automatic L2R file discovery
The system SHALL locate the ACOLITE L2R output by matching the scene's
product basename (without `.SAFE`) to an ACOLITE output subdirectory,
with a date+tile glob fallback if the expected directory is absent.

### REQ-ACOL-005: Output file naming
The system SHALL write filtered output as `…_L2R_RRC.nc` and delete the
original `…_L2R.nc` after successful copy.

## Acceptance Scenarios

### SCENARIO-ACOL-001: Default run strips rhot and rhos
**GIVEN** `exclude_rhot=True`, `exclude_rhos=True`, `exclude_rrc_edges=False`  
**WHEN** `nc_copy_filtered` runs  
**THEN** the output contains no `rhot_*` or `rhos_*` variables but retains all `rhorc_*`

### SCENARIO-ACOL-002: Edge-band exclusion
**GIVEN** `exclude_rrc_edges=True` and input is S2A  
**WHEN** `get_bands_to_exclude` runs  
**THEN** the returned list includes `rhorc_443`, `rhorc_704`, `rhorc_740`, `rhorc_783`, `rhorc_865`

## Implementation Status (2026-04-11)

**Status**: Implemented

### What's Built
- `get_bands_to_exclude` inspects actual NetCDF variables (not hardcoded lists) (`acolite-helpers-cc`)
- `find_l2r_files` with primary + fallback glob search (`acolite-helpers-cc`)
- `nc_copy_filtered` copies with exclusions and optionally embeds masks (`nc-copy-cc`)
- Config cell with three exclusion toggles (`acolite-config-cc`)
- Run cell orchestrates ACOLITE + post-processing per scene (`acolite-run-cc`)

### Deviations from Spec
- None

### Deferred
- None
