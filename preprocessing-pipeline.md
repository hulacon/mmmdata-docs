---
title: Analysis-Ready Preprocessing Pipeline
parent: Data Organization
grand_parent: Dataset Description
nav_order: 4
---

# Analysis-Ready Preprocessing Pipeline

## Overview

A two-layer pipeline that takes fMRIPrep + NORDIC (?) outputs and produces
analysis-ready data for three distinct analysis types, all sharing a single
quality-control layer.

```
Layer 1 — Shared base (fMRIPrep outputs + human QC decisions)
    ↓
Layer 2 — Stream-specific cleaning (applied on output from layer 1)
    ├── ready/glmsingle/      ← TB sessions + block-design localizers
    ├── ready/naturalistic/   ← NAT sessions + pRF localizer
    └── ready/connectivity/   ← resting-state sessions
```

**Key design principles:**
- QC decisions (run exclusions, bad TRs, NORDIC choice) are made **once** at
  layer 1 and propagated to all streams. Comparisons across analysis types are
  valid because the QC substrate is identical.
- The BOLD timeseries is **never modified** for the GLMSingle stream — only a
  curated confounds file and TR exclusion flags are produced. GLMSingle handles
  its own noise modeling internally.
- Bandpass filtering is **stream-specific**: required for connectivity ([Cordes et al., 2001](https://pubmed.ncbi.nlm.nih.gov/11498421/)), harmful
  for GLMSingle betas ([Prince et al., 2022](https://doi.org/10.7554/eLife.77599)),
  unnecessary for naturalistic ISFC/ISC and pRF.
- Localizer runs route to streams by analysis type, not by session type.
- **Dual output space:** streams B and C produce both `MNI152NLin2009cAsym
  res-2` (volumetric NIfTI) and `fsaverage6` (surface gifti) for every run.

---

## Layer 1: Shared Base

For each subject/session/run, layer 1 is the fMRIPrep output (original or
NORDIC-denoised) plus a **QC decisions file** recording all human-review
outcomes.

**NORDIC:** a per-run flag in the QC decisions file controls whether the
NORDIC-denoised or original fMRIPrep BOLD is used as the source. Once NORDIC
is validated and deployed, the default will be `true` for TB and resting
sessions; NAT sessions are evaluated separately.

### QC decisions

One TSV per subject/session at
`derivatives/preprocessing_qc/sub-XX/sub-XX_ses-YY_qc_decisions.tsv`:

| Column | Type | Description |
|--------|------|-------------|
| `task` | str | BIDS task label |
| `run` | str | run index or `n/a` |
| `exclude` | bool | Exclude run entirely from all streams |
| `exclude_reason` | str | Free text reason |
| `nordic` | bool | Use NORDIC-denoised BOLD as source |
| `fd_threshold` | float | FD cutoff for TR flagging (default 0.5 mm) |
| `n_outlier_trs` | int | Number of TRs flagged |
| `outlier_trs` | str | Comma-separated 0-indexed TR indices |
| `notes` | str | Reviewer notes |

Stubs are auto-generated from fMRIPrep framewise displacement data (FD stats
computed, TRs flagged at default threshold, all runs included by default).
A human reviewer then opens the TSV alongside the fMRIPrep HTML report and
MRIQC outputs and edits as needed. The file is version-controlled as a record
of all QC decisions.

---

## Layer 2: Analysis Streams

### Stream A — GLMSingle (`ready/glmsingle/`)

**Sessions:** TB (TBencoding, TBretrieval, TBmath, TBresting) + block-design
localizers (fLoc, motor, auditory, tone)

**What it produces per run:**
- `*_desc-confounds_ready.tsv` — curated confounds with one spike regressor
  per outlier TR
- `*_desc-outliers_mask.tsv` — boolean mask of bad TRs

**What it does NOT do:** modify the BOLD NIfTI. The source BOLD (fMRIPrep or
NORDIC fMRIPrep) is read directly by GLMSingle or the localizer GLM.

**Confound strategy (36-parameter + spikes):**
- 24 motion parameters (Friston 24: 6 realignment params + derivatives +
  quadratics)
- 6 anatomical CompCor components (combined WM+CSF mask)
- Cosine drift regressors (handles low-frequency drift without bandpass)
- One spike regressor per outlier TR

Spike regressors are the appropriate way to handle outlier TRs in a GLM
framework — the affected timepoint is down-weighted without removing it and
breaking the temporal structure of the design matrix.

---

### Localizer runs — stream assignment

Localizer sessions (ses-02, ses-03, ses-30) split across streams by analysis
type:

- **Block-design GLMs** (fLoc, motor, auditory, tone) → **GLMSingle stream**.
  Same recipe: BOLD untouched, curated confounds + TR flags passed to the
  analysis tool.
- **pRF / travelling wave** (prf task) → **Naturalistic stream**. prfpy
  expects confounds regressed, high-pass filtered, no bandpass, no smoothing,
  surface space preferred. The naturalistic stream's default dual-space output
  satisfies all of these with no special handling.

---

### Stream B — Naturalistic (`ready/naturalistic/`)

**Sessions:** NAT (movie-viewing, free recall, cued recall) + pRF localizer
**Analyses:** pattern similarity (RSA), ISFC, ISC, pRF model fitting

**What it produces per run (both spaces):**
- `*_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz`
- `*_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii`
- `*_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii`
- `*_desc-confounds_ready.tsv` (shared across spaces)

**Cleaning steps:**
1. Select confounds: 24HMP + 6 aCompCor + cosines
2. Interpolate outlier TRs (cubic spline) — before regression, to prevent
   contamination of confound estimates
3. Regress confounds via OLS
4. High-pass filter only (0.01 Hz / 100s)
5. No spatial smoothing

**Why no bandpass:** ISFC/ISC operate on shared inter-subject variance across
all frequencies; pRF stimuli modulate BOLD at a specific sweep frequency.
Both are harmed by lowpass filtering. High-pass only is the standard approach
for naturalistic paradigms.

---

### Stream C — Connectivity (`ready/connectivity/`)

**Sessions:** TBresting (and any dedicated resting runs added in future)
**Analyses:** seed-based FC, ICA, graph metrics

**What it produces per run (both spaces):**
- `*_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz`
- `*_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii`
- `*_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii`
- `*_desc-confounds_ready.tsv` (shared across spaces)
- `*_desc-outliers_mask.tsv`

**Cleaning steps:**
1. Select confounds: 24HMP + 6 aCompCor + cosines
2. Interpolate outlier TRs (cubic spline)
3. Regress confounds
4. Bandpass filter: 0.01–0.1 Hz
5. Scrub interpolated TRs from output (flagged in outliers mask)
6. Spatial smoothing: 4mm FWHM (volumetric for MNI; geodesic for fsaverage6)

**Global signal regression (GSR):** not applied by default due to introduced
anticorrelations. Deferred to when FC analyses begin.

---

## Filesystem layout

```
derivatives/
├── fmriprep/                     # Original fMRIPrep outputs (unchanged)
├── fmriprep_nordic/              # NORDIC fMRIPrep outputs (unchanged)
├── preprocessing_qc/
│   └── sub-XX/
│       └── sub-XX_ses-YY_qc_decisions.tsv
└── ready/
    ├── glmsingle/
    │   └── sub-XX/ses-YY/func/
    │       ├── *_desc-confounds_ready.tsv
    │       └── *_desc-outliers_mask.tsv
    ├── naturalistic/
    │   └── sub-XX/ses-YY/func/
    │       ├── *_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz
    │       ├── *_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii
    │       ├── *_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii
    │       └── *_desc-confounds_ready.tsv
    └── connectivity/
        └── sub-XX/ses-YY/func/
            ├── *_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz
            ├── *_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii
            ├── *_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii
            ├── *_desc-confounds_ready.tsv
            └── *_desc-outliers_mask.tsv
```

Brain masks are not duplicated — consumers read them from `fmriprep/` or
`fmriprep_nordic/` directly (same space, same subject).

---

## Confound strategy reference

| Component | Columns | Notes |
|-----------|---------|-------|
| Motion (Friston 24) | `trans_x/y/z`, `rot_x/y/z` + derivatives + quadratics | 24 total |
| Anatomical CompCor | `a_comp_cor_00`–`a_comp_cor_05` | Combined WM+CSF mask |
| Cosine drift | all `cosine*` columns | Handles drift without bandpass (GLMSingle stream) |
| Spike regressors | Generated per outlier TR | GLMSingle stream only |

Not included by default: global signal (anticorrelations), mean WM/CSF signals
(superseded by aCompCor), temporal CompCor.

---

## Open questions

1. **NORDIC for NAT sessions:** The NORDIC pilot covered a TB session. A NAT
   session pilot (longer timeseries, ~600+ TRs, movie-viewing) should be run
   before committing the NORDIC default for those sessions.

2. **T1w (native) space output:** Desirable for analyses requiring native
   resolution (HippUnfold, sub-millimetre ROI work). Deferred on disk space
   grounds — T1w BOLD is 2–4× larger per run than MNI res-2. fsaverage6 is
   comparatively cheap (~10% of volume size) and is included by default.

3. **Global signal regression for connectivity:** Deferred to when resting-state
   FC analyses begin; will be implemented as an opt-in variant.
