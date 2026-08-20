---
title: Analysis-Ready Preprocessing Pipeline
parent: Data Organization
grand_parent: Dataset Description
nav_order: 4
---

# Analysis-Ready Preprocessing Pipeline

> **Status — DESIGN. Layer 2 does not exist.**
> This page is a *design document*, not a description of data you can load.
> As of **2026-08-20**, `derivatives/ready/` has never been created: none of
> the three streams below has been implemented, and no stream cleaner has run
> on any subject. Layer 1 exists only in part (see
> [Layer 1](#layer-1-shared-base) for exactly what is on disk).
>
> Do not write analysis code against the `ready/` paths on this page. To use
> preprocessed BOLD today, read `derivatives/fmriprep/` or
> `derivatives/fmriprep_nordic/` directly — see
> [Derivatives & Preprocessing](derivatives.md) and
> [Output Spaces](preprocessing-spaces.md).

Status legend used below:

| Label | Meaning |
|---|---|
| **BUILT** | Exists on disk and has been produced for the subjects named |
| **PARTIAL** | Some of it exists; the section says which part |
| **PLANNED** | Design only. No code has run, no files exist |

---

## Overview

The intended design is a two-layer pipeline taking fMRIPrep (optionally
NORDIC-denoised) output and producing analysis-ready data for three analysis
types, all sharing a single quality-control layer.

```
Layer 1 — Shared base (fMRIPrep outputs + QC decisions)   [PARTIAL]
    ↓
Layer 2 — Stream-specific cleaning                         [PLANNED — nothing built]
    ├── ready/glmsingle/      ← TB sessions + block-design localizers
    ├── ready/naturalistic/   ← NAT sessions + pRF localizer
    └── ready/connectivity/   ← resting-state sessions
```

**Design principles** (these are the rationale for the design; they are not
claims about existing data):

- QC decisions (run exclusions, bad TRs, NORDIC choice) are to be made **once**
  at layer 1 and propagated to all streams, so that comparisons across analysis
  types rest on an identical QC substrate.
- The BOLD timeseries would **never be modified** for the GLMSingle stream —
  only a curated confounds file and TR exclusion flags. GLMSingle handles its
  own noise modeling internally.
- Bandpass filtering is **stream-specific**: required for connectivity
  ([Cordes et al., 2001](https://pubmed.ncbi.nlm.nih.gov/11498421/)), harmful
  for GLMSingle betas ([Prince et al., 2022](https://doi.org/10.7554/eLife.77599)),
  unnecessary for naturalistic ISFC/ISC and pRF.
- Localizer runs route to streams by analysis type, not by session type.
- **Dual output space:** streams B and C would produce both
  `MNI152NLin2009cAsym res-2` (volumetric NIfTI) and `fsaverage6` (surface
  GIFTI) for every run.

---

## Layer 1: Shared Base

**Status: PARTIAL.** The fMRIPrep half is BUILT. The QC-decision half exists as
auto-generated stubs only, in a schema different from the one this page
originally described, and **no human review has been recorded for any run**.

### What is on disk

`derivatives/preprocessing_qc/sub-XX/` holds **one JSON per BOLD run** —
203 runs each for sub-03, sub-04 and sub-05 (609 total), written 2026-04 by
`mmmdata/scripts/generate_qc_stubs.py` from `fmriprep_nordic` confounds.

Every one of those 609 records is an **auto-generated stub**
(`"reviewer": "auto-stub"`). None reflects a human judgement. Treat the
dataset as **not yet QC-reviewed**.

The record format is an append-only decision history per run:

```json
{
  "run_key": "sub-03_ses-02_task-prf_run-01_bold",
  "decisions": [
    {
      "decision": "keep",
      "reason": "auto-stub from fmriprep_nordic confounds; mean_fd=0.064mm; ...",
      "reviewer": "auto-stub",
      "timestamp": "2026-04-16T16:37:08.948001+00:00"
    }
  ]
}
```

| Field | Meaning |
|---|---|
| `run_key` | `sub-XX_ses-YY_task-ZZZ[_run-NN]_bold` |
| `decisions[]` | Append-only history; the **last** entry is current |
| `decision` | `keep`, `exclude`, `investigate`, or `pending` |
| `reason` | Free text; for stubs, the FD statistics behind the suggestion |
| `reviewer` | Person's identifier, or an automated-writer name |
| `automated` | `true` when written by tooling rather than a person |
| `recommendation` | Automated writers' advisory suggestion. Never gates anything |
| `timestamp` | UTC ISO-8601 |

Written and read by `save_decision` / `load_decisions` in
`mmmdata/src/python/neuroimaging/qc_dashboard.py`, and by the interactive QC
dashboard. Automated writers may only record `pending`, carrying any suggestion
in `recommendation` — so a record can never read as a human sign-off when it
is not one. The 2026-04 stubs on disk predate that rule and record `keep`
directly; they are still auto-generated and are not sign-offs.

**Location note:** new decisions are written to
`derivatives/duckbrain/qc/decisions/` (flat, shared with duckbrain).
`derivatives/preprocessing_qc/` and `derivatives/qc_review/decisions/` are
legacy read-only locations; `load_decisions` searches all three, newest last.

### What is not built

The fields this page previously specified — `fd_threshold`, `n_outlier_trs`,
`outlier_trs`, and a per-run `nordic` source flag — **do not exist** in any
record. Per-TR outlier flagging is part of the layer-2 design and has not been
implemented. Framewise displacement is available per run from the fMRIPrep
confounds TSVs in the meantime.

**NORDIC:** the design called for a per-run flag selecting the NORDIC or
original source. No such flag exists. Both `fmriprep/` and `fmriprep_nordic/`
are complete for all three subjects, so consumers currently choose a tree by
path. As of 2026-08, new analyses default to the **non-NORDIC** `fmriprep/`
tree.

---

## Layer 2: Analysis Streams

> **Status: PLANNED — none of the following exists.** `derivatives/ready/` has
> not been created. No file described in this section is on disk for any
> subject. The stream cleaners have not been written.

The design below is retained as the specification the streams would be built
to.

### Stream A — GLMSingle (`ready/glmsingle/`) — PLANNED

**Sessions:** TB (TBencoding, TBretrieval, TBmath, TBresting) + block-design
localizers (fLoc, motor, auditory, tone)

**Would produce per run:**
- `*_desc-confounds_ready.tsv` — curated confounds with one spike regressor
  per outlier TR
- `*_desc-outliers_mask.tsv` — boolean mask of bad TRs

**Would NOT do:** modify the BOLD NIfTI. The source BOLD (fMRIPrep or NORDIC
fMRIPrep) would be read directly by GLMSingle or the localizer GLM.

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

### Localizer runs — stream assignment — PLANNED

Localizer sessions (ses-02, ses-03, ses-30) would split across streams by
analysis type:

- **Block-design GLMs** (fLoc, motor, auditory, tone) → **GLMSingle stream**.
  Same recipe: BOLD untouched, curated confounds + TR flags passed to the
  analysis tool.
- **pRF / travelling wave** (prf task) → **Naturalistic stream**. prfpy
  expects confounds regressed, high-pass filtered, no bandpass, no smoothing,
  surface space preferred. The naturalistic stream's dual-space output would
  satisfy all of these with no special handling.

---

### Stream B — Naturalistic (`ready/naturalistic/`) — PLANNED

**Sessions:** NAT (movie-viewing, free recall, cued recall) + pRF localizer
**Analyses:** pattern similarity (RSA), ISFC, ISC, pRF model fitting

**Would produce per run (both spaces):**
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

### Stream C — Connectivity (`ready/connectivity/`) — PLANNED

**Sessions:** TBresting (and any dedicated resting runs added in future)
**Analyses:** seed-based FC, ICA, graph metrics

**Would produce per run (both spaces):**
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

**Global signal regression (GSR):** not in the default design, due to
introduced anticorrelations. Deferred to when FC analyses begin.

---

## Filesystem layout — planned

> Everything under `ready/` in this tree is **PLANNED**. Only `fmriprep/`,
> `fmriprep_nordic/` and `preprocessing_qc/` exist today.

```
derivatives/
├── fmriprep/                     # BUILT — original fMRIPrep outputs
├── fmriprep_nordic/              # BUILT — NORDIC fMRIPrep outputs
├── preprocessing_qc/             # PARTIAL — auto-stubs only, no human review
│   └── sub-XX/
│       └── sub-XX_ses-YY_task-ZZZ[_run-NN]_bold_decision.json
└── ready/                        # PLANNED — does not exist
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

Brain masks would not be duplicated — consumers read them from `fmriprep/` or
`fmriprep_nordic/` directly (same space, same subject).

---

## Confound strategy reference

Applies to the planned streams. The confound *columns* themselves are real —
they come from the existing fMRIPrep `*_desc-confounds_timeseries.tsv` files
and can be used directly today.

| Component | Columns | Notes |
|-----------|---------|-------|
| Motion (Friston 24) | `trans_x/y/z`, `rot_x/y/z` + derivatives + quadratics | 24 total |
| Anatomical CompCor | `a_comp_cor_00`–`a_comp_cor_05` | Combined WM+CSF mask |
| Cosine drift | all `cosine*` columns | Handles drift without bandpass (GLMSingle stream) |
| Spike regressors | Would be generated per outlier TR | GLMSingle stream only; not yet generated |

Not included by default: global signal (anticorrelations), mean WM/CSF signals
(superseded by aCompCor), temporal CompCor.

---

## Open questions

1. **Whether to build layer 2 at all in this form.** The streams have been
   parked since the design was written; ad-hoc analyses (GLMSingle, pattern
   similarity, ISC, SRM) have instead read `fmriprep*/` directly. Any revival
   should confirm the three-stream split still matches how the data are
   actually being used.

2. **NORDIC for NAT sessions.** Both fMRIPrep variants are complete for all
   three subjects and have been benchmarked. As of 2026-08, new analyses
   default to the non-NORDIC tree; a decision to retire the NORDIC arm
   entirely has not been recorded.

3. **T1w (native) space output.** Desirable for analyses requiring native
   resolution (HippUnfold, sub-millimetre ROI work). Deferred on disk-space
   grounds — T1w BOLD is 2–4× larger per run than MNI res-2.

4. **Global signal regression for connectivity.** Deferred to when
   resting-state FC analyses begin; would be an opt-in variant.

---

*Dataset claims on this page verified against the filesystem on 2026-08-20.*
