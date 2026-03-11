---
title: Analysis-Ready Preprocessing Pipeline
parent: Data Organization
grand_parent: Dataset Description
nav_order: 4
---

# Analysis-Ready Preprocessing Pipeline — Implementation Plan

## Overview

A two-layer pipeline that takes fMRIPrep + NORDIC outputs and produces
analysis-ready data for three distinct consumer types, all sharing a single QC
decision substrate.

```
Layer 1 — Shared base (fMRIPrep outputs + human QC decisions)
    ↓
Layer 2 — Stream-specific cleaning (deterministic from layer 1)
    ├── ready/glmsingle/      ← TB sessions + block-design localizers
    ├── ready/naturalistic/   ← NAT sessions + pRF localizer (surface variant)
    └── ready/connectivity/   ← resting-state sessions
```

**Key design principles:**
- QC decisions (run exclusions, bad TRs, NORDIC choice) are made **once** at
  layer 1 and propagated to all streams. Comparisons across analysis types are
  valid because the QC substrate is identical.
- The BOLD timeseries is **never modified** for the GLMSingle stream — only a
  curated confounds file and TR exclusion flags are produced. GLMSingle handles
  its own noise modeling; pre-cleaning would introduce double-subtraction and
  distort HRF shape.
- Bandpass filtering is **stream-specific**: required for connectivity, harmful
  for GLMSingle betas, unnecessary for naturalistic ISFC/ISC and pRF.
- Localizer runs split across streams by analysis type, not by the fact of
  being localizers (see Localizer section below).
- **Dual output space by default:** streams B and C produce both
  `MNI152NLin2009cAsym res-2` (volumetric NIfTI) and `fsaverage6` (surface
  gifti) outputs for every run. The dataset is a versatile resource and
  analysis space demands are variable. Confounds and TR masks are
  space-agnostic and shared across both spaces. T1w (native) space is an
  open question — see Open Questions.

---

## Layer 1: Shared Base

### What it contains

For each subject/session/run, layer 1 is the fMRIPrep output (original or
NORDIC-denoised, see NORDIC note below) plus a **QC decisions file** recording
all human-review outcomes.

**Source BOLD selection (NORDIC):**
Pending full-dataset NORDIC processing, the pipeline will have a single
`nordic` flag per run in the QC decisions file. Default will be `true` for all
TB and resting sessions once NORDIC passes visual QC; NAT sessions evaluated
separately (see nordic-pilot-plan.md).

### QC decisions schema

One TSV per subject/session:
`derivatives/preprocessing_qc/sub-XX/sub-XX_ses-YY_qc_decisions.tsv`

| Column | Type | Description |
|--------|------|-------------|
| `task` | str | BIDS task label |
| `run` | str | run index or `n/a` |
| `exclude` | bool | Exclude run entirely from all streams |
| `exclude_reason` | str | Free text (e.g., "excessive motion", "scanner artifact") |
| `nordic` | bool | Use NORDIC-denoised BOLD as source (vs original fMRIPrep) |
| `fd_threshold` | float | FD cutoff used for TR flagging (default 0.5 mm) |
| `n_outlier_trs` | int | Number of TRs flagged at this threshold |
| `outlier_trs` | str | Comma-separated 0-indexed TR indices |
| `notes` | str | Free-text reviewer notes |

**Generation workflow:**
1. A script (`scripts/generate_qc_stubs.py`) auto-populates stubs from
   fMRIPrep confounds TSVs: computes per-run FD stats, flags TRs at default
   threshold, fills `n_outlier_trs`/`outlier_trs`. All `exclude` values
   default to `false`.
2. Human reviewer opens the TSV alongside the fMRIPrep HTML report and MRIQC
   outputs, edits `exclude`, `fd_threshold`, `outlier_trs`, and `notes` as
   needed.
3. File is committed to mmmdata-docs (or kept in mmmdata under
   `derivatives/preprocessing_qc/`) as a versioned record of QC decisions.

---

## Layer 2: Analysis Streams

### Stream A — GLMSingle (`ready/glmsingle/`)

**Consumer:** TB sessions (TBencoding, TBretrieval, TBmath, TBresting)

**What it produces per run:**
- `*_desc-confounds_ready.tsv` — subset of fMRIPrep confounds with selected
  columns only (see below); bad TRs zeroed out via a spike regressor column
  per outlier TR
- `*_desc-outliers_mask.tsv` — single-column boolean mask of bad TRs (for
  passing to GLMSingle's `rununits`/`ignoredframes` argument directly)

**What it does NOT do:** touch the BOLD NIfTI. GLMSingle reads the fMRIPrep
(or NORDIC fMRIPrep) BOLD directly.

**Confound selection strategy (36-parameter + spikes):**
- 24 motion parameters: 6 realignment params + temporal derivatives +
  quadratics (Friston 24-parameter model)
- 6 anatomical CompCor components: `a_comp_cor_00`–`a_comp_cor_05` (top
  aCompCor components from combined WM+CSF mask — fMRIPrep's default
  `a_comp_cor_*` series)
- Cosine drift regressors: all `cosine*` columns (already in fMRIPrep output;
  handles low-frequency drift without bandpass filtering)
- One spike regressor per outlier TR (binary column, 1 at the bad TR)

**Rationale:** aCompCor is preferred over mean WM/CSF signals because it
doesn't require ROI signal choices and captures more variance. No bandpass
filter; cosines handle drift. Spike regressors are the correct way to handle
outlier TRs in a GLM framework — the trial at that TR is effectively
down-weighted without removing the timepoint and breaking the temporal
structure of the design matrix.

---

### Localizer runs — stream assignment

Localizer sessions (ses-02, ses-03, ses-30) contain two fundamentally different
analysis types that map to different existing streams:

**Block-design localizers → GLMSingle stream**
Tasks: fLoc, motor, auditory, tone (when run as a blocked GLM).
These are standard block-design GLMs (nilearn `FirstLevelModel`), not
GLMSingle per se, but the preprocessing recipe is identical: pass the fMRIPrep
BOLD untouched alongside a curated confounds TSV + TR exclusion flags. The
existing `localizer_glm_utils.py` in mmmdata already reads fMRIPrep confounds
directly — it will read from `ready/glmsingle/` once that is populated.

**pRF / travelling wave → naturalistic stream**
Task: prf (retinotopic mapping).
prfpy and similar tools expect confounds regressed, high-pass filtered, no
bandpass, no smoothing, and preferably surface space. Since the naturalistic
stream now produces fsaverage6 output by default for all runs, pRF localizer
runs require no special treatment — they slot directly into the naturalistic
stream and the surface output is already there.

This keeps the stream count at three — no dedicated localizer stream needed.

---

### Stream B — Naturalistic (`ready/naturalistic/`)

**Consumer:** NAT sessions (movie-viewing, free recall, cued recall) +
pRF localizer (surface variant)
**Analyses:** pattern similarity (RSA), ISFC, ISC, pRF model fitting

**What it produces per run (both spaces):**
- `*_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz` — cleaned BOLD, MNI volume
- `*_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii` — cleaned BOLD, left surface
- `*_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii` — cleaned BOLD, right surface
- `*_desc-confounds_ready.tsv` — shared across both spaces

**Cleaning steps (applied identically to both spaces):**
1. Select confounds: 24HMP + 6 aCompCor + cosines
2. Interpolate outlier TRs (cubic spline) — done on the timeseries before regression
3. Regress confounds via OLS
4. Apply high-pass filter only (0.01 Hz / 100s) — removes slow drift without
   attenuating the signal variance that ISFC, pattern similarity, and pRF rely on
5. **No spatial smoothing** — pattern analyses require unsmoothed data

**Rationale for no bandpass:** ISFC/ISC operate on shared inter-subject
variance across all frequencies. pRF stimuli modulate BOLD at a specific
sweep frequency. Both are harmed by lowpass filtering. High-pass only is the
standard approach for naturalistic paradigms (cf. Nastase et al. 2019).

**Rationale for interpolation before regression:** Outlier TRs with extreme
motion inflate confound regression estimates if left in. Interpolating first
stabilises the regression.

---

### Stream C — Connectivity (`ready/connectivity/`)

**Consumer:** Resting-state sessions (TBresting, and dedicated resting runs if
added later)
**Analyses:** seed-based FC, ICA, graph metrics

**What it produces per run (both spaces):**
- `*_space-MNI152NLin2009cAsym_res-2_desc-preproc_bold.nii.gz` — cleaned BOLD, MNI volume
- `*_hemi-L_space-fsaverage6_desc-preproc_bold.func.gii` — cleaned BOLD, left surface
- `*_hemi-R_space-fsaverage6_desc-preproc_bold.func.gii` — cleaned BOLD, right surface
- `*_desc-confounds_ready.tsv` — shared across both spaces
- `*_desc-outliers_mask.tsv` — TR exclusion mask

**Cleaning steps (applied identically to both spaces):**
1. Select confounds: 24HMP + 6 aCompCor + cosines
2. Interpolate outlier TRs (cubic spline)
3. Regress confounds from BOLD
4. Bandpass filter: 0.01–0.1 Hz
5. Scrub interpolated TRs from output (flagged in outliers mask)
6. Spatial smoothing: volumetric 4mm FWHM for MNI output; surface geodesic
   smoothing (FWHM equivalent) for fsaverage6 output. Parameterizable,
   default 4mm. Applied last.

**Open question — global signal regression (GSR):** GSR removes a global
artefact component but introduces artificial anticorrelations. Keep as an
off-by-default option, not applied by default. Decision deferred to when FC
analyses are actively being run.

---

## Filesystem layout

```
derivatives/
├── fmriprep/                          # Original fMRIPrep outputs (unchanged)
├── fmriprep_nordic/                   # NORDIC fMRIPrep outputs (unchanged)
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

Brain masks are not copied — consumers read them from `fmriprep/` or
`fmriprep_nordic/` directly (same space, same subject).

---

## Implementation phases

### Phase 1: QC stub generator

**Script:** `scripts/generate_qc_stubs.py` (mmmdata repo)

- Walk `derivatives/fmriprep/sub-*/ses-*/func/*_desc-confounds_timeseries.tsv`
- For each run: compute mean FD, max FD, flag TRs at default threshold (0.5mm)
- Write stub TSV to `derivatives/preprocessing_qc/sub-XX/sub-XX_ses-YY_qc_decisions.tsv`
- Skip runs where stub already exists (idempotent — don't overwrite human edits)
- `--subjects`, `--sessions`, `--fd-threshold` flags

### Phase 2: GLMSingle stream cleaner

**Script:** `scripts/clean_glmsingle.py` (mmmdata repo)

- Reads QC decisions TSV; skips excluded runs
- Selects confound columns (24HMP + 6 aCompCor + cosines)
- Adds one spike regressor column per outlier TR
- Writes `*_desc-confounds_ready.tsv` and `*_desc-outliers_mask.tsv`
- Reads from `fmriprep/` or `fmriprep_nordic/` per the `nordic` flag in QC
  decisions
- `--subjects`, `--sessions`, `--dry-run` flags

### Phase 3: Naturalistic stream cleaner

**Script:** `scripts/clean_naturalistic.py` (mmmdata repo)

- Reads QC decisions; skips excluded runs
- Interpolates outlier TRs (scipy cubic spline)
- Regresses selected confounds (nilearn `clean_img` or manual OLS)
- Applies high-pass filter (nilearn, 0.01 Hz)
- Produces both MNI NIfTI and fsaverage6 giftis (L+R) for every run
- Writes output to `ready/naturalistic/`
- `--subjects`, `--sessions`, `--dry-run` flags

### Phase 4: Connectivity stream cleaner

**Script:** `scripts/clean_connectivity.py` (mmmdata repo)

- Reads QC decisions; skips excluded runs
- Interpolates outlier TRs
- Regresses confounds
- Bandpass filters (0.01–0.1 Hz, nilearn)
- Scrubs interpolated TRs (flags in outlier mask)
- Applies spatial smoothing: volumetric Gaussian (nilearn) for MNI, geodesic
  smoothing for fsaverage6 (via nibabel/nilearn surface tools), default 4mm
- Produces both MNI NIfTI and fsaverage6 giftis (L+R) for every run
- Writes cleaned outputs + outlier mask to `ready/connectivity/`
- `--subjects`, `--sessions`, `--fwhm`, `--dry-run` flags

### Phase 5: Orchestrator script

**Script:** `scripts/run_preprocessing.sh` (mmmdata repo)

Chains all phases in order, with Slurm array job support for batch processing:
```
generate_qc_stubs.py      # only if stubs missing
[human QC review]         # manual step — script pauses / run separately
clean_glmsingle.py
clean_naturalistic.py
clean_connectivity.py
```

Logs which runs were processed / skipped / failed to
`derivatives/preprocessing_qc/logs/`.

### Phase 6: Manifest integration

- Extend `build_manifest.py` (mmmdata) to ingest `ready/` outputs into a new
  `ready_files` table: subject, session, task, run, stream, path, n_outlier_trs
- Expose via `query_manifest()` in mmmdata-agents

---

## Confound strategy reference

| Strategy component | Columns | Notes |
|--------------------|---------|-------|
| Motion (Friston 24) | `trans_x/y/z`, `rot_x/y/z` + `_derivative1` + `_power2` + `_derivative1_power2` | 24 total |
| Anatomical CompCor | `a_comp_cor_00`–`a_comp_cor_05` | Top 6 from combined WM+CSF mask |
| Cosine drift | all `cosine*` columns | Replaces highpass filter for GLMSingle stream |
| Spike regressors | Generated per outlier TR | GLMSingle stream only |

**Not used by default:**
- `global_signal` — excluded (introduces artificial anticorrelations)
- `csf` / `white_matter` mean signals — superseded by aCompCor
- `tcompcor` — temporal CompCor, less principled than anatomical for task data
- `c_comp_cor_*`, `w_comp_cor_*` — CSF-only and WM-only CompCor; `a_comp_cor`
  (combined) is preferred

---

## Open questions

1. **QC decisions storage location:** `derivatives/preprocessing_qc/` in the
   BIDS root (version-controlled via mmmdata git) vs. mmmdata-docs (shared
   docs repo). Argument for BIDS root: closer to the data it annotates.
   Argument for mmmdata-docs: makes QC decisions part of the shared knowledge
   base. Likely BIDS root is cleaner.

2. **NORDIC for NAT sessions:** Currently unresolved — pilot was on TB session.
   A NAT-session NORDIC pilot (longer timeseries, ~600+ TRs) is worth running
   before committing the `nordic` default for those sessions.

3. **GSR for connectivity:** Deferred. When resting-state FC analyses begin,
   decide whether to run a parallel GSR-corrected stream or leave it to the
   analyst.

4. **T1w (native) space output:** Conceptually desirable — some analyses
   (HippUnfold, layer fMRI, sub-millimetre ROI work) prefer native resolution
   without MNI resampling. The concern is disk space: T1w BOLD is ~2–4× larger
   per run than MNI res-2. fsaverage6 surface output is comparatively cheap
   (~10% of the volume size). Decision: defer T1w until disk space and use
   cases are clearer; the pipeline scripts should make it easy to add via a
   `--t1w` flag when needed.

5. **Session/task type → stream mapping:** Currently manual. Proposed canonical
   mapping:
   - TB sessions (TBencoding, TBretrieval, TBmath) → glmsingle
   - TBresting → connectivity
   - NAT sessions → naturalistic
   - Localizer block-design (floc, motor, auditory, tone) → glmsingle
   - Localizer pRF → naturalistic
   Could be auto-inferred from session-type + task columns in `_sessions.tsv`
   once the mapping is confirmed stable.
