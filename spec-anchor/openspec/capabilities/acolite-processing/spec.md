# ACOLITE Processing — Specification

> Version: 1.1 | Status: Implemented | Last updated: 2026-04-26

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
The system SHALL locate the ACOLITE L2R output by searching the scene's
per-scene subdirectory (REQ-ACOL-006) first, with a recursive date+tile
glob fallback across all of `acolite_out` if the expected directory is absent.

### REQ-ACOL-005: Output file naming
The system SHALL write filtered output as `…_L2R_RRC.nc` and delete the
original `…_L2R.nc` after successful copy.

### REQ-ACOL-006: Per-scene output directory
The system SHALL write each scene's ACOLITE output to a dedicated subdirectory
of `acolite_out` named `{sat}_{yyyymmdd}_{tile}`, derived from the L1C `.SAFE`
product name, where `sat` ∈ {`S2A`, `S2B`}, `yyyymmdd` is the 8-digit
acquisition date, and `tile` is the UTM tile identifier (e.g. `T22VFN`).
The `output=` line in the ACOLITE settings file SHALL be set to this
per-scene path for each iteration of the processing loop.

## Acceptance Scenarios

### SCENARIO-ACOL-001: Default run strips rhot and rhos
**GIVEN** `exclude_rhot=True`, `exclude_rhos=True`, `exclude_rrc_edges=False`  
**WHEN** `nc_copy_filtered` runs  
**THEN** the output contains no `rhot_*` or `rhos_*` variables but retains all `rhorc_*`

### SCENARIO-ACOL-002: Edge-band exclusion
**GIVEN** `exclude_rrc_edges=True` and input is S2A  
**WHEN** `get_bands_to_exclude` runs  
**THEN** the returned list includes `rhorc_443`, `rhorc_704`, `rhorc_740`, `rhorc_783`, `rhorc_865`

### SCENARIO-ACOL-003: Per-scene subdirectory naming
**GIVEN** L1C product `S2A_MSIL1C_20240525T143001_N0510_R139_T22VFN_20240525T180921.SAFE`  
**WHEN** `scene_output_dir(acolite_out, l1c)` is called  
**THEN** the returned path is `{acolite_out}/S2A_20240525_T22VFN`

### SCENARIO-ACOL-004: Settings file uses per-scene output
**GIVEN** a loop over multiple L1C scenes  
**WHEN** the ACOLITE settings file is written for each scene  
**THEN** the `output=` line is `{acolite_out}/{sat}_{yyyymmdd}_{tile}` for that scene

## Implementation Status (2026-04-26)

**Status**: Implemented

### What's Built
- `scene_output_dir(acolite_out, l1c)` — parses sat/date/tile from SAFE name, returns per-scene path (`acolite-helpers-cc`)
- `find_l2r_files` updated to use `scene_output_dir` as primary search, recursive glob fallback (`acolite-helpers-cc`)
- `get_bands_to_exclude` inspects actual NetCDF variables (not hardcoded lists) (`acolite-helpers-cc`)
- `nc_copy_filtered` copies with exclusions and optionally embeds masks (`nc-copy-cc`)
- Config cell: `output=` removed from `set_str`; written per-scene in run cell (`acolite-config-cc`)
- Run cell: computes `scene_out` per iteration, writes `output={scene_out}` to settings file (`acolite-run-cc`)
- Post-processing cell: loops over `l1c_dirs`, uses `find_l2r_files` per scene (no flat glob) (`dd8af76b`)

### Deviations from Spec
- None

### Deferred
- None
