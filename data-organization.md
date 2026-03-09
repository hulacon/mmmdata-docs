---
title: Data Organization
parent: Dataset Description
has_children: true
nav_order: 5
---

# Data Organization

MMMData follows the [Brain Imaging Data Structure (BIDS) v1.9.0](https://bids-specification.readthedocs.io/)
standard. Data are organized into three tiers:

- **BIDS raw**: Converted NIfTI + JSON + events TSV files
- **Source data**: Original DICOMs, PsychoPy output, audio recordings
- **Derivatives**: Preprocessed outputs (fMRIPrep, MRIQC)

## Per-Session Scan Inventories (`scans.tsv`)

Each subject/session directory contains a `sub-XX_ses-YY_scans.tsv` file
listing every primary NIfTI scan in that session. These are
[BIDS-standard scans files](https://bids-specification.readthedocs.io/en/stable/modality-agnostic-files.html#scans-file)
extended with custom columns:

| Column | Description |
|--------|-------------|
| `filename` | Relative path to the NIfTI file (BIDS required) |
| `acq_time` | Acquisition timestamp from JSON sidecar |
| `n_volumes` | Number of volumes (4th dimension; 1 for 3D scans) |
| `duration_s` | Scan duration in seconds |
| `n_events` | Row count of matching `_events.tsv` (n/a if none) |
| `physio_cardiac`, `physio_pulse`, `physio_respiratory` | Whether each physio channel exists |
| `eyetracking` | Whether eyetracking recording exists |
| `has_sbref`, `has_json`, `has_events_json` | Companion file presence |

Custom columns are documented in the BIDS-root `scans.json` sidecar (via
BIDS inheritance). Regenerate with:

```bash
.venv/bin/python3 scripts/build_scans_tsv.py
```

## Manifest Database

A SQLite database at `inventory/manifest.db` aggregates all scans.tsv files,
sourcedata metadata, derivative file listings, and session metadata into a
single queryable store. Key tables:

| Table | Contents |
|-------|----------|
| `files` | Every BIDS file with parsed entities (subject, session, task, run, suffix) |
| `nifti_meta` | NIfTI header info (dimensions, voxel size, TR) |
| `events_meta` | Events file stats (row count, columns, onset range) |
| `physio_meta` | Physio recording info (channel, sampling rate) |
| `sourcedata` | Raw data inventory (DICOMs, behavioral, audio, eyetracking) |
| `derivatives` | fMRIPrep and MRIQC output files |
| `session_metadata` | Per-session info from `_sessions.tsv` (date, notes, physiology) |
| `validation_results` | Automated check results (pass/fail/warn) |

The manifest is rebuilt on demand and is not version-controlled:

```bash
.venv/bin/python3 scripts/build_manifest.py          # full rebuild
.venv/bin/python3 scripts/build_manifest.py --skip-nifti  # fast (skip NIfTI headers)
```

## Validation

The dataset expectations schema (`dataset_expectations.toml`) defines what
*should* exist — expected tasks, run counts, volume counts, event
structures, and physio/eyetracking coverage. The validation engine compares
the manifest against these expectations:

```bash
.venv/bin/python3 -m validation.run                     # full validation
.venv/bin/python3 -m validation.run --checks file_presence  # specific check
.venv/bin/python3 -m validation.run --subjects sub-03       # specific subject
```

Known deviations (documented as `[[exceptions]]` in the schema) are
automatically matched and downgraded to informational status.
