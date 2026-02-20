---
title: Sourcedata
parent: Dataset Description
nav_order: 5
---

# Sourcedata

```
sourcedata/
├── archive/          # Archived materials (notes, pilot data, recruitment)
├── dicoms/           # Raw DICOMs (sub-01 through sub-05)
├── shared/           # Shared resources (experiment code, stimuli, scan logs, conversion configs)
├── sub-03/           # Per-subject organized sourcedata
│   ├── ses-01/ through ses-30/
│   │   ├── audio/        # Microphone recordings (.wav), voice memos (.m4a), transcriptions
│   │   ├── behavioral/   # PsychoPy output (.csv, .log, .psydat)
│   │   ├── dicom/        # Raw DICOM series directories
│   │   ├── eyetracking/  # Eye tracking data
│   │   └── other/        # Miscellaneous session files
├── sub-04/
├── sub-05/
└── rawto_be_bids/    # Legacy staging area (being migrated into per-subject structure)
    ├── alldata/
    └── to_be_bidsified/
```

- Pilot subjects (sub-01, sub-02) have DICOMs in `dicoms/` but should not be BIDSified.
- Per-subject sourcedata is organized by session with modality subdirectories
  (`audio/`, `behavioral/`, `dicom/`, `eyetracking/`, `other/`).
- `rawto_be_bids/` is a legacy staging area. Most files have been migrated into
  the per-subject structure; remaining contents are duplicates or documentation
  files.
- Long-term goal: eliminate the `rawto_be_bids` directory entirely and use
  scripts that know which source files need conversion.
