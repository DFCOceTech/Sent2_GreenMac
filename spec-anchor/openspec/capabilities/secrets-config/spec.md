# Secrets and Local Config — Specification

> Version: 1.0 | Status: Done | Last updated: 2026-04-26

## Purpose

Move all machine-local and sensitive values (credentials, absolute paths) out
of the committed notebook into an untracked `secrets_config.yml` file.  The
notebook loads this file at runtime; `secrets_config.yml.example` (committed)
serves as a template.  A `.gitignore` entry ensures the real file is never
accidentally committed.

## Functional Requirements

### REQ-SEC-001: secrets_config.yml not committed
`secrets_config.yml` SHALL be listed in `.gitignore` so it is never staged or
pushed to the remote repository.

### REQ-SEC-002: secrets_config.yml.example committed
A `secrets_config.yml.example` file with placeholder values SHALL be committed
to the repository so new users know the expected schema.

### REQ-SEC-003: Schema
`secrets_config.yml` SHALL have four top-level keys:

| Key | Sub-keys | Content |
|-----|----------|---------|
| `cdse` | `username`, `password` | Copernicus Data Space Ecosystem login |
| `earthdata` | `username`, `password` | NASA Earthdata login (for ACOLITE ancillary) |
| `paths` | `download`, `acolite_dir`, `acolite_out`, `mask_config` | Absolute paths for this machine |

### REQ-SEC-004: Single load cell in notebook
The notebook SHALL contain exactly one `secrets-load-cc` cell (placed
immediately after the imports cell) that opens `secrets_config.yml` relative
to the notebook's working directory and assigns credentials and paths to
module-level variables.

### REQ-SEC-005: No hardcoded secrets in notebook
After loading from secrets, the `config-cc` and `acolite-config-cc` cells
SHALL NOT contain any literal credential strings or absolute paths.  They
reference only the variables assigned in `secrets-load-cc`.

### REQ-SEC-006: Step 4 rrc_path remains editable
`rrc_path` (Step 4 config cell) is session-specific and SHALL remain a
notebook variable the user edits.  It does not appear in `secrets_config.yml`.

## Acceptance Scenarios

### SCENARIO-SEC-001: No secrets in git diff
**GIVEN** a notebook with `secrets-load-cc` and a real `secrets_config.yml`  
**WHEN** `git diff HEAD` is inspected  
**THEN** no username, password, or `/Users/` path appears in the diff

### SCENARIO-SEC-002: Notebook runs after loading secrets
**GIVEN** a valid `secrets_config.yml`  
**WHEN** cells are run top-to-bottom  
**THEN** `username`, `password`, `download_path`, `acolite_dir`, `acolite_out`,
`earthdata_user`, `earthdata_pass`, `mask_config_path` are all defined

### SCENARIO-SEC-003: Missing secrets file raises clear error
**GIVEN** `secrets_config.yml` is absent  
**WHEN** `secrets-load-cc` runs  
**THEN** Python raises `FileNotFoundError` with the resolved path in the message

## Implementation Status (2026-04-26)

**Status**: Done

### What's Built
- `secrets_config.yml` — machine-local secrets (not in git)
- `secrets_config.yml.example` — committed template with placeholder values
- `.gitignore` — excludes `secrets_config.yml`, `.DS_Store`, `.ipynb_checkpoints/`
- `secrets-load-cc` notebook cell: loads YAML and assigns all credentials/paths
- `config-cc` stripped of `username`, `password`, `download_path`
- `acolite-config-cc` stripped of `earthdata_user`, `earthdata_pass`, `acolite_dir`,
  `acolite_out`, `mask_config_path`

### Deviations from Spec
- None
