---
title: Derivatives & Preprocessing
parent: Data Organization
grand_parent: Dataset Description
nav_order: 3
---

# Derivatives & Preprocessing

> **How to check what actually exists.** This page lists the derivative trees
> and what each one is for. It does *not* assert per-subject completeness —
> that goes stale. Query the Contract A catalog
> (`inventory/catalog.duckdb`) for which subjects/sessions a given pipeline
> has covered.

The listing below is the set of derivative tree *names* the project uses. It
is not a claim that every one exists today: the NORDIC-arm deletion of
2026-08-26 (see [NORDIC arm (retired)](#nordic-arm-retired)) removed every
24.1.1-derived downstream tree, and this page does not track which of the
analysis trees (`glmsingle/`, `glmsingle_nat/`, `pattern_similarity/`,
`srm_stimulus_space/`, `qc/`, `qc_review/`, `preprocessing_qc/`) have since
been regenerated. The catalog's `datasets` table is the authority on which
trees exist and at what pipeline version.

```
derivatives/
├── fmriprep/               # fMRIPrep preprocessed data (see version note below)
├── mriqc/                  # MRIQC quality metrics and HTML reports
├── preprocessing_qc/       # Per-run QC decision records (auto-stubs; see below)
├── qc_review/              # HTML QC dashboards and BOLD QC benchmarks
├── qc/                     # QC benchmark outputs (GLMsingle, ISC comparisons)
├── atlases/                # Group-level reference atlases in template space
├── anatomical_rois/        # Convention only — EMPTY (see below)
├── functional_rois/        # Convention only — EMPTY (see below)
├── hippunfold/             # HippUnfold hippocampal surface unfolding (whole-brain T2w)
├── hippunfold_oblcor/      # HippUnfold using oblique-coronal T2w acquisition
├── hsf/                    # Hippocampal subfield segmentation — HSF (Poiret et al.)
├── hsf_oblcor/             # HSF using oblique-coronal T2w acquisition
├── glmsingle/              # GLMsingle single-trial betas, TB tasks
├── glmsingle_nat/          # GLMsingle betas, NAT tasks
├── pattern_similarity/     # Pattern-similarity benchmark results
├── srm_stimulus_space/     # Shared-response-model stimulus-space analyses
├── stimuli_features/       # Computational stimulus features (see stimuli_features.md)
└── behavioral_analysis/    # Behavioral accuracy, d-prime, and learning analyses
```

**Pipeline versions.** The derivative trees on disk record their own
provenance in `dataset_description.json` — read it there rather than trusting
a number quoted in prose. `fmriprep/` is **fMRIPrep 25.2.5** output; `mriqc/`
was produced with **MRIQC 24.1.0**.

> **Version note.** A re-preprocessing campaign finished **2026-08-26**.
> `derivatives/fmriprep/` now holds fMRIPrep **25.2.5** output, with external
> FreeSurfer **8.2.0** recons, for every functional run of the BIDSified
> subjects at that date (sub-03 through sub-07). **No 24.1.1 output exists
> anywhere in the dataset.** The 25.2.5 tree emits native func-space BOLD
> (the `func` output space, no `space-` entity) alongside
> `MNI152NLin2009cAsym res-2` and `fsaverage6`; see
> [Output Spaces](preprocessing-spaces.md). For runs added after the
> campaign, query the catalog rather than assuming coverage.

**Trees removed since earlier versions of this page:** `bids_validation/`
(deleted 2026-08 after review; 256 GB) and `fmriprep_pre022426/` (archived
legacy fMRIPrep run). Neither exists any more.

## Pipeline overview

```
sourcedata (DICOMs, raw behavioral)          [mmmsourcedata/, outside the BIDS tree]
    │
    ▼
BIDS raw (NIfTI + JSON + events TSV)
    │
    ├──▶ MRIQC (quality metrics, outlier detection)
    │
    ├──▶ fMRIPrep (registration, distortion correction, confounds)
    │        │
    │        ├──▶ glmsingle* (single-trial betas)
    │        ├──▶ pattern_similarity/, srm_stimulus_space/ (analyses)
    │        └──▶ preprocessing_qc/ (per-run QC decision records)
    │
    ├──▶ atlases/ (reference parcellations in template space)
    │
    ├──▶ anatomical_rois/, hippunfold*/, hsf*/ (structural segmentations)
    │
    └──▶ functional_rois/ (subject-specific task-defined ROIs)
```

- MRIQC and fMRIPrep run in parallel, producing complementary QA output.
- Downstream analysis trees (`glmsingle*`, `pattern_similarity/`,
  `srm_stimulus_space/`) read `fmriprep/` **directly**.
- `derivatives/ready/` — the "analysis-ready streams" layer — is a **design
  that has not been implemented**. The directory does not exist and no stream
  cleaner has run. See
  [Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md), which
  labels each part of that design. Do not write code against `ready/` paths.

## Atlases

Group-level reference atlases stored at the template level (no per-subject data).
Each atlas has a `dataset_description.json` and an `atlas-<label>_description.json`
sidecar with provenance, license, and citation info.

### Schaefer 2018

Local-global cortical parcellation (Schaefer et al., 2018, _Cerebral Cortex_).
Available in **7-network** and **17-network** solutions at 8 granularities
(100, 200, 300, 400, 500, 600, 800, 1000 parcels). All files are in
`MNI152NLin2009cAsym` space at 2 mm resolution, matching fMRIPrep output.

```
derivatives/atlases/
├── dataset_description.json
├── atlas-Schaefer2018_description.json
└── tpl-MNI152NLin2009cAsym/
    └── anat/
        ├── tpl-MNI152NLin2009cAsym_atlas-Schaefer2018_seg-7n_scale-100_res-2_dseg.nii.gz
        ├── tpl-MNI152NLin2009cAsym_atlas-Schaefer2018_seg-7n_scale-100_res-2_dseg.tsv
        ├── ...  (7n × 8 scales + 17n × 8 scales = 32 NIfTI + 32 TSV)
        └── tpl-MNI152NLin2009cAsym_atlas-Schaefer2018_seg-17n_scale-1000_res-2_dseg.tsv
```

**Entity key:** `seg-7n` / `seg-17n` = network solution, `scale-100` = number of
parcels, `res-2` = 2 mm isotropic resolution.

## Anatomical ROIs

> **Status: EMPTY — naming convention only.** As of 2026-08-20
> `derivatives/anatomical_rois/` contains nothing but its
> `dataset_description.json`. No ROI has been generated. The tree below is the
> convention new ROIs should follow, not a listing of files you can load.
> Hippocampal segmentations that *do* exist are in `hippunfold*/` and `hsf*/`.

Intended for subject-specific ROIs derived from structural imaging (e.g.,
hippocampal subfield segmentations, custom FreeSurfer-based masks), stored
per-subject (and optionally per-session if the definition is session-specific).

```
derivatives/anatomical_rois/
├── dataset_description.json
└── sub-03/
    └── anat/
        ├── sub-03_seg-hippsubfields_dseg.nii.gz     # Full subfield parcellation
        ├── sub-03_seg-hippsubfields_dseg.tsv         # Label lookup table
        ├── sub-03_label-CA1_mask.nii.gz              # Individual subfield mask
        └── sub-03_label-CA1_mask.json                # {"Type": "ROI", "Sources": [...]}
```

## Functional ROIs

> **Status: EMPTY — naming convention only.** As of 2026-08-20
> `derivatives/functional_rois/` contains nothing but its
> `dataset_description.json`. No functional ROI has been generated.

Intended for subject-specific ROIs derived from task-based or resting-state
analyses (e.g., localizer contrasts, seed-based connectivity maps). Each mask's
JSON sidecar would document the contrast, threshold, and statistical criteria.

```
derivatives/functional_rois/
├── dataset_description.json
└── sub-03/
    └── ses-01/
        └── func/
            ├── sub-03_ses-01_task-localizer_space-MNI152NLin2009cAsym_label-FFA_mask.nii.gz
            ├── sub-03_ses-01_task-localizer_space-MNI152NLin2009cAsym_label-FFA_mask.json
            └── ...
```

## NORDIC arm (retired)

The dataset previously carried a parallel NORDIC-denoised preprocessing arm:
`nordic/` (raw NORDIC outputs) and `fmriprep_nordic/` (fMRIPrep run on the
denoised BOLD), with `_nordic`-suffixed GLMsingle trees downstream. NORDIC
retention was dissolved on 2026-08-25 and those trees were **deleted on
2026-08-26**, together with every other 24.1.1-derived downstream tree. The
default (and only) preprocessed tree is `derivatives/fmriprep/`. There was
never a per-run NORDIC source flag anywhere in the dataset. Re-running NORDIC
on the new preprocessing is recorded as a "later" todo, not a plan.

## QC Review

HTML dashboards for visual QC of structural and functional scans, generated
from MRIQC outputs. Includes per-subject and group-level views for T1w, T2w,
and BOLD, plus `bold_qc_benchmarks.md` — a reference table of absolute IQM
thresholds with citations.

```
derivatives/qc_review/
├── bold_qc_benchmarks.md
├── dashboards/
│   ├── qc_dashboard_all_bold.html
│   ├── qc_dashboard_sub-03_bold.html
│   └── ...  (per-subject T1w, T2w, BOLD dashboards)
└── ...
```

## MRIQC

Image quality metrics generated by MRIQC v24.1.0 for structural and functional
scans. Outputs include individual HTML visual reports and per-image quality
metric (IQM) JSON files for automated outlier detection.

```
derivatives/mriqc/
├── dataset_description.json
├── logs/
├── sub-03/
│   ├── figures/
│   ├── ses-01/
│   │   └── anat/       # T1w, T2w quality metrics + figures
│   ├── ses-02/ ...
│   └── ses-30/
│       └── func/       # BOLD quality metrics + figures
├── sub-04/
├── sub-05/
├── sub-03_ses-01_acq-MPR_run-01_T1w.html   # Individual report pages
└── ...
```

## Behavioral Analysis

Group-level behavioral analysis results from the trial-based memory paradigm,
generated by `analyze_behavior.py` in the mmmdata codebase.

> **Status: PARTIAL.** Only the two group-level tables exist. The
> `figures/`, `sub-03/`, `sub-04/` and `sub-05/` directories are present but
> **empty** — no per-subject outputs or figures have been generated.

```
derivatives/behavioral_analysis/
├── group/
│   ├── accuracy_by_enCon.tsv    # Accuracy by encoding condition   [exists]
│   └── dprime_by_subject.tsv    # Signal detection (d') per subject [exists]
├── figures/                      # empty
├── sub-03/                       # empty
├── sub-04/                       # empty
└── sub-05/                       # empty
```

## QC decisions (`preprocessing_qc/`)

One JSON per BOLD run recording an append-only decision history
(`keep` / `exclude` / `investigate` / `pending`).

**Every record currently on disk is an auto-generated stub** written in
2026-04 from framewise-displacement statistics — no run has a human QC
sign-off yet. Treat the dataset as not yet QC-reviewed. Schema and the
read/write API are documented on the
[Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md) page.

## Stimulus features (`stimuli_features/`)

Computational features for the three stimulus sets, on the Contract B 0.5 s
grid. See [Computational Feature Extraction](stimuli_features.md).

---

*Tree listing on this page verified against the filesystem on 2026-08-20;
derivative-tree and version claims updated 2026-09-05 from the
reprocessing-campaign record. Per-subject completeness is deliberately not
asserted here — query the catalog.*
