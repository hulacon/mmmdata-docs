---
title: Compliance Status
parent: Dataset Description
nav_order: 8
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

## In Progress

- [ ] **Task relabeling**: encoding/retrieval/math/resting → TB*/NAT* prefixes
- [ ] **Events files**: Task timing data exists in sourcedata but has not been
      converted to BIDS `_events.tsv` format (blocked on task relabeling)
- [ ] **Behavioral data (ses-29)**: Needs BIDSification
- [ ] **Fieldmap `IntendedFor`**: Sidecar mappings need updating after task relabeling
- [ ] **MRIQC**: Needs to be run for all subjects/sessions

## Needs Fixing

- [ ] **`participants.tsv`**: Contains wrong subject IDs (lists sub-01/02/03
      instead of sub-03/04/05). Demographics TBD.
- [ ] **`README`**: Empty file, needs dataset description
- [ ] **Cross-subject inconsistencies**: Fieldmap run-entity labeling differs
      across subjects in some sessions (e.g., ses-30)

## Open Questions

- Are the 4 DWI phase-encoding directions (AP/PA/LR/RL) intentional for all
  future subjects, or are LR/RL legacy acquisitions?
- What are the final task IDs for session 30 memory tests?
- Data sharing policy: what is public-facing vs restricted? (Deferred for now.)
