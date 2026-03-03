---
title: Quality & Compliance
parent: Dataset Description
nav_order: 6
---

# BIDS Compliance Status

## Complete

- [x] BIDS directory hierarchy (sub/ses/datatype)
- [x] File naming with proper BIDS entities
- [x] JSON sidecars for all NIfTI files
- [x] DWI .bval/.bvec files
- [x] Fieldmap AP/PA pairs for distortion correction
- [x] `dataset_description.json`
- [x] File inventory system (`inventory/bids_file_inventory.tsv`)
- [x] **Task relabeling**: Tasks use `TB*` (trial-based), `NAT*` (naturalistic),
      `FIN*` (final session), and `INIT*` (baseline) prefixes
- [x] **Events files**: `_events.tsv` + `_events.json` generated for all
      functional runs across sessions
- [x] **Fieldmap matching**: Uses `B0FieldIdentifier` / `B0FieldSource` sidecar
      approach (not `IntendedFor`)
- [x] **MRIQC**: Running for all subjects/sessions; HTML reports generated for
      structural and functional scans
- [x] **`participants.tsv`**: Correct subject IDs (sub-03/04/05) with full
      demographics (age, sex, handedness, vision, education, etc.)
- [x] **Phenotype data**: `phenotype/vviq.tsv` and `phenotype/final_debriefing.tsv`
      with JSON sidecar dictionaries
- [x] **Physiological recordings**: Scanner physio (`recording-cardiac`,
      `recording-pulse`, `recording-respiratory`) and eye-tracking
      (`recording-eye`) converted to BIDS `_physio.tsv.gz` format
- [x] **Behavioral sidecar files**: `_beh.tsv` + `_beh.json` in `beh/` directories

## In Progress

- [ ] **Behavioral data (ses-29)**: Out-of-scanner behavioral session not yet
      BIDSified for any subject
- [ ] **MRIQC group report**: Individual reports available; group-level
      aggregation pending

## Needs Fixing

- [ ] **`README`**: Empty file at BIDS root, needs dataset description
- [ ] **Cross-subject inconsistencies**: Fieldmap run-entity labeling differs
      across subjects in some sessions (e.g., ses-30)

## Open Questions

- Are the 4 DWI phase-encoding directions (AP/PA/LR/RL) intentional for all
  future subjects, or are LR/RL legacy acquisitions?
- Data sharing policy: what is public-facing vs restricted? (Deferred for now.)
