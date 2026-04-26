# S03-01: Secrets and Local Config File

> Status: Done | Epic: 03 | Last updated: 2026-04-26

## Description

Move all credentials and absolute paths out of the committed notebook into
an untracked `secrets_config.yml`.  Add `.gitignore`.  Commit a
`secrets_config.yml.example` template.

## OpenSpec References
- Spec: `openspec/capabilities/secrets-config/spec.md`
- Requirements: REQ-SEC-001 through REQ-SEC-006
- Scenarios: SCENARIO-SEC-001 through SCENARIO-SEC-003

## Acceptance Criteria
- [x] `secrets_config.yml` is in `.gitignore` and not committed (REQ-SEC-001)
- [x] `secrets_config.yml.example` with placeholders is committed (REQ-SEC-002)
- [x] Schema has `cdse`, `earthdata`, `paths` keys (REQ-SEC-003)
- [x] Single `secrets-load-cc` cell loads file and assigns all variables (REQ-SEC-004)
- [x] `config-cc` and `acolite-config-cc` contain no literal credentials or `/Users/` paths (REQ-SEC-005)
- [x] `rrc_path` remains a notebook variable (REQ-SEC-006)

## Tasks
1. Write `openspec/capabilities/secrets-config/spec.md`
2. Create `secrets_config.yml` (not committed)
3. Create `secrets_config.yml.example` (committed)
4. Create `.gitignore`
5. Add `secrets-load-cc` cell to notebook (after imports)
6. Strip secrets from `config-cc` and `acolite-config-cc`
7. Update ops files

## Implementation Notes
- **Key files**: `CDSE_OData_Workflow_v1.ipynb`, `secrets_config.yml`, `.gitignore`
- `rrc_path` is session-specific (changes per processing run) so stays in the notebook
