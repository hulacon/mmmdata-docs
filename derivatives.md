---
title: Derivatives & Preprocessing
parent: Data Organization
grand_parent: Dataset Description
nav_order: 3
---

# Derivatives & Preprocessing

```
derivatives/
├── mriqc/        # MRIQC output (currently empty — not yet run)
├── fmriprep/     # fMRIPrep preprocessed data (all sessions)
├── fmriprepFR/   # Earlier fMRIPrep run (subset of sessions + Psychopy output)
└── QA/           # Legacy QA images (likely superseded by MRIQC)
```

## Target Pipeline

```
sourcedata (DICOMs, raw behavioral)
    │
    ▼
BIDS raw (NIfTI + JSON + events TSV)
    │
    ├──▶ MRIQC (quality metrics, outlier detection)
    │
    ├──▶ fMRIPrep (preprocessing: registration, distortion correction, confounds)
    │
    └──▶ [downstream analysis stages TBD]
```

- MRIQC and fMRIPrep can likely run in parallel, producing complementary QA output.
- Long-term goal: a single `fmriprep/` output directory (the current `fmriprepFR/`
  and `QA/` directories are legacy and will be deprecated).

## Minimally Preprocessed Version

_Documentation forthcoming._

## GLMSingle Output

_Documentation forthcoming._
