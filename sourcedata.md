---
title: Source Data
parent: Data Organization
grand_parent: Dataset Description
nav_order: 2
---

# Sourcedata

Source data lives **outside the BIDS tree**, in a sibling directory of the
BIDS root: `/gpfs/projects/hulacon/shared/mmmsourcedata/`. It was moved out
of the dataset so that the BIDS project directory can be shared without
exposing PII — everything identifying (raw DICOMs, audio recordings,
recruitment materials) stays in `mmmsourcedata/`. Doc references to
`sourcedata/` paths are relative to this root.

```
mmmsourcedata/
├── archive/          # Archived materials (notes, pilot data, recruitment, legacy staging)
│   ├── notes/
│   ├── pilot/            # Pilot subjects (sub-01, sub-02): DICOMs only, not BIDSified
│   ├── rawto_be_bids/    # Legacy staging area (migrated; kept for reference)
│   └── recruitment/
├── shared/           # Shared resources
│   ├── conversion/       # BIDS conversion configs, event file generators, file inventory
│   ├── experiment_code/  # Task implementations (localizer, cued/free recall, final session)
│   ├── scan_logs/        # Scanning session logs
│   └── stimuli/          # Stimulus preparation metadata
├── sub-03/           # Per-subject organized sourcedata
│   ├── ses-01/ through ses-30/
│   │   ├── audio/        # Microphone recordings (.wav), voice memos (.m4a), transcriptions
│   │   ├── behavioral/   # PsychoPy output (.csv, .log, .psydat)
│   │   ├── dicom/        # Raw DICOM series directories
│   │   ├── eyetracking/  # EyeLink EDF files
│   │   └── other/        # Miscellaneous session files
├── sub-04/
├── sub-05/
├── sub-06/           # Data collection in progress (not yet BIDSified)
├── sub-07/           # Data collection in progress (not yet BIDSified)
└── sub-08/           # Data collection in progress (not yet BIDSified)
```

- Pilot subjects (sub-01, sub-02) have DICOMs in `archive/pilot/` but are not BIDSified.
- Per-subject sourcedata is organized by session with modality subdirectories
  (`audio/`, `behavioral/`, `dicom/`, `eyetracking/`, `other/`).
- The legacy `rawto_be_bids/` staging area has been moved to `archive/`.
  All active source files now live in the per-subject structure.
- `shared/conversion/` contains the file inventory (`file_inventory.csv`) and
  event file generation scripts used by the automated BIDS conversion pipeline.
