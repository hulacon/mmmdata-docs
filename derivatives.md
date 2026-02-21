---
title: Derivatives & Preprocessing
parent: Data Organization
grand_parent: Dataset Description
nav_order: 3
---

# Derivatives & Preprocessing

```
derivatives/
├── fmriprep/           # fMRIPrep preprocessed data (all sessions)
├── atlases/            # Group-level reference atlases in template space
├── anatomical_rois/    # Subject-specific anatomically-defined ROIs
├── functional_rois/    # Subject-specific functionally-defined ROIs
├── mriqc/              # MRIQC output (currently empty — not yet run)
├── fmriprepFR/         # Earlier fMRIPrep run (legacy — to be deprecated)
└── QA/                 # Legacy QA images (likely superseded by MRIQC)
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
    ├──▶ atlases/ (reference parcellations in template space)
    │
    ├──▶ anatomical_rois/ (subject-specific anatomical segmentations)
    │
    ├──▶ functional_rois/ (subject-specific task-defined ROIs)
    │
    └──▶ [downstream analysis stages TBD]
```

- MRIQC and fMRIPrep can likely run in parallel, producing complementary QA output.
- Long-term goal: a single `fmriprep/` output directory (the current `fmriprepFR/`
  and `QA/` directories are legacy and will be deprecated).

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

## Minimally Preprocessed Version

_Documentation forthcoming._

## GLMSingle Output

_Documentation forthcoming._
