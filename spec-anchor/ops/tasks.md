# Active Tasks

> **How to use this file**
>
> One story at a time. At session start: copy the story's task list here and expand into
> concrete sub-tasks. During work: update status here — do not edit the story file mid-session.
> At session end: run the reconciliation checklist at the bottom, then clear this file
> (replace with the next story's tasks, or leave the header with "No active story").
>
> Status markers: `[ ]` todo · `[~]` in progress · `[x]` done · `[!]` blocked

---

## Story: S02-02 — FAI Patch Detection and Report

> Story file: `epics/stories/s02-02-fai-patches.md`
> Spec: `openspec/capabilities/fai-patches/spec.md`
> Requirements in scope: REQ-FAI-001 through REQ-FAI-006
> Started: 2026-04-25

### Acceptance Criteria (from story file)
- [x] `compute_fai_patches` masks FAI, thresholds at `fai_thrsh`, labels with 4-connectivity, filters area ≤ `ar_thrsh`
- [x] Patch centroids geocoded via `lon`/`lat` arrays from RRC NetCDF
- [x] `write_fai_report` writes `%`-header then CSV `num,longitude,latitude,area`
- [x] Report filename follows `_FAI_report.txt` convention
- [x] Both functions called from Step 4 after GeoTIFF outputs
- [ ] Verify against a known scene (check patch count and centroid accuracy)

### Tasks

#### Implement helper functions
- [x] `compute_fai_patches(fai, mask_combined, fai_thrsh, ar_thrsh)` in RRC helpers cell
- [x] `write_fai_report(patches, rrc_path, out_path, fai_thrsh, ar_thrsh)` in RRC helpers cell
- [x] Add `skimage.measure` import to imports cell
- [x] Add `from datetime import datetime` to imports cell

#### Extend Step 4
- [x] Add `fai_thrsh`, `ar_thrsh` to Step 4 config cell
- [x] Append patch detection + `write_fai_report` to Step 4 execution cell
- [x] Update Step 4 markdown to mention FAI patch report output

#### Spec and ops
- [x] Create `openspec/capabilities/fai-patches/spec.md`
- [x] Create `epics/stories/s02-02-fai-patches.md`
- [x] Update `epics/epic-02-rrc-analysis.md`
- [x] Update `ops/status.md` and `ops/changelog.md`

#### Verification (needs RRC file)
- [ ] Run Step 4 on a known scene; confirm patch count and spot-check centroid lat/lon

### Decisions & Deviations
| Decision | Rationale | Spec impact |
|----------|-----------|-------------|
| Use `mask_combined` from RRC file | Already computed in Step 3; avoids duplicate mask logic | Documented in spec deviations |
| No creator metadata block in report | Simplified; user can add via notebook if needed | Noted in spec |

---

### Session-End Reconciliation Checklist
- [x] Acceptance criteria checked off (pending verification)
- [x] `openspec/capabilities/fai-patches/spec.md` — Status = Done
- [x] `epics/stories/s02-02-fai-patches.md` — Status = Done, ACs checked
- [x] `epics/epic-02-rrc-analysis.md` — S02-02 row added, status = Done
- [ ] `_bmad/traceability.md` — new FR rows added (REQ-FAI-*)
- [x] `ops/status.md` — updated
- [x] `ops/changelog.md` — entry added
- [ ] This file cleared after verification run
