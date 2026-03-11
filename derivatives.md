---
title: Derivatives & Preprocessing
parent: Data Organization
grand_parent: Dataset Description
nav_order: 3
---

# Derivatives & Preprocessing

```
derivatives/
├── fmriprep/              # fMRIPrep v24.1.1 preprocessed data (all sessions)
├── fmriprep_nordic/       # fMRIPrep v24.1.1 run on NORDIC-denoised BOLD
├── nordic/                # Raw NORDIC denoising outputs (pre-fMRIPrep)
├── mriqc/                 # MRIQC v24.1.0 quality metrics and HTML reports
├── qc_review/             # HTML QC dashboards and BOLD QC benchmarks
├── atlases/               # Group-level reference atlases in template space
├── anatomical_rois/       # Subject-specific anatomically-defined ROIs
├── functional_rois/       # Subject-specific functionally-defined ROIs
├── hippunfold/            # HippUnfold hippocampal surface unfolding (whole-brain T2w)
├── hippunfold_oblcor/     # HippUnfold using oblique-coronal T2w acquisition
├── hsf/                   # Hippocampal subfield segmentation — HSF (Poiret et al.)
├── hsf_oblcor/            # HSF using oblique-coronal T2w acquisition
├── behavioral_analysis/   # Behavioral accuracy, d-prime, and learning analyses
├── bids_validation/       # Validation outputs, extracted event files, survey logs
└── fmriprep_pre022426/    # Archived earlier fMRIPrep run (legacy)
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
    ├──▶ fMRIPrep / fMRIPrep+NORDIC (preprocessing: registration, distortion correction, confounds)
    │
    ├──▶ atlases/ (reference parcellations in template space)
    │
    ├──▶ anatomical_rois/ (subject-specific anatomical segmentations)
    │
    ├──▶ functional_rois/ (subject-specific task-defined ROIs)
    │
    └──▶ ready/ (analysis-ready streams — see Analysis-Ready Preprocessing Pipeline)
```

- MRIQC and fMRIPrep run in parallel, producing complementary QA output.
- `fmriprep_pre022426/` is an archived earlier run; the canonical output is in
  `fmriprep/`.
- Analysis-ready outputs (`ready/glmsingle/`, `ready/naturalistic/`,
  `ready/connectivity/`) are documented in the
  [Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md) page.

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

Subject-specific ROIs derived from structural imaging (e.g., hippocampal subfield
segmentations, custom FreeSurfer-based masks). Stored per-subject (and optionally
per-session if the definition is session-specific).

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

Subject-specific ROIs derived from task-based or resting-state analyses (e.g.,
localizer contrasts, seed-based connectivity maps). Each mask's JSON sidecar
documents the contrast, threshold, and statistical criteria.

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

## fMRIPrep (NORDIC)

`fmriprep_nordic/` contains a parallel fMRIPrep run using NORDIC-denoised BOLD
as input (see `nordic/` below). Structure mirrors `fmriprep/` exactly. The
canonical source for each run (original vs. NORDIC) is recorded per-run in the
QC decisions file; see [Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md).

## NORDIC

Raw outputs from the NORDIC thermal noise denoising step, applied to BOLD data
prior to fMRIPrep. Each run produces a denoised NIfTI (`.nii.gz`) and a
diagnostics file (`.mat`).

```
derivatives/nordic/
└── sub-03/
    └── ses-04/
        └── func/
            ├── sub-03_ses-04_task-TBencoding_run-01_bold.nii.gz
            ├── sub-03_ses-04_task-TBencoding_run-01_bold.mat
            └── ...
```

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

Group-level and per-subject behavioral analysis results from the trial-based
memory paradigm, generated by `analyze_behavior.py` in the mmmdata codebase.

```
derivatives/behavioral_analysis/
├── group/
│   ├── accuracy_by_enCon.tsv    # Accuracy broken down by encoding condition
│   └── dprime_by_subject.tsv    # Signal detection (d') per subject
├── figures/                      # Visualization outputs
├── sub-03/
├── sub-04/
└── sub-05/
```

## BIDS Validation

Outputs from BIDS validation and event file extraction processes.

```
derivatives/bids_validation/
├── dataset_description.json
├── eventfiles/          # Extracted event files per subject
│   ├── sub-03/
│   ├── sub-04/
│   └── sub-05/
└── survey_logs/         # Pre-scan questionnaire processing logs
```

## Analysis-Ready Outputs

The `ready/` directory (GLMSingle, naturalistic, and connectivity streams) and
the `preprocessing_qc/` QC decisions files are documented in the
[Analysis-Ready Preprocessing Pipeline](preprocessing-pipeline.md) page.
