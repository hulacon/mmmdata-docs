---
title: Quality & Compliance
parent: Dataset Description
has_children: true
nav_order: 6
---

# BIDS Compliance Status

*Last checked against the dataset: 2026-08-20. Items are checked off only
where the check was actually run — see the note on each.*

## Complete

- [x] BIDS directory hierarchy (sub/ses/datatype)
- [x] File naming with proper BIDS entities
- [x] JSON sidecars for all NIfTI files
- [x] DWI .bval/.bvec files
- [x] Fieldmap AP/PA pairs for distortion correction
- [x] `dataset_description.json`
- [x] File inventory system — now the Contract A catalog at
      `inventory/catalog.duckdb` (the earlier `inventory/bids_file_inventory.tsv`
      no longer exists; `inventory/manifest.db` is superseded and unmaintained)
- [x] **Task relabeling**: Tasks use `TB*` (trial-based), `NAT*` (naturalistic),
      `FIN*` (final session), and `INIT*` (baseline) prefixes. Verified
      2026-08-20: no unprefixed `task-encoding` / `task-retrieval` /
      `task-math` / `task-resting` files remain anywhere in the dataset
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
- [x] **Stimulus registry**: canonical `stimulus_id` tables for all three
      stimulus sets at `stimuli/stimulus_registry/`, with every events-file
      reference verified to resolve

## In Progress

- [ ] **Behavioral data (ses-29)**: the out-of-scanner behavioral session is
      still not BIDS. sub-03 has a single legacy non-BIDS file
      (`ses-29/beh/Sub3Final Recall_word_timestamps.csv`); sub-04 and sub-05
      have no `ses-29/` directory at all. Transcription is done for all three
      subjects and a converter is drafted; the work is blocked on a manual
      listening/verification pass.
- [ ] **MRIQC group report**: individual reports available; no group-level
      report exists in `derivatives/mriqc/`.
- [ ] **QC review**: `derivatives/preprocessing_qc/` holds a decision record
      for all 609 BOLD runs across sub-03/04/05, but **every one is an
      auto-generated stub** — no run carries a human sign-off. See
      [Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md).
- [ ] **Derivative provenance**: not every derivative tree carries a
      `dataset_description.json` with meaningful `GeneratedBy`
      (`derivatives/stimuli_features/` currently has none).

## Needs Fixing

- [ ] **`README`**: still a 0-byte file at the BIDS root; needs a dataset
      description before any release
- [ ] **Cross-subject inconsistencies**: Fieldmap run-entity labeling differs
      across subjects in some sessions (e.g., ses-30)

## Open Questions

- Are the 4 DWI phase-encoding directions (AP/PA/LR/RL) intentional for all
  future subjects, or are LR/RL legacy acquisitions?
- Data sharing policy: what is public-facing vs restricted? (Deferred for
  now. Source data already lives outside the BIDS tree, which is the
  structural half of the answer — see [Source Data](sourcedata.md).)

---

## What this page does not claim

Per-subject and per-session completeness is **not** tracked here — it goes
stale silently. Query the catalog (`inventory/catalog.duckdb`, see
[Data Organization](data-organization.md)) for coverage questions.
