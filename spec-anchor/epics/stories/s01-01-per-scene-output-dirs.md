# S01-01: Per-Scene ACOLITE Output Directories

> Status: Done | Epic: 01 | Last updated: 2026-04-26

## Description

Organise ACOLITE output by writing each scene into a dedicated subdirectory
`{sat}_{yyyymmdd}_{tile}` (e.g. `S2A_20240525_T22VFN`) rather than a flat
`acolite_out` root.  Also fixes a hardcoded product name in `acolite-run-cc`
and a dead `find_l2r_files` call in `dd8af76b` that was overridden by a
flat glob.

## OpenSpec References
- Spec: `openspec/capabilities/acolite-processing/spec.md`
- Requirements: REQ-ACOL-004 (updated), REQ-ACOL-006 (new)
- Scenarios: SCENARIO-ACOL-003, SCENARIO-ACOL-004

## Acceptance Criteria
- [x] `scene_output_dir(acolite_out, l1c)` returns `{acolite_out}/{sat}_{yyyymmdd}_{tile}` (REQ-ACOL-006, SCENARIO-ACOL-003)
- [x] `find_l2r_files` searches the per-scene subdir first, falls back to recursive glob (REQ-ACOL-004)
- [x] `output=` in ACOLITE settings file is per-scene, not the root `acolite_out` (SCENARIO-ACOL-004)
- [x] `acolite-run-cc` searches for `.SAFE` dirs, not a hardcoded product name
- [x] Post-processing cell (`dd8af76b`) loops `l1c_dirs` and uses `find_l2r_files` — flat glob removed
- [x] `acolite_out` in `secrets_config.yml` updated to root output directory

## Tasks
1. Add `scene_output_dir` helper; update `find_l2r_files` to use it
2. Remove `output=` from `set_str`; write `output={scene_out}` per-iteration in run cell
3. Fix `.SAFE` search string in `acolite-run-cc`
4. Fix `dd8af76b` to loop `l1c_dirs` and use `find_l2r_files` per scene
5. Update `secrets_config.yml` and `secrets_config.yml.example`

## Implementation Notes
- **Key files**: `CDSE_OData_Workflow_v1.ipynb`, `secrets_config.yml`
- Old `acolite_output_dir` helper replaced by `scene_output_dir` (different naming scheme)
